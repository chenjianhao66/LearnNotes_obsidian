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










## Git使用中踩过的坑
### git status出现乱码
```
# 解决方案，全局增加设置
git config --global core.quotepath false
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8

☝ 增加以上配置后 git status 出现的中文乱码问题以解决。
```
