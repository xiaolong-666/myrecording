# Tex Live



## 安装

按照官方文档安装即可：

> 1. `cd /tmp` #您选择的工作目录
> 2. 下载：`wget `https://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz      或： `curl -L -o install-tl-unx.tar.gz https://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz`（或通过您喜欢的任何其他方法）
>    ``
> 3. `zcat < install-tl-unx.tar.gz | tar xf - ``# 注意最后 - 在该命令行上`
> 4. `cd 安装-tl-*`
> 5. `perl ./install-tl --no-interaction # 以 root 身份或具有 可写目标`
>    `# 可能需要几个小时才能运行`
> 6. 最后，将`/usr/local/texlive/YYYY/bin/PLATFORM`添加到您的 PATH 前面，
>    例如`/usr/local/texlive/2024/bin/x86_64-linux`



## 运行

VsCode 中安装 插件 LaTex Workshop 插件

直接编译时如果提示 `lualatex: unrecognized option '-pdf'` 错误，则：

> 1. 打开 `.vscode/settings.json` 文件：
>    - 在 VS Code 中，按下 `Ctrl + Shift + P` 打开命令面板，输入 `Preferences: Open Workspace Settings (JSON)` 并选择。
> 2. 查找并修改 `latex-workshop.latex.tools` 配置：
>    - 确保 `lualatex` 的配置项中没有 `-pdf` 选项。
> 3. 保存文件：
>    - 保存对 `settings.json` 文件的修改。
> 4. 重新编译文档：
>    - 使用 LaTeX Workshop 重新编译您的 `deepin-compatible.tex` 文件。



语法参考：https://www.latexstudio.net/LearnLaTeX/basic/03.html



## 参考链接

https://www.tug.org/texlive/quickinstall.html