# python logging 使用

## 简介

logging 模块有这几种 Python 类型，Logger、LogRecord、Filter、Handler、Formatter，解释如下：

- Logger：即 Logger Main Class，是我们进行日志记录时创建的对象，我们可以调用它的方法传入日志模板和信息，来生成一条条日志记录，称作 Log Record。
- Log Record：就代指生成的一条条日志记录。
- Handler：即用来处理日志记录的类，它可以将 Log Record 输出到我们指定的日志位置和存储形式等，如我们可以指定将日志通过 FTP 协议记录到远程的服务器上，Handler 就会帮我们完成这些事情。
- Formatter：实际上生成的 Log Record 也是一个个对象，那么我们想要把它们保存成一条条我们想要的日志文本的话，就需要有一个格式化的过程，那么这个过程就由 Formatter 来完成，返回的就是日志字符串，然后传回给 Handler 来处理。
- Filter：另外保存日志的时候我们可能不需要全部保存，我们可能只需要保存我们想要的部分就可以了，所以保存前还需要进行一下过滤，留下我们想要的日志，如只保存某个级别的日志，或只保存包含某个关键字的日志等，那么这个过滤过程就交给 Filter 来完成

在开发过程中，将配置在代码里面写死并不是一个好的习惯，更好的做法是将配置写在配置文件里面，我们可以将配置写入到配置文件，然后运行时读取配置文件里面的配置，这样是更方便管理和维护的。常见的配置文件有 ini 格式、yaml 格式、JSON 格式。本文推荐yaml格式，`logging.yaml`内容如下：

```yaml
version: 1
formatters:
  brief:
    format: "%(asctime)s - %(message)s"
  simple:
    format: "%(asctime)s - %(filename)s-%(lineno)d - %(levelname)s : %(message)s"
filters:
  onlineFilter:
     # 继承filter 自己实现过滤要求
    (): oem.onlineFilter
handlers:
  console:
    class: logging.StreamHandler
    formatter: brief
    level: WARNING
    stream: ext://sys.stdout
    filters: ["onlineFilter"]
  rotateFile:
    class: logging.handlers.RotatingFileHandler
    level: DEBUG
    formatter: simple
#    filename: /var/tmp/oemscan.log
    filename: ./oemscan.log
    maxBytes: 10485760 # 10MB
    backupCount: 3
    encoding: utf8
  mailhandler:
    # 需要修改默认的SMTPHandler.emit 中： mtplib.SMTP -->smtplib.SMTP_SSL
    class: logging.handlers.SMTPHandler
    level: WARNING
    formatter: simple
    mailhost: !!python/tuple
      - 'smtp.exmail.qq.com'
      - 465
    fromaddr: "xxxx@xxxx.com"
    toaddrs:
      - "xxxx@xxx.com"
      - "xxxx@xxxx.com"
    subject: "安全扫描结果"
    credentials: !!python/tuple
      - 'xxxx@xxxx.com'
      - '******'

loggers:
  oemscan:
    level: DEBUG
    handlers: [console,rotateFile,mailhandler]
```

加载配置使用方式：

```python
import logging.config
import yaml

class onlineFilter(logging.Filter):
    def filter(self, record) -> bool:
        # 过滤异常堆栈信息
        if record.exc_info:
            if "Traceback (most recent call last)" in record.exc_text:
                return False
        return True

with open("./logging.yaml", 'r') as f:
    config = yaml.unsafe_load(f.read())
    logging.config.dictConfig(config)

logger = logging.getLogger("oemscan")

```





## 参考链接

https://zhuanlan.zhihu.com/p/454463040

https://madmalls.com/blog/post/smtphandler-send-error-email/

https://stackoverflow.com/questions/61456543/logging-in-python-with-yaml-and-filter