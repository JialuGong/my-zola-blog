+++
title= "my manjaro" 
date=2020-04-21
tags=["manjaro"]
process = 0.333
+++
我的manjaro配置

##### 一 配置国内镜像
添加国内清华源

`/etc/pacman.conf`
```conf
[archlinuxcn]
SigLevel= TrustedOnly
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```
安装密钥
```shell
$ sudo pacman -Syu archlinuxcn-keyring
```
更新
```shell
$ sudo pacman -Syyu
```
##### 二 设置manjaro时间
```shell
$ sudo timedatectl set-local-rtc true
```
##### 三 一些工具的安装
1. yay
```
sudo pacman -S yay
```
2. 输入法

```shell
$ sudo pacman -S fcitx fcitx-qt4 fcitx-im  fcitx-configtool fcitx-sogoupinyin
$ sudo pacman -S fcitx-rime fcitx-cloudpinyin fcitx-googlepinyin
```
在 `~/.xprofile`中
```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```
3. 安装zsh
```shell
$ sudo pacman -S zsh
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
安装插件
```shell
$ git clone https://github.com/zsh-users/zsh-completions ~/.oh-my-zsh/custom/plugins/zsh-completions

$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/plugins/zsh-syntax-highlighting

$ git clone https://github.com/zsh-users/zsh-autosuggestions.git ~/.oh-my-zsh/plugins/zsh-autosuggestions

$ vim ~/.zshrc
  # edit plugins & save
  plugins=(git zsh-syntax-highlighting docker docker-compose zsh-autosuggestions zsh-completions)

$ autoload -U compinit && compinit
```
4. 安装tim
```shell
$ yay -S deepin-wine-tim
```
软链接tim文件地址
```shell
$ ln -s /home/huaiyu/.deepinwine/Deepin-TIM/drive_c/users/huaiyu/'My Documents'/'Tencent Files'/1368012668/FileRecv ~/download/Tim-FileRe
```
5. 安装wps
```shell
$ sudo pacman -S ttf-wps-fonts
$ sudo pacman -S wps-offitce
```
6. 安装telegram
```shell
$ sudo pacman -S telegram-desktop
```
7. 安装v2ray
```
sudo pacman -S v2ray
```
`/etc/v2ray/config.json`
```json
{
  "inbound": {
    "port": 1080,
    "listen": "127.0.0.1",
    "protocol": "socks",
    "settings": {
      "auth": "noauth"
    }
  },
  "outbound": {
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
          "address": "{ip}",
          "port": {port},
          "users": [
            {
              "id": "{id}",
              "alterId": {alterId},
              "security": "{security}"
            }
          ]
        }
      ]
    },
    "streamSettings": {
      "network": "tcp"
    }
  }
}
```
8. 安装proxychains
```
$ sudo pacman -S proxychains-ng
```
配置`/etc/proxydriver.d/default.conf`
```conf
enabled=true
autoconfig_url='{pac path}'
```
9. 安装google chrome
```shell
$ sudo pacman -S google-chrome
```
10. 安装firacode
```
$ sudo pacman -S otf-fira-code
```
11. 安装sdcv
```
$ sudo pacman -S sdcv
```
在`/home/huaiyu/.stardict/dic`
```shell 
tar -xvf stardict-cedict-gb-2.4.2.tar.bz2 -C .
```
###### 四、开发环境配置
1. java
```shell 
$ sudo pacman -S java-runtime-common java-environment-common
$ yay jdk8  # Select extra/jdk8-openjdk
```
列举java环境
```shell 
$ archlinux-java status
```
设置java环境
```shell
$ sudo archlinux-java set java-8-openjdk
```
2. node
```
$ sudo pacman nvm
$ nvm install --lts
$ nvm use v12.15.0
```
配置nvm环境

`~/.zshrc`
```
[ -z "$NVM_DIR" ] && export NVM_DIR="$HOME/.nvm"
source /usr/share/nvm/nvm.sh
source /usr/share/nvm/bash_completion
source /usr/share/nvm/install-nvm-exec
```
配置npm淘宝源
`~/.npmrc`
```
registry=https://registry.npm.taobao.org
```
3. rust 开发环境配置
```shell 
$ yay install rustup
$ rustup  toolchain install stable//选择了stable tool chain
$ rustup default
```
环境配置

`~/.zshrc`
```
export CARGOPATH=~/.cargo
export PATH=$PATH:$CARGOPATH/bin
```
安装一些rust开发工具
```shell 
$ rustup component add rustfmt
$ rustup component add clippy
$ rustup component add rust-analysis
$ rustup component add rls
```
安装gdb用于调试
```
# sudo pacman -S gdb
```
4. C++
```
sudo pacman -S clang
```
5. mongodb
```shell 
$ sudo pacman -S mongodb-bin 
```
6. latex
```shell 
#texlive
$ sudo pacman -S texlive-core texlive-langchinese
#texstudio
$ sudo pacman -S texstudio
```
7. redis
```
$ sudo pacman -S redis
$ sudo systemctl start redis
```
##### 五　安装开发工具
1.  安装vscode
```shell 
$ sudo pacman -S visual-studio-code-bin
```
2. jetbrains toolbox
```
$ yay -S jetbrains-toolbox
```
##### 其它工具
- postman
- typora
- wireshark
- tar
- pandoc
- zoom
- 网易云音乐

