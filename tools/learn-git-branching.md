# 本地

## git commit

git仓库中的提交记录保存的是你目录下所有文件的快照

```
git commit
```

该语句意味着创建一个新的提交记录，父节点是我当前所在的节点，不过原分支也会移动到新的提交上

**不想要原来的提交记录，使用amend参数覆盖掉上一次的提交**

```
git commit --amend//上一次提交记录就不见了
```



## git branch

创建分支不会造成内存的开销

使用分支意味着：我想基于这个提交以及它所有的 parent 提交进行新的工作。

```
git branch <newBranch>
git branch -f <name>//强制指向该name分支
git branch -d <name>//删除name分支
```

## git checkout

切换分支

```
git checkout <name>
git checkout -b <name>//创建且切换到该name分支
```

## 合并分支

### git merge：

merge意味着：我要把这两个 parent 节点本身及它们所有的祖先都包含进来。

```
git merge <name>//把name合并到当前分支
把name和当前分支合并起来，然后当前分支指向合并后的新提交
```

### git rebase：

取出一系列的提交记录，“复制”它们，然后在另外一个地方逐个的放下去。

```
git rabase <name>//把当前分支合并到name
复制一个当前分支的提交记录，然后放到name分支下
```

ps：注意两个语句的作用对象

### rebase和merge的优缺点

优点:

- Rebase 使你的提交树变得很干净, 所有的提交都在一条线上

缺点:

- Rebase 修改了提交树的历史

## HEAD

HEAD 是一个对当前所在分支的符号引用 

可以使用git checkout <hash>,使得分离的HEAD指向具体的几条记录而不是分支名

## 相对引用

- 使用 `^` 向上移动 1 个提交记录

- 使用 `~<num>` 向上移动多个提交记录，如 `~3`

  ```
  git checkout <name>^^
  git checkout <name>~2
  //切换到name上两个节点
  ```

### ^后接数字

指定合并提交记录的某个 parent 提交，

使用git checkout HEAD^，如果有两个parent，git默认选择合并提交记录正上方那个提交记录

```
git checkout HEAD^2
```

ps:所有东西可以搭配使用

## 撤销变更

### git reset

将分支记录回退几个提交记录

```
git reset --hard HEAD~1//回退到上一个提交记录，该分支也会指向上一个提交记录
```

git reset对于远程分支无效，所以需要git revert

### git revert

创建一个新提交，这个新提交的内容就是撤销后的内容

```
git revert HEAD//撤销更改，回退到上一个提交记录
```

## 复制提交记录

### git cherry-pick

git cherry-pick后面需要跟提交id，也就是哈希

把提交记录复制到当前位置

```
git cherry-pick <id1> <id2>//把id1和id2对于提交记录复制到当前所在的分支
```

### git rebase -i

当不知道提交记录的哈希时使用它

```
git rebase -i HEAD~4//把前四个提交选择性复制到倒数第一个提交下
```

下图是忽略了c2后的结果

![image-20230703215425900](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20230703215425900.png)

## git tag

*永远*指向某个提交记录的标识，并不会随着新的提交而移动

```
git tag v1 c1//给c1创建一个标签v1
```

## git describe

描述离你最近的标签

```
git describe <ref>
```

<tag>_<numCommits>_g<hash>

tag：离ref最近的标签

numCommits：ref和tags相差多少个提交记录

hash：所给ref的提交记录哈希值的前几位

ps：如果ref上有标签就只输出标签，其他什么都不输出



# 远程

切换到远程分支的时候，会自动进入分离HEAD状态，因为git不允许直接操作远程分支

远程分支名字 origin/branchName

## git clone

克隆一个仓库

## git fetch

从远程仓库获取数据

无参数时会下载所有提交记录

```
git fetch origin <source>:<destination>见push
```

**完成的步骤：**

- 从远程仓库下载本地仓库中**缺失**的提交记录
- 更新远程分支指针(如 `o/main`)

`git fetch` 并不会改变你本地仓库的状态。它不会更新你的 `main` 分支，也不会修改你磁盘上的文件。

## git pull

git fetch + get merge <远程分支>

### git pull --rebase

git fetch + git rebase <远程分支>

ps：远程分支，是我便于理解写出来的，实际没有这个

## git push

```
git push origin main//main跟踪origin/main的情况

```

```
//如果本地和远程名字不一样的话
git push origin <source>:<destination>
//fetch也有这个语法，只不过source表示远程而已
//如果destination不存在，git还会再远程仓库创建一个分支
```

```
git push origin :foo//删除远程的foo分支
git fetch origin :foo//创建一个本地foo
```

如果不指定分支，就是用的HEAD所在的分支

将变更上传到指定的远程仓库,并在远程仓库上合并新提交记录

## 远程跟踪

clone时，git会给远程分支在本地仓库创建一个远程分支（如origin/main），然后创建一个跟踪远程仓库活动分支的本地分支（如main）

### 自行指定跟踪属性

用main2去跟踪origin/main的两个办法

```
git checkout -b main2 origin/main
```

```
git branch -u origin/main main2
```

## git log

显示仓库历史记录中的所有提交