# pytest 测试 getpass

近期编写 `pytest ` 用例时，发现需要用户输入密码的 `getpass` 模块，无法做到自动填充密码，为此分析了下原因。

默认情况下，`getpass()` 将提示符写入 `/dev/tty` 中，如果写入不了，就会写入到 `sys.stderr` 中。这也是在终端中看不见回显的原因。

知道原理后，那么就不难理解，在 `pytest` 中，使用默认的 `sys.stdin` 进行自动填充输入，是不可行的。因为 `getpass` 没有写入 `sys.stdin` 流中。

最终通过 `monkeypatch.setattr()` 方法，对 `getpass` 做额外处理。

```pytho
def test_passwd_format(self,monkeypatch):
    """
    测试自动化安装,密码格式非法
    Args:

    Returns:

    """
    test_args = ["user", "xiaolong", "--create"]
    monkeypatch.setattr("getpass.getpass", lambda _: "-")
    assert iso_tailor_test(test_args) == ResultCode.INVALID_PARAMETER_CODE
```

