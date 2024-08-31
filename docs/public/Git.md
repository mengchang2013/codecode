# GIT

## git分支
```shell
git remote -v #查看远程分支
git branch dev #创建分支
git checkout dev #切换分支
git checkout -		#切换到上一个分支 
git branch  #命令查看本地分支，列出所有分支，当前分支前面会标一个*号
git branch -r #所有远程分支 
git branch -a #所有分支，本地和远程 
git checkout -b <分支名称> #创建并切换分支
git branch -d <分支名称>		#删除分支（本地）
git push origin --delete <分支名称>		#删除远程分支
git push origin <分支名称> #推送分支到远程
git checkout -b new_branch_name origin/source_branch   #基于已有分支代码创建新的分支
```

## git stash
```shell
# save执行存储时，添加备注，方便查找, -u：添加未被git跟踪管理的文件
git stash save "save message" -u

# 查看stash了哪些存储
git stash list

# 应用某个存储
git stash apply stash@{$num}

# 丢弃stash@{$num}存储，从列表中删除这个存储
git stash drop stash@{$num}

# 删除所有缓存的stash
git stash clear

# 应用并删除缓存堆栈中的对应stash，默认为第一个stash,即stash@{0}
git stash pop
```



## git回滚

```shell
### 1、已修改，未提交暂存区，重置工作区内容为暂存区内容:
	git restore <file> 
	git checkout -- <file>   
	git checkout -- .

### 2、添加到暂存区，未提交到版本库; 即取消git add操作	
git reset HEAD -- <file> 
git reset HEAD -- . 
git restore --staged <file>

### 3、已经执行了commit命令
#### 方式一：参数为 需要回退到的版本号，不会产生新的提交记录
#①、查看操作记录
git reflog
#②、执行回滚,7335548=操作记录的前一个版本号，即需要回到的版本号
git reset --hard 7335548     
# 参数解析： 
#       --soft 仅移动本地库head指针
#       --hard 移动本地库指针，重置暂存区、工作区
#       --mixed 本地库移动指针，重置暂存区，默认参数
#③、强制推送远程服务
git push -f origin master

#### 方式二： 参数为需要回退的操作版本号,重新生成一条新的提交记录
git revert 7335548
```

## idea 使用git

### 解决Terminal使用vim时Esc无法退出问题  
**问题：**  
将Terminal配置为git bash。在Terminal工具窗口下执行git命令，包括vim操作。  
在Vim编辑模式下，按ESC键退出insert mode时，idea会隐藏整个Terminal工具窗口，并且光标自动定位到idea的编辑窗口，十分不方便。  
**解决：**  
```text
使用Ctrl+Alt+S快捷键打开idea全局配置，
在快捷键映射配置Settings -> keymap中，
找到 Plugins -> Terminal -> Switch Focus To Editor，删除其快捷键绑定即可。
```

