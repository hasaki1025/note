# Git

- ### Git执行流程

  - 工作区

    用于开发程序代码，一般在项目的根路径下执行git init去让git 接管项目

  - 暂存区

    在内存中暂时存放工作区中修改好的内容，使用git add将工作区中的内容存放至暂存区

    也可以通过git reset 将暂存区中的内容回滚至工作区

  - 本地版本仓库

    用来存放本地开发的项目,使用git commit -m 注释 将暂存区中的内容提交至本地版本仓库

    可以通过git reset --hard 版本号回滚版本

  - 远程版本仓库

    在云端保存写好的项目，可以在任意的电脑上随时查看和修改,使用git push origin master将本地库中的内容推送至远程仓库

    使用 git pull origin master 从远程仓库中的最新代码更新到本地

- ### 常用命令

  - 查看当前用户名和邮箱

    ```shell
    git config --global user.name
    git config --global user.email
    ```

  - 设置用户名和邮箱

    ```shell
    git config --global user.name "用户名"
    git config --global user.email "邮箱"#这里的邮箱是一个虚拟邮箱
    ```

    - 设置好git后可以在当前使用的用户目录下找到.gitconfig文件，其中包含了用户名和邮箱
    - 该用户签名和github上的用户没有关系

  - git解决乱码问题

    ```shell
    #第1步
    git config --global core.quotepath false
    #第2步
    #打开${git_home}/etc/bash.bashrc文件，在其后添加两行
    export LAND="zh_CN.UTF-8"
    export LC_ALL="zh_CN.UTF-8"
    ```

  - git初始化本地库

    如果想要git管理对应的目录只需要在该目录下使用git bash并输入git init就可将该目录交给git管理，使用了该命令后该目录下会生成.git文件夹

    ```shell
    git init
    ```

  -  查看git仓库状态

    ```shell
    git status
    ```

  - 添加暂存区

    ```shell
    git add 文件名称
    ```

  - 取消追踪暂存区中的文件

    ```shell
    git rm --cached 文件名称
    ```

  - 提交本地库(每次提交暂存区的内容都会修改head指针指向最新提交的版本)

    ```shell
    git commit -m "日志信息" 文件名称
    
    $ git commit -m "1.0.0Lastest" *
    warning: LF will be replaced by CRLF in hello.txt.
    The file will have its original line endings in your working directory
    warning: LF will be replaced by CRLF in tt.txt.
    The file will have its original line endings in your working directory
    [master (root-commit) a68936e] 1.0.0Lastest
     2 files changed, 5 insertions(+)
     create mode 100644 hello.txt
     create mode 100644 tt.txt
    
    #a68936e代表的就是这次提交的版本号
    ```

  - 显示可引用的历史版本记录。

    ```shell
    git reflog
    #使用示例
    $ git reflog
    a68936e (HEAD -> master) HEAD@{0}: commit (initial): 1.0.0Lastest
    
    ```

  - 查看详细日志信息

    ```shell
    git log
    ```

  - 版本回溯(将工作区中的内容回溯至指定版本)

    ```shell
    git reset --hard 回溯到哪个版本的版本号
    ```

- ### Git分支操作

  - Git分支概念

    分支可以理解为副本，每一个项目都有一个master主线分支，该分支作为最终项目开发，如果master分支上需要添加或者是修改某些内容则可以添加一条新的分支（其实就是复制master分支上的内容并新建一个项目），新的分支并不会干扰主线的分支的开发，新分支开发完成后可以 将该分支添加至主线分支上

  - 分支基础操作

    - 创建分支

      ```shell
      git branch 分支名称
      ```

    - 查看分支

      ```shell
      git branch -v
      ```

    - 切换分支

      ```shell
      git checkout 分支名称
      ```

      - 切换分支会修改本地文件

    - 把指定分支合并到当前分支上

      ```java
      git merge 分支名称
      ```

      - 冲突合并

        假设两个分支上对于同一个文件都有不同的修改此时git无法知道哪个分支是正确的需要人为干预

        ```shell
        pcdn@DESKTOP-LQH5E9F MINGW64 ~/Desktop/gittest (master)
        $ git merge test
        Auto-merging gg.txt
        CONFLICT (content): Merge conflict in gg.txt
        Automatic merge failed; fix conflicts and then commit the result.
        
        pcdn@DESKTOP-LQH5E9F MINGW64 ~/Desktop/gittest (master|MERGING)
        $ git status
        On branch master
        You have unmerged paths.
        (fix conflicts and run "git commit")
        (use "git merge --abort" to abort the merge)
        
        Unmerged paths:
        (use "git add <file>..." to mark resolution)
        both modified:   gg.txt
        
        no changes added to commit (use "git add" and/or "git commit -a")
        
        ```

      - 此时打开发生冲突的文件，文件的末尾会写出冲突的地方，可以修改当前分支上两者冲突的位置，并添加至缓存区，之后直接提交（此次提交不能带有文件名否则报错）

- ### Git远程仓库使用

  - #### 基本概念

    - 同一团队

      用户可以将代码推送（push）至远程仓库中，其他用户可以使用clone将远程库中的代码克隆至本地仓库，其他用户修改代码后也可以将自己本地的代码push至远程仓库（但是需要是同一团队的授权），与此同时其他用户如果已经clone了远程仓库的代码也可以使用pull将远程仓库的代码拉取到本地仓库（同步修改）

    - 跨团队

      如果其他团队想要该代码可以使用fork（复制一份副本）将该代码放在自己的远程库中并clone到本地，修改过后可以使用pull request告诉原本代码团队，原本代码团队通过审核后可以选择是否将该代码同步到自己的代码库（merge）

  - #### 基本使用

    - 创建远程仓库

    - 创建远程仓库别名

      ```shell
      git remote -v #查看当前所有远程地址别名
      git remote add 别名 远程地址
      
      
      $ git remote -v
      demo    https://github.com/hasaki1025/git-demo.git (fetch)#用于拉取
      demo    https://github.com/hasaki1025/git-demo.git (push)#用于push
      
      ```

    - 将本地代码推送至远程仓库

      ```shell
      git push 别名/链接 分支名称
      ```

      - 产生以下错误的原因一般是服务器的SSL证书没有官方认证可以通过以下指令解决

        ```shell
        fatal: unable to access 'https://github.com/hasaki1025/git-demo.git/': OpenSSL SSL_read: Connection was aborted, errno 10053
        
        #解决
        git config --global http.sslVerify "false"
        ```

      - 提交后会弹出一个登录框需要登录gitHub的账号或者使用口令登录

    - 推送成功后可以在github上看到推送的代码并且可以直接修改

    - 拉取修改过后的代码

      ```shell
      git pull 仓库名称/链接 分支名称
      ```

      - 出现以下错误代表当前本地库中做出的修改并没有提交(在push 和pull之后都需要add和commit)

        ```shell
         Pulling is not possible because you have unmerged files.
         #解决方法就是git add 和提交（不需要文件名称）
        ```

    - 克隆远程仓库

      ```shell
      git clone 链接
      ```

      克隆之后会拉取代码并且init git仓库还会创建链接别名

    - 团队认证

      在github官网上找到指定仓库找到settting选项，在collaboratiors中可以看到add people按钮点击后输入需要添加的成员的名称即可，之后该页面中邀请栏中会产生一个用户邀请链接将其发送给被邀请的用户，该用户即可选择是否加入团队

    - fork操作

      打开一个别人的仓库点击右上角的fork即可在自己的远程仓库上创建一个别人仓库的副本副本

    - 拉取请求

      在fork之后可以在自己的仓库上看到pull request按钮点击即可发送

    - SSH免密登录

      之前采用的链接是使用https链接实际上可以使用SSH链接，使用SSH链接可以无需登录，但是默认自己账号是没有SSH密钥的需要设置一个密钥

      - 设置密钥步骤

        - 找到当前电脑用户下是否有.ssh目录，如果没有则在用户目录下打开git bash生成

          ```shell
          ssh-keygen -t rsa -C 密钥描述
          ```

        - 创建之后打开该目录其中包含两个文件，id_rsa是私钥，id_rsa.pub是公钥，打开公钥文件并复制其中内容

        - 打开GIThub官方登录账号打开用户菜单栏中的settings找到SSH and GPG keys并添加该SSH密钥

- ### 在IDEA中使用GIT

  - 环境准备

    - 配置git忽略文件(git.ignore)

      如果使用IDEA开发，IDEA会为我们的项目创建一个.idea文件夹，该文件夹是与项目无关的，可以使用git忽略文件忽略这些文件

      - 编写.ignore文件

        ```ignore
        #如果不想要所有xml文件直接编写即可
        *.xml
        ```

      - 在git.config中引用该文件

        ```config
        [core]
        	excludesfile = .ignore文件路径
        ```

  - 