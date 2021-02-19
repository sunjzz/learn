## Git版本管理工具

常用命令：

  git config --global user.email "you@example.com" 
  git config --global user.name "Your Name"

- git init	# 初始化，让git管理当前文件夹

- git status 	# 检测当前目录下文件的状态

  红色：工作区 

  绿色：暂存区

- git add	#  工作区文件提交到暂存区

- git commit -m ‘’	# 暂存区文件提交到版本库

- git log	# 查看历史提交记录

- git reset --hard 版本号 	# 回滚版本号

- git reflog 	#  查看版本号动作记录

- git reset --soft 	# 版本库撤回至暂存区

- git reset HEAD 	# 暂存区撤回至工作区（修改文件后的状态）

  git reset HEAD 文件名 

- git checkout 	# 工作区撤回初始状态 （修改文件前的状态）

  git checkout -- 文件名

- git reset --mix 版本号	# 版本库撤回工作区

- git reset --hard 版本号 	# 版本库撤回工作区初始（修改文件前的状态） 

- git branch	# 当前分支

- git branch dev	# 创建一个dev分支

- git checkout dev 	# 切换到dev分支

  ```
git checkout -b dev # 创建dev分支并切换到dev分支
  ```

  

  合并分支：

  git checkout master

  git branch

  git merge bug 	# 合并bug分支到master分支

  git branch -d bug 	# 删除bug分支 
  
  git remote add origin https://github.com/..... 	# 添加远程git仓库地址别名
  
  git push -u origin master		# 本地master分支推送至origin
  
  git clone  
  
  git pull origin master 这一句等同于：
  
  ```
  git fetch origin master
  git merge origin/master
  ```
  
  git rebase -i HEAD~3 合并最近的3次提交记录
  
  git log 
  
  git log --graph 
  
  git log --graph --pretty=format:"%h %s"
  
  git rebase
  
  git pull 
  
  ```
  git fetch origin dev
  git rebase origin/dev
  ```
  
  
  
  git tag -a v1 -m "第一版"
  
  