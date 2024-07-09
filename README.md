# GitTutorials


## git area（重要）

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

## git 撤销操作


### 使用场景 1

当你在工作区，写了若干代码，但是你发现，这些代码没用，需要将这些修改撤销，但是修改的文件较多不可能一个一个文件手动撤销，这时你可以使用如下命令：

```
// 如果文件是新增的，处于 untracked（撤销所有）
git clean -f   // 如果有目录可以加上 -d

// 如果文件是新增的（撤销指定）
git clean -f filenames

// 如果是在已经存在文件基础上修改（撤销所有）
git checkout .

// 如果是在已经存在文件基础上修改（撤销指定文件）
git checkout filenames // 撤销指定文件
git restore filenames // git 2.23.0 新增命令
```

上面的命令只会撤销 `workspace area` 的修改。

### 使用场景 2

当你写了若干代码后，执行了 git add（或 IDE 设置了自动 git add），那么文件的处于 tracked 状态，修改内容在 stage area(index area) 区域中

此时你想撤销修改，但是修改的内容还想保留到工作区（仅仅从 stage area 撤销，即变成 untracked 状态），可以使用如下命令：

```
git reset  // 针对所有修改
git reset filenames  // 指定具体文件
git restore --staged filenames  // 指定具体文件
```

如果撤销时不想保留工作区，可以使用：

```
// 不管是否是新建的文件、文件夹会一并被撤销
git checkout -f // 无法指定文件名
```

### 使用场景3

当你的修改已经 commit（local repository），此时想要撤销怎么办？可以使用如下命令：

```
// 撤销后，修改会保留在 staging area 和 workspace area
git reset --soft HEAD~1

// 撤销后，修改会保留在 workspace area
git reset --mixed HEAD~1   // 等同 git reset HEAD~1

// 同时撤销在 `local repository` 和 `staging area`、`workspace area` 的修改：
git reset --hard HEAD~1  // 使用的时候需要注意
```

> HEAD~1 表示回到当前(HEAD) commit 记录的上 1 个 commit.

### git reset VS git revert

`git revert`，它不会撤销原来的 commit 记录，而是新增一条 commit 记录。

`git revert commit_id`，会在该 commit_id 的基础上做反操作，例如，添加一行，他就会删除该行。

使用时也可以使用 `HEAD~`，如：
```
git revert HEAD~1
```

git revert 还可以撤销中间的某个 commit，其他的 commit 保持不变（git reset 则做不到这一点），如：

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

reset 和 revert 使用场景的异同：

1. 如果撤销的是你个人的分支（分支只有你一个人修改），可以使用 git reset，这样 git 记录也比较简洁
2. 如果你需要撤销 git 记录中间的某条 commit，可以使用 git revert
3. 如果撤销的是公共分支 （其他正在使用），则建议使用 git revert。如果使用 git reset 的话，当你 push 的时候后，其他人 pull，其本地记录可能会乱。



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
- **删除已经push的大文件**
  
  `python3 -m pip install git-filter-repo` （git 官方推荐）
   按照文件大小升序排列并取最后40个文件:
  ```
  git rev-list --objects --all | grep "$(git verify-pack -v .git/objects/pack/*.idx | sort -k 3 -n | tail -40 | awk '{print$1}')"
  ```
  打印文件大小(brew install coreutils)：
  ```
	git rev-list --objects --all \
	| git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
	| sed -n 's/^blob //p' \
	| sort --numeric-sort --key=2 \
	| tail -n 20 \
	| cut -c 1-12,41- \
	| $(command -v gnumfmt || echo numfmt) --field=2 --to=iec-i --suffix=B --padding=7 --round=nearest
  ```
  
  例如删除，已经 push 的 apk 文件
  ```
  git filter-repo --force --invert-paths --path-regex .+\.apk
  ```
  最后 push
  ```
  git push -f origin master
  ```
  > https://stackoverflow.com/questions/37937984/git-refusing-to-merge-unrelated-histories-on-rebase
    

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

