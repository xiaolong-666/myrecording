# find 查找多个目录中匹配文件并删除

直接使用`find`命令搭配`-exec`参数即可

```bash
find /usr/lib/python3/dist-packages/iso-tailor /usr/share/iso-tailor -name '__pycache__' -prune -exec rm -rf {} \;
```

其中 `-prune` 参数意味着不递归找到的目标子目录。

