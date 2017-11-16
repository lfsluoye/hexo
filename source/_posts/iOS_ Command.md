layout: post
title: "常用命令"
date: 2017-1-31 
categories: Suibi
comments: false
tags: 命令行
---
### 允许任何来源
sudo spctl --master-disable
### brew的命令行
1. 软件的安装: brew install
2. 软件的搜索: brew search
3. 软件的信息: brew info
4. 软件的卸载: brew uninstall

### brew的扩充
1. brew install caskroom/cask/brew-cask
2. 软件安装: brew cask install
3. 软件卸载: brew cask uninstall

### xcode
1. 查看当前: xcode xcode-select --print-path
2. 切换xcode: sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer


### cocopods安装异常
```
pod repo remove master
pod repo add master https://gitcafe.com/akuandev/Specs.git
pod repo update
git clone https://git.coding.net/CocoaPods/Specs.git ~/.cocoapods/repos/master
```

### cocopods 更新
 > pod update --verbose --no-repo-update
 
### Alcatraz插件管理工具
##### 安装
> curl -fsSL https://raw.github.com/supermarin/Alcatraz/master/Scripts/install.sh | sh

##### 删除
> rm -rf ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins/Alcatraz.xcplugin

##### 删除所有通过Alcatraz安装的安装包
> rm -rf ~/Library/Application\ Support/Alcatraz/

### git
查看状态
> git status

查看分支
> git branch

添加到仓库
> git add .

提交到仓库
> git commit -m"提交说明"

切换分支
> git checkout dev

创建并且切换到分支
> git checkout -b dev

查看远程tag 查看远程版本库
> git ls-remote -h -t

删除分支
> git branch -d dev

合并分支
> git merge --no-ff -m "merge with no-ff" dev

放弃所有修改
> git clean -xdf

推送tag
> git push origin [tagname]


