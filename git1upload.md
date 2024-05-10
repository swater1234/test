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

## Objects and content-addressing

An “object” is a blob, tree, or commit:

```
type object = blob | tree | commit
```

In Git data store, all objects are content-addressed by their [SHA-1 hash](https://en.wikipedia.org/wiki/SHA-1).

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blobs, trees, and commits are unified in this way: they are all objects. When they reference other objects, they don’t actually *contain* them in their on-disk representation, but have a reference to them by their hash.

For example, the tree for the example directory structure [above](https://missing.csail.mit.edu/2020/version-control/#snapshots) (visualized using `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`), looks like this:

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

The tree itself contains pointers to its contents, `baz.txt` (a blob) and `foo` (a tree). If we look at the contents addressed by the hash corresponding to baz.txt with `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`, we get the following:

```
git is wonderful
```





