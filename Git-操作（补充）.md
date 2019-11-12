## cherry-pick

*cherry-pick* 可以将分支 *B* 的任何**一个** *commit* 合并到分支 *A*。

**情景**：*dev* 分支有两个 *commit*，分别是：创建 *b.txt* 文件 *commitB*、创建 *c.txt* 文件 *commitC*。现在我们只想要合并 *commitAB 到 *master* 分支。下面是 *master* 和 *dev* 两个分支的情况：

![image.png](http://upload-images.jianshu.io/upload_images/659084-049b7c95c62a3131.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](http://upload-images.jianshu.io/upload_images/659084-d46ecad032fee3ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果直接用 *git merge* 会将 *commitC* 也合并到 *master*，这不是我们想要的结果。这时候，*cherry-pick* 就派上用场了。用法如下：

- 切换到目标分支。
- *cherry-pick* 待合并的 *commitID*。

上面的情景，只需要：

```powershell
git checkout master
git cherry-pick 8f2aa26
```

结果如下：

![image.png](http://upload-images.jianshu.io/upload_images/659084-4c396adb0bb8588a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不过，*cherry-pick* 只能合并单个指定的 *commit*。☞ [cherry-pick 更多操作](http://git-scm.com/docs/git-cherry-pick)

## rebase

*rebase变基* 是 *Git* 中整合来自不同分支修改的一种操作。（另一种是 *merge*）

**情景1**：*dev* 分支有两个 *commit* ，分别是：创建 *d.txt* 文件 *commitD*、创建 *e.txt* 文件 *commitE*。现在我们想要将 *dev* 分支的所有 *commit* 合并到 *master*。下面是 *dev* 分支的情况：

![image.png](http://upload-images.jianshu.io/upload_images/659084-293e36b78fb47d5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

想要通过 *rebase* 变基将 *commit* 合并到 *master*，只需要：

- 将 *dev* 分支变基到 *master* 分支。
- 切换到 *master* 分支。
- 执行 *merge* 操作。

```powershell
git rebase master
git checkout master 
git merge --no-ff dev
```

最后结果如下：

![1570868576457.png](http://upload-images.jianshu.io/upload_images/659084-0417d63fc36aef1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过 *rebase* 合并分支跟 *merge* 合并是不一样的。☞ [点击查看区别]([http://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA](http://git-scm.com/book/zh/v2/Git-分支-变基))

**情景2**：*dev* 分支上有四个 *commit*，顺序分别是：创建 *f.txt* 文件 *commitF*、增加 *f.txt* 文件内容 *commitF1*、创建 *g.txt* 文件 *commitG*、增加 *g.txt* 文件内容 *commitG1*。现在只想要将 *dev* 分支上关于文件 *f.txt* 的所有操作合并到 *master* 分支。下面是 *dev* 分支的情况：

![image.png](http://upload-images.jianshu.io/upload_images/659084-b9f954b059404b15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在只想要将 *dev* 分支上关于文件 *f.txt* 的所有操作合并到 *master* 分支，需要：

- 以 *f.txt* 的最后一个修改的 *commitID* 新建一个分支 *dev-temp*。
- 将以 *dev-temp* 分支上关于 *f.txt* 的第一个修改的 *commitID* *rabase* 变基到 *master* 分支。
- 切换到 *master* 分支。
- *merge* *dev-temp* 分支到 *master* 分支。

```powershell
git checkout -b dev-temp 884c858
git rebase --onto master 3b0bfd7^
git checkout master 
git merge --no-ff dev-temp
```

![image.png](http://upload-images.jianshu.io/upload_images/659084-e52de0c710b4437c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**情景3**：*dev* 分支上有四个 *commit*，顺序分别是：创建 *i.txt* 文件 *commitI*、增加 *i.txt* 文件内容 *commitI1*、创建 *j.txt* 文件 *commitJ*、增加 *j.txt* 文件内容 *commitJ1*。现在只想要将 *dev* 分支上关于文件 *j.txt* 的所有操作合并到 *master* 分支。下面是 *dev* 分支的情况：（其实跟上面的情况和操作一样）

![image.png](http://upload-images.jianshu.io/upload_images/659084-59f2f8f60f66ef93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在只想要将 *dev* 分支上关于文件 *j.txt* 的所有操作合并到 *master* 分支，需要：

- 以 *j.txt* 的最后一个修改的 *commitID* 新建一个分支 *dev-temp1*。
- 将以 *dev-temp1* 分支上关于 *j.txt* 的第一个修改的 *commitID* *rabase* 变基到 *master* 分支。
- 切换到 *master* 分支。
- *merge* *dev-temp1* 分支到 *master* 分支。

```powershell
git checkout -b dev-temp1 54ea4bd
git rebase --onto master e31249b^
git checkout master 
git merge --no-ff dev-temp1
```

![image.png](http://upload-images.jianshu.io/upload_images/659084-8507dd6818db90d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[rebase更多操作]([http://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA](http://git-scm.com/book/zh/v2/Git-分支-变基))

## 合并多个 commit 为一个 commit

合并多个 *commit* 为一个 *commit* 同样需要 *rebase*。

如下图：要将最新的两个 *commit* 合并为一个 *commit*。具体操作如下：

- 找到这三个 *commit* 的前一个 *commit* 的 *commitID*，执行 `git rebase -i commitID`

- 这时候会进入 *vi* 编辑。

  - `pick` 的意思是要会执行这个 *commit*
  - `squash` 的意思是这个 commit 会被合并到前一个 *commit*

- 之后会进入 *vi* 编辑 *commit* 信息。只需要将原先的 *commit* 信息注释，在顶部添加新的 *commit* 信息。

- 完成。

  ![image.png](http://upload-images.jianshu.io/upload_images/659084-a8a97ecd1b5c4901.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  ![image.png](http://upload-images.jianshu.io/upload_images/659084-5528059b7ca9273d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  ![image.png](http://upload-images.jianshu.io/upload_images/659084-d173423d001b4e65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  ![image.png](http://upload-images.jianshu.io/upload_images/659084-d7e5d016bfa4b80b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  ![image.png](http://upload-images.jianshu.io/upload_images/659084-cb502bcc195b4fba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 删除分支

删除分支包括删除本地分支和远程分支：

- 删除本地分支
  - 如本地分支不是打开状态，则：`git branch -d branchName`
  - 如本地分支是打开状态，则：`git branch -D branchName`
- 删除远程分支
  - `git push origin --delete branchName`

## 恢复分支

恢复分支，我们只需要以之前分支的 *commitID* 新开一个分支即可。

![image.png](http://upload-images.jianshu.io/upload_images/659084-a2e8638e3cc8a3ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果不记得 *commitID*，可以通过 *git reflog* 进行查看。

![1570871432663.png](http://upload-images.jianshu.io/upload_images/659084-ba394e5b699ff0f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 撤销本地修改

- 撤销指定本地文件的修改（未提交状态）：`git checkout -- fileName`

  ![image.png](http://upload-images.jianshu.io/upload_images/659084-75f7a0a74ac422d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 撤销本地的所有修改（未提交状态）：`git reset --hard`

  ![image.png](http://upload-images.jianshu.io/upload_images/659084-0359a8aee6d9ae94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 其他基本操作

- 生成本地的 *key*，并添加到远程仓库

```
ssh-keygen -t rsa
```

- 初始化本地目录

```
git init
```

- 添加到本地仓库

```
git add filename
```

- 提交到仓库（*git* 提交的是修改）

```
git commit -m "description"
```

- 查看仓库状态

```
git status
```

- 查看不同点

```
git diff filename
```

- 版本回退

```
git reset --hard
```

- 第一分支 *master* 创建分支 *v1* （*HEAD* 指向当前分支）

```
// 创建并指向新分支
git checkout -b v1 
// 分步执行
git branch v1
git checkouut v1
```

- 合并到当前分支

```
git merge v1
```

- 删除指定分支

```
git branch -d v1
```

- 指定分支将远程仓库的代码拉到本地

```
git clone gitAddress -b branchName
```

- 将本地的代码提交到远程仓库

```
git pull    //获取最新的更改
git add modefied_filename
git commit -m "description"
git push  // 推送
```

- 冲突解决
  可以用上面的方法解决就解决，不行的话就用下面的方法：
  - 用远程仓库的代码完全覆盖本地代码。

```
git reset --hard // 版本回退
git pull

```

- 修改 commit 提交的内容

```
git commit --amend

```

- 打印 log 数量

```
git log -p -3

```

- 文件被添加到暂存区，撤销该文件到工作目录

```
git reset HEAD fliename

```

- 工作目的删除文件后，还有在暂存区中删除该文件

```
rm filename
git rm filename

```

- 撤销对当前文件的修改，恢复到上一次快照的状态

```
git checkout -- filename

```

- *HEAD* 指针指向当前的分支，提交后都会快进（*fast-forward* ：指针右移）。
- *rebase* 合并分支
  变基是将一系列提交按照原有次序依次应用到另一分支上，而合并是把最终结果合在一起。使得提交历史更加整洁。 你在查看一个经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的，但它们看上去就像是串行的一样，提交历史是一条直线没有分叉。

```
git checkout dev
......修改......
git add filename
git commit -m "......"
git rebase master
git checkout master
git merge dev

```
