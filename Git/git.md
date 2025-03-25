[TOC]

# 配置仓库的Commiter信息

> Git仓库需要配置作者的名字和邮箱，当我们在仓库Commit后，每一次的commit会关联配置的名字和邮箱，用来表示和追溯提交者的信息。
> 如果不配置名字和邮箱，Commit就不会包含作者和邮箱信息，这种Commit可能导致push到远程仓库时报错，因为不包含作者和邮箱信息或者邮箱不满足某种格式的Commit会被禁止推送到远程代码仓库，如企业版的Gitlab，Gitee，这些推送限制准则是公司的Git仓库管理员配置的。

可以在3个级别配置Git仓库的提交者信息: ① 系统 ② 全局 ③ 仓库。

优先级：仓库 > 全局 > 系统。

如果仓库内配置了，则使用仓库内配置，否则使用全局配置。如果全局也未配置，则使用系统配置。如果系统也未配置，那么commit不携带提交者的名字和邮箱。

- 系统配置：任何用户登录主机使用Git管理仓库，每个仓库都会继承系统配置

  ```shell
  git  config   --system  user.name     "euv"
  git  config   --system   user.email   "euv@gmail.com"
  ```

  系统配置存储路径是 `‪D:\Program Files\Git\etc\gitconfig` （git安装目录/etc/gitconfig）

- 全局配置：当前用户使用Git管理仓库，每个仓库都会继承全局配置

  ```shell
  git  config   --global   user.name     "euv"
  git  config   --global   user.email    "euv@gmail.com"
  ```

  全局配置存储路径是 `‪C:\Users\liliuwei\.gitconfig` （C:/users/用户/.gitconfig）

- 仓库内配置

  必须在当前仓库目录(.git同级目录)下打开Git Bash输入下面的命令

  ```shell
  git   config   --local   user.name    "euv"
  git   config   --local   user.email   "euv@gmail.com"
  ```

  仓库内配置存储路径: `.git/config`

上面3种配置方式生成的用户和邮箱信息在存储文件的显示如下：

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202306/1037641-20230626212253709-623663508.png" width="35%" height="35%" title=""/></div>

我们可以直接按照路径打开相应的配置文件查看当前配置或直接编辑文件(INI格式)进行配置。

---

**使用Visual Studio配置仓库作者名称和邮箱**

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202306/1037641-20230626212237354-1478666496.png" width="50%" height="50%" title=""/></div>

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202306/1037641-20230626212220353-271245437.png" width="50%" height="50%" title=""/></div>

---

**Summary**

我们的电脑上有存储在Gitee上的公司的项目，也有自己的存储在Github上的开源项目，公司项目仓库和开源项目仓库要使用不同的作者信息（公司使用公司邮箱，开源项目使用个人邮箱），这时候我们需要单独设置仓库的作者信息，单独为仓库设置的信息就不再继承git全局设置的作者信息。

如果电脑上的所有仓库都未指定作者信息，那么这些仓库的作者信息全部来自全局设置或系统设置。

# git status

> git status可列出工作区之于暂存区的文件差异，暂存区之于HEAD的文件差异。注意区分git status和git diff差异的含义：git status的差异是指文件状态的差异，如新增，删除，修改，待提交，待暂存；git diff的差异是指文件内容的差异，是指文件的具体内容发生了什么变化；

<font>工作区之于暂存区的差异</font>

| 序号 | 工作区 | 暂存区 | 差异     | 状态   |
| ---- | ------ | ------ | -------- | ------ |
| ①    | 有     | 有     | 内容不同 | M      |
| ②    | 有     | 无     |          | U      |
| ③    | 无     | 有     |          | D      |
| ④    | 有     | 有     | 内容相同 | 不显示 |

<img src="D:\OneDrive\markdownImages\image-20240717000909932.png" alt="image-20240717000909932" style="zoom:80%;" />

- `Changes not staged for commit`：这是提示信息，表示工作区的修改尚未进入暂存区而无法提交，且这些文件的更早修改也存在于暂存区。
- `(use "git add <file>..." to update what will be committed)`：这是一条建议性的消息，告诉你可以使用`git add`命令将更改暂存起来，以便提交。
- `(use "git restore <file>..." to discard changes in working directory)`：这是一条建议性的消息，告诉你可以使用`git restore`命令让暂存区的文件覆盖掉工作区相应的文件，也就是放弃对工作目录中文件的更改。
- `modified: 2.txt`：这表示`2.txt`文件已被修改。
- `deleted: 4.txt`：这表示`4.txt`文件已被删除。

- `Untracked files`：这是提示信息，表示这些文件刚在工作区被新建，未被纳入版本控制。
- `(use "git add <file>..." to include in what will be committed)`：这是一条建议性的消息，告诉你可以使用`git add`命令将这些文件加入版本控制。

<font>暂存区之于HEAD的差异</font>

| 序号 | 暂存区 | HEAD | 差异     | 状态   |
| ---- | ------ | ---- | -------- | ------ |
| ①    | 有     | 有   | 内容不同 | M      |
| ②    | 有     | 无   |          | D      |
| ③    | 无     | 有   |          | A      |
| ④    | 有     | 有   | 内容相同 | 不显示 |

<img src="D:\OneDrive\markdownImages\image-20240717004133387.png" alt="image-20240717004133387" style="zoom:80%;" />

- `Changes to be committed `表示暂存区的文件尚未正式提交到版本库。
- `git restore --staged <file>`这是一条建议性信息，表示可以使用HEAD中的文件覆盖掉暂存区相应的文件，相当于撤销git add.

> `Changes not staged for commit`和`Untracked files`描述的是工作区之于暂存区的差异，红色字体。
> `Changes to be committed` 描述的是暂存区之于HEAD的差异，绿色字体。

```shell
Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   1.txt
        both modified:   2.txt
```

Merge产生冲突时，运行git status，会出现节点`Unmerged paths`,它实际上指的是`工作区之于暂存区的差异！`

# git merge & git rebase

On branch main Your branch is up to date with 'origin/main'.

Your branch is ahead of 'origin/master' by 1 commit.  (use "git push" to publish your local commits)

git fetch 更新origin，会导致ahead。





# 忽略有道
## .gitignore的作用及如何避免规则无效

被.gitignore忽略的文件和目录，在执行`git add .`时，不会被添加到暂存区，自然不会流入版本库。换句话说，就是不会被追踪。

> 如果文件或目录已经在暂存区，则忽略规则无效。

<font>Example 1：a.txt位于工作区和暂存区，尚未提交到过版本库</font>

```tex
版本库: None

暂存区：a.txt

工作区: a.txt
```

.gitignore添加一行忽略规则`a.txt`，此忽略规则无效(使用`git check-ignore -v a.txt`可以验证并发现忽略规则`a.txt`未生效)。执行`git add .`，工作区的`a.txt`依旧会流入暂存区并覆盖暂存区原有的`a.txt`。

<img src="https://gitee.com/li6v/img/raw/master/202407152124282.png" alt="image-20240715212420186" style="zoom:67%;" />

> 解决方案是，使用`git rm -rf --cached a.txt`将`a.txt`从暂存区删除，这时忽略规则就生效了。

<img src="https://gitee.com/li6v/img/raw/master/202407152131633.png" alt="image-20240715213114558" style="zoom:67%;" />

<font>Example 2: a.txt位于版本库，暂存区，工作区</font>

```tex
版本库: a.txt

暂存区：a.txt

工作区: a.txt
```

<img src="https://gitee.com/li6v/img/raw/master/202407152135767.png" alt="image-20240715213551698" style="zoom:67%;" />

Example2和Example1区别不大，`重点分析理解下面流程中文件状态的流转过程`。

1. a.txt --> .gitignore，发现a.txt忽略规则并未生效。
2. git rm -rf --cached a.txt，a.txt从暂存区被删除，暂存区不再存在a.txt，但是版本库HEAD和工作区仍旧都存在a.txt。此时，a.txt忽略规则生效，执行git status,可看到显示版本库比暂存区多出a.txt(deleted: a.txt)，但是看不到显示工作区比暂存区多出a.txt，原因是忽略规则生效了！`忽略规则仅仅是只影响工作区和暂存区之间的差异以及工作区文件的可暂存性，并不会影响暂存区相较于HEAD,HEAD和HEAD^等等。`
3. 执行 git commit，此时版本库HEAD和暂存区都没有了a.txt(`HEAD^有a.txt`)
4. 执行git add，工作区的a.txt并不会进入暂存区。
5. *如果.gitignore是新建的，之前从未进入过暂存区和版本库，放到最后再提交.gitignore也是完全合理可行的。*

> 如果忽略规则命中的文件已经在暂存区，则忽略规则不会生效(相当于根本没有写到.gitignore里面)，执行git add 仍旧会正常提交。必须从暂存区删除规则命中的文件，规则才会生效。
>
> .gitignore只影响git add和git status这两个命令。
>
> git add: git add . 会将工作区的所有文件进入暂存区，.gitignore可以禁止工作区的某些文件或目录进入暂存区导致流入版本库，也就是不对某些文件和目录进行版本管理。
>
> git status：git status会列出暂存区之于HEAD之间的文件差异，以及工作区之于暂存区的差异。.gitignore只影响后者，让工作区的部分文件不参与它相较于暂存区的状态差异。


##  .gitignore Templates

https://github.com/github/gitignore/blob/main/VisualStudio.gitignore

## 规则定义

`本文讲解都假设.gitignore和.git都在文件夹repo下。`

- repo/
  - .git/
  - .gitignore




- 把.gitignore放置在.git的所在目录，即二者在同一级目录。
- 每条规则独占一行
- 空行无效，仅隔离以增强可读性
- \#开头的行表示注释，\ 可用于转义
- 路径分隔符统一使用 / ，不要用 \  ，即使是Windows系统
- 当/在末尾时，只匹配目录，否则，同名的目录和文件都将匹配。ax,会忽略repo/ax(ax是文件夹)和repo/ax(ax是文件)。ax/，会忽略repo/ax(ax是文件夹),但不会忽略repo/ax(ax是文件)。
- ** 用于匹配多级目录，可在开始，中间，结束。举例：`**/a`，可以匹配repo/a, repo/1/a, repo/1/2/a, repo/1/2/3/a;  `a/**/b`可以匹配a/b, a/1/b, a/1/2/b, a/1/2/3/b .
- *可以匹配任何字符(0次或多次)，？可以匹配任何字符(1次)【注意它们都不可以匹配 / 】
- \[ ] 通常用于匹配一个字符列表，如：a[mn]z可匹配amz和anz
- 原先被排除的文件，使用!模式可以被重新包含，但是如果该文件的父级目录被排除了，即使使用!也不会再被包含进来。

## 难点规则

难点1：`规则字符串的开头是否是/ `

当`/`在开头时，表示根目录。

`/log`，仅忽略根目录下的log，即repo/log；不会忽略repo/abc/log。

当开头不是`/`时，情况比较复杂，要分类。

**如果表示目录，则任意下级都被匹配。**

`logs/`

会忽略

repo/logs/

repo/plc/logs/

`pro/logs/`

会忽略

repo/pro/logs/

repo/plc/pro/logs/

**如果表示文档(文件夹和文件)，要分2种情况:
第一种是无前级目录，即规则字符串无/，示例，*.log, [1-9].json, configs, 这种情况时，任意下级都被匹配。
第二种是有前级目录，即规则字符串有/，示例，logs/config, logs/?.json,这种情况时，只能是根目录下；如果想任意下级都匹配，可改写成\*\*/logs/?.json**

*注：.gitignore的每行规则只能表示文件夹( / 结尾)，或文件夹和文件(非 / 结尾)，不能做到只表示文件*



---


难点2：解析步骤

.gitignore的规则可分成2类，一类是非!类，这种规则导致文件被排除，另一类是!类，这种规则把排除的文件重新捞回来。

匹配时，把!类行全部放到非!类后面。先从上到下解析每条非!规则，非!规则行作用的是原文件集合，把当前行匹配的文件全部扔到被排除集合中，再匹配下一行，匹配下一行时，下一行匹配到的文件同样会扔到排除集合中，所以，这相当于求所有行规则的并集，解析的行越多，被排除集合中的文件数量越多。直到所有的非!类规则解析完毕，开始解析!行，!行作用的集合是被排除集合，从被排除集合中拿出文件恢复到原集合。直到所有的行解析完毕，被排除集合残留的文件就是最终被忽略的文件。

## 规则案例



| Pattern                       | Example matches                                              | Explanation                                                  |
| ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `**/logs`                     | `logs/debug.log`<br> `logs/monday/foo.bar` <br/>`build/logs/debug.log` | You can prepend a pattern with a double asterisk to match directories anywhere in the repository. |
| `**/logs/debug.log`           | `logs/debug.log` <br/>`build/logs/debug.log` <br/>*but not<br/>*`logs/build/debug.log` | You can also use a double asterisk to match files based on their name and the name of their parent directory. |
| `*.log`                       | `debug.log` <br/>`foo.log` <br/>`.log` <br/>`logs/debug.log` | An asterisk is a wildcard that matches zero or more characters. |
| `/debug.log`                  | `debug.log` <br/>*but not* <br/>`logs/debug.log`             | Patterns defined after a negating pattern will re-ignore any previously negated files. |
| `debug.log`                   | `debug.log` <br/>`logs/debug.log`                            | Prepending a slash matches files only in the repository root. |
| `debug?.log`                  | `debug0.log` <br/>`debugg.log` <br/>*but not* <br/>`debug10.log` | A question mark matches exactly one character.               |
| `debug[0-9].log`              | `debug0.log` <br/>`debug1.log` <br/>*but not* <br/>`debug10.log` | Square brackets can also be used to match a single character from a specified range. |
| `debug[01].log`               | `debug0.log`<br/> `debug1.log` <br/>*but not* <br/>`debug2.log` <br/>`debug01.log` | Square brackets match a single character form the specified set. |
| `debug[!01].log`              | `debug2.log`<br/> *but not*<br/>`debug0.log`<br/> `debug1.log` <br/>`debug01.log` | An exclamation mark can be used to match any character except one from the specified set. |
| `debug[a-z].log`              | `debuga.log`<br/> `debugb.log` <br/>*but not<br/>* `debug1.log` | Ranges can be numeric or alphabetic.                         |
| `logs`                        | `logs` <br/>`logs/debug.log<br/>` <br/>`logs/latest/foo.bar` <br/>`build/logs` <br/>`build/logs/debug.log` | If you don't append a slash, the pattern will match both files and the contents of directories with that name. In the example matches on the left, both directories and files named *logs* are ignored |
| `logs/`                       | `logs/debug.log` <br/>`logs/latest/foo.bar` <br/>`build/logs/foo.bar` <br/>`build/logs/latest/debug.log` | Appending a slash indicates the pattern is a directory. The entire contents of any directory in the repository matching that name – including all of its files and subdirectories – will be ignored |
| `logs/` `!logs/important.log` | `logs/debug.log` <br/>`logs/important.log`                   | Wait a minute! Shouldn't `logs/important.log`be negated in the example on the left  Nope! Due to a performance-related quirk in Git, you can not negate a file that is ignored due to a pattern matching a directory |
| `logs/**/debug.log`           | `logs/debug.log` <br/>`logs/monday/debug.log` <br/>`logs/monday/pm/debug.log` | A double asterisk matches zero or more directories.          |
| `logs/*day/debug.log`         | `logs/monday/debug.log` <br/>`logs/tuesday/debug.log` <br/>*but not* <br/>`logs/latest/debug.log` | Wildcards can be used in directory names as well.            |
| `logs/debug.log`              | `logs/debug.log` <br/>*but not* <br/>`debug.log` <br/>`build/logs/debug.log` | Patterns specifying a file in a particular directory are relative to the repository root. (You can prepend a slash if you like, but it doesn't do anything special.) |





|                                                         |                                                              |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| 忽略所有内容                                            | `/*`                                                         |
| 忽略所有的目录                                          | `*/`                                                         |
| 忽略public目录下的所有文件，除了favicon.ico文件         | `public/*`<br>`!public/favicon.ico`<br>but not <br>`public/`<br/>`!public/favicon.ico`<br>but not <br>`public`<br/>`!public/favicon.ico` |
| 只保留public目录下的所有a{一个字符}z.{后缀名}的所有文件 | `public/*`<br/>`!public/a?z.*`                               |
|                                                         |                                                              |

## Debugging .gitignore files

**检查某个文件或目录是否已经被忽略**

```text
git check-ignore -v logs/debug.log
git check-ignore -v logs/*.log
git check-ignore -v logs/
```

check-ignore后的路径字符串是相对路径，相对于当前目录。如果没被忽略，则不会输出任何内容；如果被忽略，则输出路径字符串和导致其被忽略的规则，如下

```powershell
PS D:\repo> git check-ignore -v .\logs\debug.log
.gitignore:1:/logs/debug.log    ".\\logs\\debug.log"
```

*check-ignore后的文档名称不需要是工作区真实存在的文档，可以是假设的想测试的某个格式的合法路径字符串。*

**列出所有被忽略的文件**
git status -s --ignored



# git add



git add -u 只添加已经追踪的文件

git add . 添加所有文件

git restore要保证源有文件。

git restore --stated 后，暂存区仍旧存在文件，但是git rm 后，暂存区无文件。

”git rm --cached \<file>..." to unstage   从暂存区删除

"git restore --staged \<file>..." to unstage  恢复到暂存区

