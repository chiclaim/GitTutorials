# GitTutorials





## git 多用户

描述：在同一台机器上，我们可能需要配置多个用户，例如公司的项目，需要使用公司的 git 账户，但是有的时候需要维护 github 上开源的项目。提交代码的时候需要区分用户。

我们可以在不同的 repo 仓库使用 git config 来配置，如：

```
git config user.name "xxx"
git config user.email "xxx@xxx"
```

但是项目多了，就比较麻烦，而且容易忘记。可以使用 `includeIf` 来设置，步骤如下：

创建两个 `git-config` 文件：

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

