# echo打印带有颜色的字
## 命令格式
echo -e "\033[字背景颜色;文字颜色m 字符串 \033[0m"
例如：
```bash
echo -e "\033[47;30m xiaolong test! \033[0m"
```
其中47代表背景色，30代表字体颜色。需要使用 `-e`参数，从`man`手册可知：`-e     enable interpretation of backslash escapes`

## 测试代码
```bash
echo -e "\033[30m 黑色字 \033[0m"        # 
echo -e "\033[31m 红色字 \033[0m"
echo -e "\033[32m 绿色字 \033[0m"
echo -e "\033[37m 白色字 \033[0m"

echo -e "\033[40;37m 黑底白字 \033[0m"
echo -e "\033[41;37m 红底白字 \033[0m"
echo -e "\033[42;37m 绿底白字 \033[0m"
echo -e "\033[43;37m 黄底白字 \033[0m"
echo -e "\033[44;37m 蓝底白字 \033[0m"
echo -e "\033[45;37m 紫底白字 \033[0m"
echo -e "\033[46;37m 天蓝底白字 \033[0m"
echo -e "\033[47;30m 白底黑字 \033[0m"

```
## 其他有趣指令
```bash
echo -e "\033[0m 默认的颜色 \033[0m"
echo -e "\033[4m 下划线 \033[0m"
echo -e "\033[5m 闪烁 \033[0m"

echo -e "\033[5m \033[4m 闪烁加下划线 \033[0m"
```

## 参考链接
https://blog.csdn.net/qq_37858386/article/details/78614418