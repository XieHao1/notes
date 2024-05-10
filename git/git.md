# git

# 一.git工作机制

![img](img.assets\1706b3553447ac9f8d95377d965d4fd2.png)

# 二.Git 常用命令

| 命令名称                             | 作用              |
| ------------------------------------ | ----------------- |
| git config --global user.name 用户名 | 设置用户签名      |
| git config --global user.email 邮箱  | 设置用户email地址 |
| git init                             | 初始化本地库      |
| git status                           | 查看本地库状态    |
| git add 文件名                       | 添加到暂存区      |
| git commit -m “日志信息” 文件名      | 提交到本地库      |
| git reflog                           | 查看历史记录      |
| git log                              | 查看历史记录详情  |
| git reset --hard 版本号              | 版本穿梭          |

### 分支的操作

| 命令名称            | 作用                         |
| ------------------- | ---------------------------- |
| git branch 分支名   | 创建分支                     |
| git branch -v       | 查看分支                     |
| git checkout 分支名 | 切换分支                     |
| git merge 分支名    | 把指定的分支合并到当前分支上 |

分支冲突解决:

编辑有冲突的文件，**删除特殊符号**，决定要使用的内容

```tex
<<<<<<< HEAD
当前分支的代码 
======= 
合并过来的代码 
>>>>>>> hot-fix
```

### 远程仓库(github)操作

| 命令名称                           | 作用                                                      |
| ---------------------------------- | --------------------------------------------------------- |
| git remote -v                      | 查看当前所有远程地址别名                                  |
| git remote add 别名 远程地址       | 起别名                                                    |
| git push 别名 分支                 | 推送本地分支上的内容到远程仓库                            |
| git clone 远程地址                 | 将远程仓库的内容克隆到本地                                |
| git pull 远程库地址别名 远程分支名 | 将远程仓库对于分支最新内容拉下来后与 当前本地分支直接合并 |



# 三.IDEA集成Git

## 配置 Git 忽略文件

创建忽略规则文件 **xxxx.ignore**，一般项目工程中会自带 

```
# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
.classpath
.project
.settings
target
.idea
*.iml
```

在.gitconfig 文件中引用忽略配置文件（此文件在 Windows 的家目录中）

```
[user]
    name = Layne
    email = Layne@atguigu.com
[core]
	excludesfile = C:/Users/86177/git.ignore
```

**这里要使用“正斜线（/）”，不要使用“反斜线（\）**

## 定位git程序

![image-20220418135211162](img.assets\image-20220418135211162.png)

## 初始化Git

在菜单栏VCS ->  Create Git Repository

![image-20220418135823775](img.assets\image-20220418135823775.png)

## 上传本地仓库，提交到远程

![image-20220418143307952](img.assets\image-20220418143307952.png)



## 切换版本

![image-20220418143458645](img.assets\image-20220418143458645.png)

## 创建分支-快捷键Ctrl+Shift+`

![image-20220418143751872](E:\笔记\git\img.assets\image-20220418143751872.png)

或者在右下角中的master:

![image-20220418143959030](img.assets\image-20220418143959030.png)

## 合并分支

![image-20220418144534478](img.assets\image-20220418144534478.png)



# 四.IDEA集成githup

在菜单栏File->Setting->搜索栏搜GitHub，添加GitHub账号：

![image-20220418145045354](img.assets\image-20220418145045354.png)

**使用token登录**

![image-20220418145700259](img.assets\image-20220418145700259.png)

**生成token**

![image-20220418145801863](img.assets\image-20220418145801863.png)

## 拉取远程库代码合并本地库

![image-20220418150119410](img.assets\image-20220418150119410.png)

push 的操作是会被拒绝的。也就是说， 要想 push 成功，一定要保证本地库的版本要比远程库的版本高！ **因此一个成熟的程序员在动手改本地代码之前，一定会先检查下远程库跟本地代码的区别！如果本地的代码版本已经落后，切记要先 pull 拉取一下远程库的代码，将本地代码更新到最新以后，然后再修改，提交，推送！**



# 五.IDEA集成Gitee码云

首先，要在IDEA安装Gitee插件。

在菜单栏选File->Settings->Plugins，搜Gitee。

![image-20220418151122706](img.assets\image-20220418151122706.png)

登录：使用邮箱和密码进行登录

![image-20220418151615909](img.assets\image-20220418151615909.png)

## 导入githup项目

![image-20220418152529610](img.assets\image-20220418152529610.png)