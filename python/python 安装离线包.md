# python 安装离线包

最近需要将 `ansible` 部署到内网环境中，可使用 `pip3` 的离线安装方式

## 下载离线相关包

```bash
pip3 download -d ./ ansible 
```

执行结束后，会在当前目录下，下载 `ansible` 的源码和相应的依赖包

然后将所有的离线包，上传到内网环境中的指定目录 `ansible_src` 下

## 使用 `pip3` 离线安装方式

```bash
pip3 install --no-index --find-links=file:./ansible_src ansible
```



## 参考链接

https://imshuai.com/python-pip-install-package-offline-tensorflow

