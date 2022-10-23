# GitTutorials


## git area

git area 主要由以下几个：
- workspace area 也称之为 working tree
- index area 也称之为 staging area
- repository area 也称之为 local repository 

如下图所示：

![git areas](images/git-areas.png)



- `git add` 用于将文件内容添加到 `index`
- `git commit` 将修改记录到 `repository`

---

- `git ls-files` 显示 index area 或 working tree 的文件信息
- `git status` 显示 working tree 的状态

## git 撤销操作汇总

撤销 `workspace area` 的修改，可以使用：

```
git checkout filenames
git restore filenames // git 2.23.0 新增命令
```

撤销 `stage area` 的修改（会保留 workspace area，文件会变成 untracked 状态），可以使用：

```
git reset filenames
git restore --staged filenames
```

撤销 `local repository` 的修改，可以使用：

```
git reset --soft HEAD~1
```
> HEAD~1 表示回到当前(HEADER) commit 记录的上 1 个 commit.

同时撤销文件在 `local repository` 和 `staging area` 的修改：

```
git reset --mixed HEAD~1
// 等同
git reset HEAD~1
```

同时撤销文件在 `local repository` 和 `staging area`、`workspace area` 的修改：

```
git reset --hard HEAD~1  // 使用的时候需要注意
```


除了 `git reset` 还可以使用 `git revert`，它不会撤销原来的 commit 记录，而是新增一条记录。

```
git revert HEAD~1
```

git revert 还可以撤销中间的某个 commit，其他的 commit 保持不变（git reset 则很难做到这一点），如：

```
// 例如 commit 4 次，每次新增了一个文件，log 日志如下所示：
~$ git reflog
11584b6 (HEAD -> master) HEAD@{0}: commit: t4
947f0cc HEAD@{1}: commit: t3
8f3f8ee HEAD@{2}: commit: t2
8bc1c0b HEAD@{3}: commit (initial): t1
// revert 第三次 commit
~$ git revert 947f0cc
~$ git reflog
64eea6b (HEAD -> master) HEAD@{0}: revert: Revert "t3"
11584b6 HEAD@{1}: commit: t4
947f0cc HEAD@{2}: commit: t3
8f3f8ee HEAD@{3}: commit: t2
8bc1c0b HEAD@{4}: commit (initial): t1
~$ ls -l
total 0
-rw-r--r--  1 yuzhiqiang  staff  0 10 23 19:06 t1.txt
-rw-r--r--  1 yuzhiqiang  staff  0 10 23 19:06 t2.txt
-rw-r--r--  1 yuzhiqiang  staff  0 10 23 19:06 t4.txt
// 可以看到第三次提交的文件不见了
```

> 如果是个人分支如 feature，没有其他人修改，撤销时可以使用 git reset.
> 如果是公共分支，其他人也正在使用，如果使用 git reset，然后 push -f，那么其他人的本地 pull 的时候，他本地的记录可能就乱了。所以公共分支建议使用 git revert

## 删除操作汇总

- **git clean 删除 working tree 中的 untracked file**

  `git clean -f -d` 其中 -f(force)，-d(directory)    
   需要注意的是，该命令不能删除 tracked file，如果需要将丢弃所有最近未 commit 的内容
   可以先执行 `git checkout .` 然后执行 `git clean -f -d`


- **git rm 从 working tree 和 index 中移除文件**

    `git rm file` 文件会从 working tree 和 index 中移除
    如果仅仅从 index 中移除，但保留在 working tree，可以使用 `git rm --cache`，那么文件会保留(cache)在 working tree 中
    如果文件不在 index 中，即 untracked，那么 git rm 无法删除该文件
    例如：你想将某个已提交的目录(.idea)，加入到 ignore 中，那么你可以：`git rm --cached -r .idea`, 然后修改 .gitignore 文件，最后，commit, push
    

## git 多用户

描述：在同一台机器上，我们可能需要配置多个用户，例如公司的项目，需要使用公司的 git 账户，但是有的时候需要维护 github 上开源的项目。提交代码的时候需要区分用户。

我们可以在不同的 repo 仓库使用 git config 来配置，如：

```
git config user.name "xxx"
git config user.email "xxx@xxx"
```

但是项目多了，就比较麻烦，而且容易忘记。可以使用 `includeIf` 来设置，步骤如下：

创建两个 `git-config` 文件，一个用于公司项目 一个用于个人项目：

**.gitconfig-personal** 

```
[user]
	name = your_name
	email = your_email
```



**.gitconfig-work**
```
[user]
	name = work_nick_name
	email = your_work_email
```

然后在 `.gitconfig` 中指定某个文件夹（如 CompanyProjects）使用 `.gitconfig-work` 的配置，其他文件夹均使用 .gitconfig-personal 的配置：

```
[include]
    path = ~/.gitconfig-personal
[includeIf "gitdir:~/CompanyProjects/"]
    path = ~/.gitconfig-work
```

