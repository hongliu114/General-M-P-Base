参考教程:
[改变了世界的软件！程序员的基本功，Git 应该如何使用？](https://www.bilibili.com/video/BV1u94y1n73L?vd_source=34dd435d3d651e8a3a60eacd5873aed9)
[纯享Git小动画！轻松看懂git命令](https://www.bilibili.com/video/BV1Ev4y1L7dh?vd_source=34dd435d3d651e8a3a60eacd5873aed9)
## Git基本操作
### 建立仓库
进入Git bash,并初始化基本信息

	git config --global user.name [……]  初始化用户名
	git config --global user.email [……]  初始化邮箱
#### 1 从零开始建立仓库

	git init  git初始化
	git remote add origin [网页]
	git push -u origin master
#### 2 直接clone仓库到本地

	git clone [网址]：将仓库快捷下载到本地
### 基本操作

	git add [文件名]  添加文件到暂存区
		git add .  把所有文件添加到暂存区
	git commit  提交文件到仓库
		-m "[备注]" 给提交添加备注
	git push
	git pull
	git log  查看提交日志
	git status（状态）
## Bug实录
#### Git clone报错：ERROR: RPC failed;
[应对方法](https://blog.csdn.net/JISOOLUO/article/details/103625488)
#### fatal: The remote end hung up unexpectedly
原因：有文件过大
处理方式：用.gitignore排除或者删除文件
参考链接：[Git忽略文件的几种方法，以及.gitignore文件的忽略规则](https://blog.csdn.net/qq_41428418/article/details/132737503)
> [!tip]
> 若处理后还是提示这些文件报错，可能是Git 的历史记录中仍然保留了大文件的记录；我采用的办法是重建git仓库（简单粗暴），据AI所述还有git filter-repo等彻底删除大文件的方法，没试过。

