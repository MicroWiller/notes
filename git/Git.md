
## Git特点

<img src="Git.assets/image-20200714135525033.png" alt="image-20200714135525033" style="zoom:67%;" />



## 配置

```git
git config --global user.name 'your_name'
git config --global user.email 'your_email'
```

<img src="Git.assets/image-20200714140912532.png" alt="image-20200714140912532" style="zoom:67%;" />





## Git结构图



![](Git.assets/20200306122919034_15244.png )





## Git命令



![](Git.assets/20200306123030411_6765.png )





### 建Git仓库

1. 将已有的项目代码纳入Git管理

   ```git
   cd 项目所在的文件夹
   git init
   ```

2. 新建的项目直接用Git管理

   ```git
   cd 某个文件夹
   git init your_project	# 会在当前路径下创建和项目名称同名的文件夹
   cd your_project
   ```

初步提交案例：

<img src="Git.assets/image-20200714144049870.png" alt="image-20200714144049870" style="zoom:67%;" />





### 工作区和暂存区

<img src="Git.assets/image-20200714150438128.png" alt="image-20200714150438128" style="zoom:67%;" />

```git
git add -u
# 直接把工作区当中还未 添加到暂存区的文件 全部添加到暂存区
# 不用后面一个一个的写准备添加的文件
```

```git
git commit -am'your_commit_message'    # 直接提交，不到暂存区
```



### 文件重命名

- 笨重的方法：

  ```git
  mv readme readme.md
  git add readme.md
  git rm readme
  ```

- 直接使用git的命令来 重命名

  ```git
  git mv readme readme.md
  git commit -m'Move readme ro readme.md'
  ```

  

复原成原来的样子：

```git
git reset --hard
# 撤销工作目录中所有未提交文件的修改内容
# 暂存区和工作去上所以的变更 都会被清理掉
```





### 版本演变历史

```git
git log --oneline  		# 分支的演进历史
git log --oneline --all # 所有分支的演进历史
git log -n2 --oneline  # n2表示显示最近的两行
```

```git
git branch -v 			# 本地多少分支
```

```git
git log   				# 当前分支的历史
git log --all  			 # 显示所有分支对应commit的信息
git log --all --graph 	 # 图形化的显示所有分支信息
```





### 修改commit的message



```git
git commit --amend  # 修改最新commit的message
```

```git
git rebase -i father_CommitID   # father_CommitID 基于待改变commit的父亲
```



### 整理多个Commit

```git
git rebase -i father_CommitID   # father_CommitID 基于待改变commit的父亲
```

回车，**几个连续commit**，选择策略：

<img src="Git.assets/image-20200716173918458.png" alt="image-20200716173918458" style="zoom:50%;" />

几个间隔commit：将需要合并的CommitID号 放一块 并改动前面的策略

可能会出现：两个孤独的树

<img src="Git.assets/image-20200716175043601.png" alt="image-20200716175043601" style="zoom:80%;" />





### diff 

<img src="Git.assets/image-20200716175357646.png" alt="image-20200716175357646" style="zoom:80%;" />

```git
git diff --cached #暂存区和HEAD
git diff #暂存区和 工作区
git diff -- 文件名(可以是多个)
```

```git
# 比较temp和master两个分支的差异 【可以有commit_id】
git diff temp master
```



### 恢复和撤销

```git
# 撤销暂存区的add，恢复成和HEAD的一样
git reset HEAD  
```

![image-20200717160657051](Git.assets/image-20200717160657051.png)

```git
# 撤销部分暂存区的add
git reset HEAD --file_name
```



```git
# 将工作区的文件，恢复为之前add的暂存区
git checkout --file_name
```



```git
# 撤销最近的几次commit
# HEAD重新指向另外commit，同时暂存区和工作区的内容都要回到该commit的状态
git reset --hard point_commit_id
```





### 删除文件

```git
git rm file_name
```

<img src="Git.assets/image-20200717165237278.png" alt="image-20200717165237278" style="zoom:67%;" />



### 存放状态

```git
git stash
```

<img src="Git.assets/image-20200717165956966.png" alt="image-20200717165956966" style="zoom:74%;" />

```git
# 恢复 之前的状态
git stash apply  # stash对应的状态没有清除
git stash pop    # stash对应的状态被清除
```







## .git目录

隐藏文件夹 **.git** 是`版本库` 

<img src="Git.assets/image-20200715155550366.png" alt="image-20200715155550366" style="zoom:67%;" />

**HEAD是一个引用文件**，正工作在哪个分支上，最终指向一个commit

<img src="Git.assets/image-20200715155855305.png" alt="image-20200715155855305" style="zoom:60%;" />

```git
git checkout <branch/tag>  # 切换到指定分支或标签
git diff 两个commit号
git diff HEAD HEAD^1       # ^ 表示head的父亲
```

 **config**存放了本地仓库的配置信息

 **refs**	：

<img src="Git.assets/image-20200715161034113.png" alt="image-20200715161034113" style="zoom:67%;" />

<img src="Git.assets/image-20200715161608275.png" alt="image-20200715161608275" style="zoom:67%;" />

**objects**：	

<img src="Git.assets/image-20200715162543216.png" alt="image-20200715162543216" style="zoom:67%;" />





## .gitignore

<img src="Git.assets/image-20200717171022414.png" alt="image-20200717171022414" style="zoom:80%;" />







## 三种类型

<img src="Git.assets/image-20200715163216455.png" alt="image-20200715163216455" style="zoom:50%;" />

commit、tree、blob(文件对象)

- **一个commit对应一颗树**，树代表了该commit的视图？？，视图存放了快照，快照的集合里面放了当前commit对应的本项目仓库的所有的文件夹以及文件的一个快照
- 快照：某一个时间点，这个文件夹长什么样？？文件是什么样？？
- tree：类比文件夹
- blob跟文件名**没有关系** ，只要文件的内容相同，在git眼里它就是唯一的blob



<img src="Git.assets/image-20200715182517792.png" alt="image-20200715182517792" style="zoom:60%;" />





## 分离头指针

> 分离头指针，变更没有基于任何 branch 去做；
>

HEAD 没有**绑定** 任何<u>分支和标签</u>： 

<img src="Git.assets/image-20200715183925160.png" alt="image-20200715183925160" style="zoom:67%;" />

- 分支切换的时候，分离头指针上的 commit 很容易被当做垃圾清掉

修复方法： 

<img src="Git.assets/image-20200715184621678.png" alt="image-20200715184621678" style="zoom:70%;" />





## 分支

1. HEAD指针严格来说不是指向提交，而是指向master，`master才指向提交`，所以，HEAD指向的就是当前分支。
![](Git.assets/20200306120739933_2876.png )

2.  当我们创建新的分支，例如dev，Git新建一个指针叫dev，指向master相同的提交，再把HEAD指向dev，就表示`当前分支`在dev上：
![](Git.assets/20200306121028163_3834.png )

3. 从现在开始，对工作区的修改和提交就是`针对dev分支`，比如新提交一次后，dev指针往前移动一步，而master指针不变：
![](Git.assets/20200306121154123_14764.png )

4. 把master指向dev的当前提交，就完成了合并：
![](Git.assets/20200306121321326_7039.png )

5. 合并完分支后，甚至可以删除dev分支。就剩下了一条master分支：
![](Git.assets/20200306121408744_1342.png )

6. 多个分支可能造成`冲突`，不同分支可能都做了不同的修改



## 传输协议



<img src="Git.assets/image-20200717171454060.png" alt="image-20200717171454060" style="zoom:67%;" />

区别：

<img src="Git.assets/image-20200717171619807.png" alt="image-20200717171619807" style="zoom:50%;" />
