# git学习
## 将代码上传到GitHub通常涉及以下步骤：

1. **在本地初始化 Git 仓库：**  
+ 打开命令行，导航到您的项目目录。
运行以下命令初始化 Git 仓库：
```
git init
```

2. **将文件添加到仓库：**   
+ 使用以下命令将所有文件添加到暂存区：
```bash
git add .
```
3. **使用以下命令提交更改：**  
+  使用以下命令提交更改：
  ```bash
  git commit -m "Initial commit"
```

4. **将本地仓库关联到 GitHub 仓库：** 
+ 在 GitHub 上创建的仓库页面中，复制仓库的 HTTPS 或 SSH URL。   
+ 在命令行中运行以下命令，将本地仓库与 GitHub 仓库关联：
```bash
git remote add origin <GitHub-Repository-URL>
```
5. **推送到 GitHub：** 
+ 使用以下命令将代码推送到 GitHub：
```bash
git push -u origin master
```
**如果是首次推送，可能需要输入 GitHub 用户名和密码。**  

---
## git的工作流程
</div><img src = https://www.runoob.com/wp-content/uploads/2015/02/git-command.jpg></div>

- **workspace：** 工作区：就是你在电脑里能看到的目录。
- **staging area：** 暂存区/缓存区：英文叫 stage 或 index。一般存放在 .git 目录下的 index 文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。
- **local repository：** 版本库或本地仓库:工作区有一个隐藏目录 .git，这个不算工作区，而是 Git 的版本库
-  **remote repository：** 远程仓库
---
## git分支
几乎每一种版本控制系统都以某种形式支持分支，一个分支代表一条独立的开发线。

使用分支意味着你可以从开发主线上分离开来，然后在不影响主线的同时继续工作。
```shell
git branch <分支名字>
```
切换分支命令:
```shell
git checkout <branchname>
```
列出分支
```shell
git branch
```

---

## Snapshots

A snapshot is the top-level tree that is being tracked. For example, we might have a tree as follows:

```shell
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

## Modeling history: relating snapshots

How should a version control system relate snapshots? One simple model would be to have a linear history. A history would be a list of snapshots in time-order. For many reasons, Git doesn’t use a simple model like this.

In Git, a history is a directed acyclic graph (DAG) of snapshots. That may sound like a fancy math word, but don’t be intimidated. All this means is that each snapshot in Git refers to a set of “parents”, the snapshots that preceded it. It’s a set of parents rather than a single parent (as would be the case in a linear history) because a snapshot might descend from multiple parents, for example, due to combining (merging) two parallel branches of development.

Git calls these snapshots “commit”s. Visualizing a commit history might look something like this:

```shell
o <-- o <-- o <-- o
            ^
             \
              --- o <-- o
```

In the ASCII art above, the `o`s correspond to individual commits (snapshots). The arrows point to the parent of each commit (it’s a “comes before” relation, not “comes after”). After the third commit, the history branches into two separate branches. This might correspond to, for example, two separate features being developed in parallel, independently from each other. In the future, these branches may be merged to create a new snapshot that incorporates both of the features, producing a new history that looks like this, with the newly created merge commit shown in bold:

```shell
o <-- o <-- o <-- o <---- o
            ^            /
             \          v
              --- o <-- o
```

Commits in Git are immutable. This doesn’t mean that mistakes can’t be corrected, however; it’s just that “edits” to the commit history are actually creating entirely new commits, and references (see below) are updated to point to the new ones.

## Data model, as pseudocode

It may be instructive to see Git’s data model written down in pseudocode:

```shell
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | blob>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

## Objects and content-addressing

An “object” is a blob, tree, or commit:

```shell
type object = blob | tree | commit
```

In Git data store, all objects are content-addressed by their [SHA-1 hash](https://en.wikipedia.org/wiki/SHA-1).(**SHA-1 哈希值确保了对象内容的完整性。如果对象的内容发生了任何变化，即使是一个字符，其哈希值也会发生显著变化**)

```shell
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blobs, trees, and commits are unified in this way: they are all objects. When they reference other objects, they don’t actually *contain* them in their on-disk representation, but have a reference to them by their hash.

For example, the tree for the example directory structure [above](https://missing.csail.mit.edu/2020/version-control/#snapshots) (visualized using `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`), looks like this:

```shell
//解释以下git cat-file -p 
//git cat-file 是一个用于查看存储在 Git 仓库中的对象（如 blob、tree、commit 或 tag）内容的命令行工具。-p（print）选项用于打印对象的内容，并且是 git cat-file 命令最常用的选项之一。
git cat-file -p <blob_hash>
```



```shell
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

The tree itself contains pointers to its contents, `baz.txt` (a blob) and `foo` (a tree). If we look at the contents addressed by the hash corresponding to baz.txt with `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`, we get the following:

```shell
git is wonderful
```

---

## References

Now, all snapshots can be identified by their SHA-1 hashes. That’s inconvenient, because humans aren’t good at remembering strings of 40 hexadecimal characters.

Git’s solution to this problem is human-readable names for SHA-1 hashes, called “references”. References are pointers to commits. Unlike objects, which are immutable, references are mutable (can be updated to point to a new commit). For example, the `master` reference usually points to the latest commit in the main branch of development.

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

# Staging area

This is another concept that’s orthogonal to the data model, but it’s a part of the interface to create commits.

One way you might imagine implementing snapshotting as described above is to have a “create snapshot” command that creates a new snapshot based on the *current state* of the working directory. Some version control tools work like this, but not Git. We want clean snapshots, and it might not always be ideal to make a snapshot from the current state. For example, imagine a scenario where you’ve implemented two separate features, and you want to create two separate commits, where the first introduces the first feature, and the next introduces the second feature. Or imagine a scenario where you have debugging print statements added all over your code, along with a bugfix; you want to commit the bugfix while discarding all the print statements.

Git accommodates such scenarios by allowing you to specify which modifications should be included in the next snapshot through a mechanism called the “staging area”.

---

# Git 工作流程

本章节我们将为大家介绍 Git 的工作流程。

一般工作流程如下：

- 克隆 Git 资源作为工作目录。
- 在克隆的资源上添加或修改文件。
- 如果其他人修改了，你可以更新资源。
- 在提交前查看修改。
- 提交修改。
- 在修改完成后，如果发现错误，可以撤回提交并再次修改并提交。

下图展示了 Git 的工作流程：

![img](https://www.runoob.com/wp-content/uploads/2015/02/git-process.png)

# git push -u origin main

`git push -u origin main` 是一个 Git 命令，用于将本地的 `main` 分支推送到远程仓库 `origin`。下面是该命令各部分的详细解释：

- `git push`：Git 命令，用于将本地分支的更改推送到远程仓库。
- `-u`：这个选项告诉 Git 在推送过程中同时设置上游（tracking）信息。这意味着之后你可以简单地使用 `git push` 来推送该分支，而不需要每次都指定远程仓库和分支名。`-u` 是 `--set-upstream` 的简写。
- `origin`：这是远程仓库的默认短名称。在第一次使用 `git remote add` 添加远程仓库时，通常使用 `origin` 作为远程仓库的别名。
- `main`：这是你想要推送的本地分支的名称。在许多新的 Git 仓库中，主分支通常被命名为 `main`，这是 `master` 这一传统名称的替代，后者在一些项目中出于尊重和包容的考虑而被更改。

当你运行 `git push -u origin main` 时，Git 会执行以下操作：

1. 将本地 `main` 分支的工作成果打包成一个“commit 集合”。
2. 将这个集合发送到你指定的远程仓库 `origin`。
3. 如果远程仓库中没有对应的 `main` 分支，Git 会创建它，并将其设置为跟踪你的本地 `main` 分支。
4. 如果 `-u` 选项被使用，Git 会记录 `origin` 仓库的 `main` 分支作为本地 `main` 分支的上游分支，这样你就可以使用 `git push` 或 `git pull` 而无需指定远程仓库和分支。

如果 `main` 分支之前已经推送过，并且你已经设置了上游跟踪，那么你可以使用更简短的命令 `git push`，Git 会根据之前的设置知道将更改推送到哪个远程分支。

请注意，如果你的远程仓库使用的是 SSH 认证方式，确保你的 SSH 密钥已经添加到远程仓库托管服务（如 GitHub 或 GitLab）的账户中，并且 SSH 客户端已正确配置。如果是使用 HTTPS 认证方式，确保你的认证信息（如用户名和密码或个人访问令牌）是最新的。
