# sed命令

## 替换匹配内容的下一行
```bash
sed -i '/ATTest/{n;n;s/"actual.*/"actual": "123.123.123"/;}' amd64_ut_data.json
```
其中 `n`代表下一行，