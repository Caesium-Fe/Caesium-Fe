<h1>Git手册</h1>

<h2>SVN和Git区别</h2>

svn：集中式版本控制系统，所有版本库都是集中放在中央服务器的，更新提交都是提交到中央服务器，所以需要联网才能工作，局域网和广域网都可以，但是会受到网速的影响。

git：分布式版本控制系统，每个用户的电脑都可以是一个完整的版本库，只需要相互之间推送修改就可以。

<h2>Git指令</h2>

git config --global 参数 “数据”  属于是全局配置即所有的版本库都是用这个配置

git init 会把当前目录变成git可以管理的仓库，然后这个目录下多出一个.git文件夹，该文件夹用来记录版本变更。

git add 文件名      将该文件添加到暂存区。

git commit 文件名     将存在暂存区的所有内容提交到当前分支上。

git status      用于查看是否还有文件未提交。

git diff 文件名     用于查看当前文件和上一个版本之前进行了什么修改。

git log 可以查看每次提交的版本号，修改的内容，附加的内容 -pretty=oneline设置为一行展示。

git reset --hard HEAD^     将版本回滚到上个版本  HEAD^^回滚到上两个版本， HEAD~100回滚到上100个版本。

git reset --hard 版本号      会退到明确的版本。

git reflog     可以用来查看该文件的所有版本号。搭配上一条命令，就可以会退到任何版本。

cat 文件名      用来查看文件内容。

git checkout -- 文件名        中的 -- 很重要，如果没有 -- 的话，那么命令变成创建分支了。撤销工作区上的所有修改，退回到上一个版本，若暂存区有文件的话，退回到暂存区版本。

git token 密钥 git remote add origin https://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@github.com/Caesium-Fe/markdownInJob.git

