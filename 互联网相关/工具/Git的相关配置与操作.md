#ubuntu #git
这里仅记录Git的相关操作，以防日后忘记。
## init_初始化
```
# 进入到要创建仓库的文件夹
git init
```

## config_配置信息
git的配置分等级的，从高到低分别是 global(全局)、system(系统级)、local(仓库级)
```
# 模板
git config (配置等级) 配置建 "配置值"

# 示例
git config --global user.name "陈建浩"
git config --global user.emil "1079170090@qq.com"
```

## remote_配置远端仓库信息
这里需要获取到远端仓库的https地址，或者git地址
### add
添加一个远端仓库
```
# 模板
git remote add (变量名) (项目地址)
# 示例
git remote add origin https://github.com/chenjianhao66/LearnNotes_obsidian.git
```
### remove
删除一个远端仓库
```
## 模板
git remote remove (远端仓库名)
# 示例
git remote remove origin
```

## checkout_切换分支

## rm_删除文件
在git工作空间删除一个文件，会提示你只是在git工作空间删除但文件不会删除，还是文件和git工作空间一起删除。
想要git工作空间和文件一起删除的话，需要加参数 `-f ` 
只是在git工作空间删除但文件保留，需要加参数 `--cached`
```
# 模板
git rm [文件名]

#示例
git rm git配置.md

# git工作空间和文件一起删除
git rm git配置.md -f

# git工作空间记录删除，但是文件没有删除
git rm git配置.md -cached
```

## ubuntu 命令行终端显示分支名
1. 进入home 目录
```
$ cd ~
```

 2. 编辑.bashrc文件
```
$ gedit .bashrc
```

3. 在文件末尾添加以下信息

```
PS1=""
PS1="$PS1"'\[\033[32m\]'        # change to green
PS1="$PS1"'\u@\h '              # user@host<space>
PS1="$PS1"'\[\033[33m\]'        # change to yellow
PS1="$PS1"'\w '                 # pwd
PS1="$PS1"'\[\033[36m\]'        # change color to cyan
PS1="$PS1"'`__git_ps1`'         # git branch
PS1="$PS1"'\[\033[0m\]'         # change color
PS1="$PS1"'\n'                  # new line
PS1="$PS1"'$ '                  # prompt: always $
```

4. 重新启动命令行，效果下图：
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-08-14_16-48.png)




## Git使用中踩过的坑
### git status出现乱码
```
# 解决方案，全局增加设置
git config --global core.quotepath false
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8

☝ 增加以上配置后 git status 出现的中文乱码问题以解决。
```


