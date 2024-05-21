---------------git原理
rep/.git/objects# ls
4f/ 63/ 7e/ 82/ ab/ f5/ info/ pack/

commit1:sha1Id ->
	提交的备注
	提交者信息
	tree:sha1Id ->
		blob:根据my.txt内容得到的(my.txt) ->
			file content
commit2:sha1Id ->
    提交的备注
    提交者信息
	parent: commit1
    tree:sha1Id ->
        blob:根据my.txt内容得到的(my.txt) ->
            file content

分支信息
HEADS(ref: refs/heads/master) ->
	refs/heads/master: 82a6exxxxxxxxxxxxxxxxxxxxxxxxxx ->
		82a6exxxxxxxxxxxxxxxxxxxxxxxxxx:commits ...
工作区
working directory：操作系统上的文件，所有的代码开发编辑都在这上面完成
	a.txt: 111
	b.txt: 222
index or staging area:暂存区域，这里的代码下一次commit时将加入到提交到git仓库
	HEADS
git repository:由git object记录着每一次提交的快照，以及链式结构的提交变更历史
流程：
	1.echo 333 > a.txt
	1.1修改工作区中的a.txt，内容变成:333
	2.git add a.txt
	2.1.将a.txt添加到git仓库，生成一个新的blob节点(sha1值,blob类型,内容)
	2.2.更新index区中的索引，让其指向新的blob
	3.git commit -m 'update'
	3.1.git先根据当前的索引产生一个tree object,充当新提交的一个快照
	3.2.创建一个新的commit object,将这次commit的信息存储起来，并且parent指向一个commit,组成一条链记录变更历史
	3.3.将master分支的指针移到新的commit节点(修改master指针的值)

rep/.git/objects# git cat-file -t 63401bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
commit
rep/.git/objects# git cat-file -p 63401bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
tree 7e85ffxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
author jcluo <jcluo@example.com> 1715755308 +0800
committer jcluo <jcluo@example.com> 1715755308 +0800
add 3 and 4

rep/.git/objects# git cat-file -t 7e85ffxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
tree
rep/.git/objects# git cat-file -p 7e85ffxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
100644 blob 4f142exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  my.txt
rep/.git/objects# git cat-file -t 4f142exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
blob
rep/.git/objects# git cat-file -p 4f142exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
111
222

rep/.git/objects# git cat-file -t 82a6exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
commit
rep/.git/objects# git cat-file -p 82a6exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
tree abf27xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
parent 63401bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
author jcluo <jcluo@example.com> 1715755308 +0800
committer jcluo <jcluo@example.com> 1715755308 +0800
add 3 and 4

rep/.git/objects# git cat-file -t abf27xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
tree
rep/.git/objects# git cat-file -p abf27xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
100644 blob f58a3xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  my.txt
rep/.git/objects# git cat-file -t f58a3xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
blob
rep/.git/objects# git cat-file -p f58a3xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
111
222
333

########checkout:可以理解为拿出来查看
当我们 git checkout 一个分支或提交时（会更新所有的三棵树，使其和commit状态保持一致）：
1.从 repository 到 暂存区域
它会修改 HEAD 指向新的分支引用或提交，将暂存区填充为该次提交的文件快照
2.从 暂存区域 到 工作区
将暂存区的内容解包复制到工作区中
若工作区与暂存区存在未提交的本地修改，checkout还会尝试将文件快照与本地更改做简单的合并，若合并失败，将会终止操作并恢复到checkout之前的状态。因此checkout对工作区是安全的，它不会丢弃工作区做得更改

-----------------git使用
git init  //在当前目录新建一个git代码库
git clone [url] //下载一个项目和它整个代码历史

//git配置文件为.gitconfig，它可以在用户主目录下(全局配置)，也可以在项目目录下（项目配置）
git config --list //显示当前的git配置
git config -e [--global] // 编辑git配置文件
//提交代码时的用户信息
git config [--global] user.name "[name]" 
git config [--global] user.email "[email.address]" 

git add [file1] [file2] ...  //添加files到暂存区
git add [dir]  //添加dir（包括子目录）到暂存区
git add .  //添加当前目录的所有文件到暂存区
git rm [file1] [file2] ...  //删除工作区files，并将这次删除放入暂存区
git mv [file-original] [file-renamed] //更改文件名，并将这次改名放入暂存区

git commit -m  [commit.message] //提交暂存区到仓库

git branch //列出所有的本地分支，当前分支会加上星号*并高亮显示
git branch -r //列出所有的远程分支
git branch -a //列出所有的本地和远程分支
git branch -M [<oldbranch>] <newbranch> //将oldbranch(默认为当前分支)，修改为newbranch
	git branch -M main

git remote -v //显示所有远程仓库
git remote add <shortname> <URL> //为在<url>的仓库增加一个远程名<shortname>，这个<shortname>能够用于 create and update remote-tracking branches <shortname>/<branch>
	git remote add origin git@github.com:jensenluo1/ver1.git
	git push -u origin main
git push <remote> <branch> //上传本地指定分支到远程仓库
git pull <remote> <branch> //取回远程仓库的变化，并与本地分支合并
	
我在本地修改了一些文件还未提交，但我想放弃某些文件的更改。
我不小心 git add 了错误的文件，现在我不想把它和其他文件一起提交了。
我刚刚执行的提交添加了不该提交的文件，我想取消这次提交，但保留（或不保留）对本地文件所作的修改。
我不小心在一个提交中引入了 Bug 并且还推送到了远程分支，现在想回滚到原来的状态

git checkout [file] //恢复暂存区的指定文件到工作区
git checkout [file] //恢复暂存区的指定文件到工作区
git checkout <branch> //切换到指定分支，恢复该分支到工作区
git checkout -b <branch> //新建一个分支，恢复该分支到工作区
git checkout . //恢复暂存区的所有文件到工作区
