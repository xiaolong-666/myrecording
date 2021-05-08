# oh-my-zsh让终端更好用
## 安装zsh
oh-my-zsh为开源项目，基于zsh，需要先安装zsh
```bash
sudo apt install zsh
```

### 安装oh-my-zsh
开源项目地址：https://github.com/ohmyzsh/ohmyzsh
```bash
wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh
sh install.sh
```

## 插件推荐
### autojump
功能：实现目录间跳转，想去哪个目录直接`j + 目录名`，不用在频繁的cd了。`jo + 目录名`在快速用文件管理器打开。
autojump 就是通过记录你在 history 中的行为把你访问过的文件夹路径都 cache 下来，当你输入路径名的时候会模糊匹配你之前cd过的目录路径，配合后面的自动提示插件

```bash
git clone git://github.com/joelthelion/autojump.git
cd autojump
./install.py
vim ~/.zshrc
# 在文件里找到plugins，添加
plugins=(autojump)
# 在文件末尾添加
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
```

### zsh-autosuggestion
输入命令时可提示自动补全（灰色部分），然后按键盘 → （！！！！上下左右的右键，不是tab键）即可补全
```bash
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
vim ~/.zshrc
# 在文件里找到plugins，添加
plugins=(
  autojump
  zsh-autosuggestions
)
```
### zsh-syntax-highlighting
日常用的命令会高亮显示，命令错误显示红色
```bash
git clone git://github.com/zsh-users/zsh-syntax-highlighting $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
vim ~/.zshrc
# 在文件里找到plugins，添加
plugins=(
  autojump
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```
