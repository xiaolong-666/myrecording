# 编写ansible连接插件
本文主要是记录如何编写ansible的连接插件（systemd-nspawn替换chroot连接），并在自己的项目中应用进去。

## 编写插件文件
模仿内置的`chroot`连接插件方式，修改一下。通过pip3方式安装的连接插件目录：`/usr/local/lib/python3.7/dist-packages/ansible/plugins/connection/`在此目录下，可以看到官方的所有连接插件。

本文从`ansible`上游`issue`中，找到一份`PR`, 摘取下来，改动部分内容
```python
# This file is part of Ansible
#
# Ansible is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# Ansible is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with Ansible.  If not, see <http://www.gnu.org/licenses/>.
from __future__ import (absolute_import, division, print_function)
__metaclass__ = type

DOCUMENTATION = """
    author: Maykel Moya <mmoya@speedyrails.com>; xiaolong<longqiang@uniontech.com>
    connection: nspawn
    short_description: Interact with local systemd-nspawn container
    description:
        - Run commands or put/fetch files to an existing systemd-nspawn
          container on the Ansible controller. Unlike the chroot driver,
          this will ensure that /proc, /sys, and /dev are all properly
          configured.
        - Set "ansible_nspawn_args" to pass additional arguments to
          systemd-nspawn (e.g., --bind, --private-network, --private-user,
          etc).
    version_added: "2.9"
    options:
      remote_addr:
        description:
            - The path of the chroot you want to access.
        default: inventory_hostname
        vars:
            - name: ansible_host
      executable:
        description:
            - User specified executable shell
        ini:
          - section: defaults
            key: executable
        env:
          - name: ANSIBLE_EXECUTABLE
        vars:
          - name: ansible_executable
      nspawn_args:
        description:
          - Extra command line arguments to pass to systemd-nspawn
        env:
          - name: ANSIBLE_NSPAWN_ARGS
        vars:
          - name: ansible_nspawn_args
        default: ''
"""

import distutils.spawn
import os
import os.path
import subprocess
import traceback
import shlex

from ansible import constants as C
from ansible.module_utils import six
from ansible.module_utils.six.moves import shlex_quote
from ansible.errors import AnsibleError
from ansible.module_utils._text import to_bytes, to_text
from ansible.plugins.connection import ConnectionBase, BUFSIZE

try:
    from __main__ import display
except ImportError:
    from ansible.utils.display import Display
    display = Display()


class Connection(ConnectionBase):
    ''' Local nspawn based connections '''

    transport = 'nspawn'
    has_pipelining = True
    # become_methods = frozenset(C.BECOME_METHODS)

    def __init__(self, play_context, new_stdin, *args, **kwargs):
        super(Connection, self).__init__(play_context, new_stdin,
                                         *args, **kwargs)

        # display.vvv("NSPAWN ARGS %s" % self._play_context.nspawn_args)
        display.vvv("NSPAWN ARGS %s" % self.get_option('nspawn_args'))

        self.ostree = os.path.normpath(self._play_context.remote_addr)

        if os.geteuid() != 0:
            raise AnsibleError("nspawn connection requires running as root")

        # we're running as root on the local system so do some
        # trivial checks for ensuring 'host' may be an OS tree dir
        if not os.path.isdir(self.ostree):
            raise AnsibleError("%s is not a directory" % self.ostree)

        # As systemd-nspawn will, we check the existence of os-release files
        # in the container tree to think it looks like an OS tree enough
        # see man systemd-nspawn(1) and os-release(5)
        if not (
            os.path.isfile(os.path.join(self.ostree, "usr/library/os-release"))
            or os.path.isfile(os.path.join(self.ostree, "etc/os-release"))
        ):
            raise AnsibleError("%s does not contain an os-release file"
                               % self.ostree)

        self.nspawn_cmd = distutils.spawn.find_executable('systemd-nspawn')
        if not self.nspawn_cmd:
            raise AnsibleError("systemd-nspawn command not found in PATH")

    def _connect(self):
        ''' Connect to the container. Nothing to do '''
        super(Connection, self)._connect()
        if not self._connected:
            display.vvv(u"THIS IS A LOCAL NSPAWN CONTAINER", host=self.ostree)
            self._connected = True

    def _buffered_exec_command(self, cmd, stdin=subprocess.PIPE):
        ''' run a command in the container.  This is only needed for
        implementing put_file() get_file() so that we don't have to
        read the whole file into memory.
        compared to exec_command() it looses some niceties like being
        able to return the process's exit code immediately.
        '''
        executable = (
            C.DEFAULT_EXECUTABLE.split()[0]
            if C.DEFAULT_EXECUTABLE
            else '/bin/sh')

        nspawn_args = self.get_option('nspawn_args')
        if six.PY2:
            nspawn_args = shlex.split(
                to_bytes(nspawn_args, errors='surrogate_or_strict')
            )
        else:
            nspawn_args = shlex.split(
                to_text(nspawn_args, errors='surrogate_or_strict')
            )

        local_cmd = [self.nspawn_cmd, '-D', self.ostree, '-q'] + nspawn_args + [
            '--', executable, '-c', cmd]

        display.vvv("EXEC %s" % (local_cmd), host=self.ostree)
        local_cmd = [to_bytes(i, errors='surrogate_or_strict')
                     for i in local_cmd]
        p = subprocess.Popen(local_cmd, shell=False, stdin=stdin,
                             stdout=subprocess.PIPE, stderr=subprocess.PIPE)

        return p

    def exec_command(self, cmd, in_data=None, sudoable=False):
        ''' run a command in the container '''
        super(Connection, self).exec_command(cmd, in_data=in_data,
                                             sudoable=sudoable)

        p = self._buffered_exec_command(cmd)
        stdout, stderr = p.communicate(in_data)
        return (p.returncode, stdout, stderr)

    def _prefix_login_path(self, remote_path):
        ''' Make sure that we put files into a standard path
            If a path is relative, then we need to choose where to put it.
            ssh chooses $HOME but we aren't guaranteed that a home dir will
            exist in any given container. So for now we're choosing "/"
            instead.
            This also happens to be the former default.
            Can revisit using $HOME instead if it's a problem
        '''
        if not remote_path.startswith(os.path.sep):
            remote_path = os.path.join(os.path.sep, remote_path)
        return os.path.normpath(remote_path)

    def put_file(self, in_path, out_path):
        ''' transfer a file from local to the container '''
        super(Connection, self).put_file(in_path, out_path)
        display.vvv("PUT %s TO %s" % (in_path, out_path), host=self.ostree)

        out_path = shlex_quote(self._prefix_login_path(out_path))
        try:
            with open(to_bytes(in_path, errors='surrogate_or_strict'),
                      'rb') as in_file:
                try:
                    p = self._buffered_exec_command(
                        'dd of=%s bs=%s' % (out_path, BUFSIZE),
                        stdin=in_file
                    )
                except OSError:
                    raise AnsibleError(
                        "nspawn connection requires dd command in container"
                    )
                try:
                    stdout, stderr = p.communicate()
                except:
                    traceback.print_exc()
                    raise AnsibleError("failed to transfer file %s to %s"
                                       % (in_path, out_path))
                if p.returncode != 0:
                    raise AnsibleError(
                        "failed to transfer file %s to %s:\n%s\n%s"
                        % (in_path, out_path, stdout, stderr)
                    )
        except IOError:
            raise AnsibleError("file or module does not exist at: %s"
                               % in_path)

    def fetch_file(self, in_path, out_path):
        ''' fetch a file from the container to local '''
        super(Connection, self).fetch_file(in_path, out_path)
        display.vvv("FETCH %s TO %s" % (in_path, out_path), host=self.ostree)

        in_path = shlex_quote(self._prefix_login_path(in_path))
        try:
            p = self._buffered_exec_command('dd if=%s bs=%s'
                                            % (in_path, BUFSIZE))
        except OSError:
            raise AnsibleError(
                "nspawn connection requires dd command in the container"
            )

        with open(to_bytes(out_path, errors='surrogate_or_strict'),
                  'wb+') as out_file:
            try:
                chunk = p.stdout.read(BUFSIZE)
                while chunk:
                    out_file.write(chunk)
                    chunk = p.stdout.read(BUFSIZE)
            except:
                traceback.print_exc()
                raise AnsibleError("failed to transfer file %s to %s"
                                   % (in_path, out_path))
            stdout, stderr = p.communicate()
            if p.returncode != 0:
                raise AnsibleError("failed to transfer file %s to %s:\n%s\n%s"
                                   % (in_path, out_path, stdout, stderr))

    def close(self):
        ''' terminate the connection; nothing to do here '''
        super(Connection, self).close()
        self._connected = False

```

需注意：代码中的`DOCUMENTATION`字段很重要，刚开始就是忽略它，导致调用处无法传入参数

## 项目中使用

1. 在与`ansible.cfg`同层目录中，创建`connection_plugins`目录，将上面的`nspawn.py`文件放到该目录下即可使用。
2. 在`host`中，指定连接方式为`nspawn`的具体参数
```bash
[nspawn]
/var/tmp/live ansible_connection=nspawn ansible_python_interpreter=/usr/bin/python3
```
3. 在入口处，指定`hosts`为`nspawn`后，下面执行的`task`均会通过`systemd-nspawn`方式执行