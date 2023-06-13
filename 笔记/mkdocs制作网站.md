# mkdocs 制作网站

## 安装

`mkdocs` 是用 `python` 开发的工具，直接使用 `pip` 命令安装即可

```bash
pip3 install mkdocs
```

##  使用

```bash
mkdocs new project
```

就会在本地建立一个 `project` 文件夹，其中包括了一个 `mkdocs.yml` 和 一个 `docs` 文件夹

- mkdocs.yml: 配置文件，主要配置站点名字，板块等具体配置
- docs: 存放要写的 `Markdown` 文档的地方

可直接运行以下命令查看效果

```
mkdocs serve
Running at: http://127.0.0.1:8000/
...
```

后访问 `http://127.0.0.1:8000/` 就可以看到生成文档的效果了



在配置文件（mkdocs.yml）中，添加以下字段：`theme: readthedocs` 可以更改主题为红帽的主题，显着更加专业。



## 参考链接

https://github.com/zimocode/mkdocs-docs-zh/blob/master/docs/index.md