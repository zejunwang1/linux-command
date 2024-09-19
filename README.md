## Linux 常用命令总结

- **apt**

- **git**

- **find**

- **grep**

- **sort**

- **ln -s**

- **lsb_release -a**

- **free**

- **scp**

- **firewall-cmd**

- **iptables**

- **tar**

- **zip**

- **gzip**

- **ps -aux**

### apt

`apt` 是 Ubuntu 中的包管理工具。

使用 `apt update` 更新本地软件包列表：

```shell
sudo apt update
```

使用 `apt upgrade` 升级系统中的软件包：

```shell
sudo apt upgrade
```

使用 `apt search` 搜索系统中的包，可以帮助找到与搜索关键词相关的软件包：

```shell
apt search mkl
```

使用 `apt install` 安装软件包：

```shell
sudo apt install intel-mkl
```

使用 `apt policy` 显示软件包的安装状态和版本信息：

```shell
apt policy intel-mkl
```

使用 `apt remove` 移除指定的软件包：

```shell
sudo apt remove intel-mkl
# 完全卸载包
sudo apt --purge remove intel-mkl
```

#### git

![image](image/git.png)

使用 `git` 克隆远程仓库到本地：

```shell
git clone https://github.com/zejunwang1/TurboTransformers
```

如果远程仓库中存在一些子模块，可以使用如下命令确保子模块的初始化：

```shell
git submodule update --init --recursive
```

使用 `git checkout` 命令创建和切换分支：

```shell
# 检查本地仓库相对于远程仓库修改的文件
git checkout
# 切换分支
git checkout dev
# 创建并切换到新分支
git checkout -b dev
```

删除分支：

```shell
# 删除本地分支
git branch -D dev
# 删除一个已经被合并到其他分支的本地分支
git branch -d dev
# 删除远程分支
git push --delete origin dev
```

需要注意的是，在删除本地分支时，需要确保当前分支不是要删除的分支。

---

使用 `git` 提交本地代码库修改到远程：

```shell
git add .
git commit -m "Update"
git push
```

使用 `git reset` 命令回退本地仓库版本：

```shell
git reset [--soft | --mixed | --hard] [HEAD]
```

```shell
# 回退本地仓库和暂存区到上一个版本
git reset HEAD^
# 回退test.txt文件到上一个版本
git reset HEAD^ test.txt
# 向前回退2个版本
git reset HEAD~2
# 回退到某个版本52464e6
git reset 52464e6
```

`git reset --mixed`：默认方式，将待撤回的代码存放在工作区，会保留本地未提交的内容。

`git reset --soft`：回退到某个版本，将待撤回的代码存放在暂存区，会保留本地未提交的内容。

`git reset --hard`：彻底回退到某个版本，丢弃待撤回的代码，本地未提交的修改会被全部擦除。

对于已经 push 的 commit，也可以使用 reset 命令，不过再次 push 时，由于远程分支和本地分支有差异，需要强制推送 `git push -f` 来覆盖被 reset 的 commit。

---

使用 `git ls-files` 命令检查文件是否在暂存区：

```shell
git ls-files --stage | grep test.txt
```

使用 `git rm` 命令从暂存区删除文件：

```shell
# 从工作区和暂存区删除文件，本地文件被删除
git rm -r test.txt
# 从暂存区删除文件，本地文件还在，只是不希望该文件被版本控制
git rm -r --cached test.txt
```

---

当我们提交代码前想先 pull 一下时发现：

```shell
$ git pull
Updating 468bc4c..a24f8a5
error: Your local changes to the following files would be overwritten by merge:
    test.txt
Please commit your changes or stash them before you merge.
Aborting
```

说明我们本地修改的代码与别人修改提交的代码有冲突。如何解决？

1. 使用 `git stash` 命令先存储本地代码：
   
   ```shell
   git stash
   ```

2. 然后再 pull 远程代码：
   
   ```shell
   git pull
   ```
   
   ```shell
   Updating 468bc4c..a24f8a5
   Fast-forward
    test.txt | 1 +
    1 file changed, 1 insertion(+)
    create mode 100644 test.txt
   ```
   
   发现这时已经不报错了，能正常 pull 下来。

3. 最后把本地存储的代码释放出来：
   
   ```shell
   git stash pop
   ```
   
   ```shell
   CONFLICT (add/add): Merge conflict in test.txt
   Auto-merging test.txt
   ```
   
   发现在 test.txt 文件里会有冲突，我们手动解决下冲突，就可以正常提交了。

---

假设现在有两个分支 main 和 dev，在 dev 分支修改了一个文件并提交成功，而 main 分支也改了同一个文件并提交成功，当我们将 dev 分支合并至 main 时，有可能存在冲突：

```shell
git checkout main
git merge dev
```

```shell
Auto-merging test.txt
CONFLICT (content): Merge conflict in test.txt
Automatic merge failed; fix conflicts and then commit the result.
```

只需找到当前冲突文件，解决冲突即可重新提交。

```context
<<<<<<< HEAD
This is a test file for git learning.
=======
Merge branch test...
>>>>>>> dev
```

---

我们有时会遇到这样的情况，正在 dev 分支开发新功能，做到一半时团队成员反馈一个 bug，需要马上解决，但新功能又暂时不想提交，这时就可以使用 `git stash` 命令将 dev 分支的工作区和暂存区保存起来，然后切换到另一个分支去修改bug，修改完提交后再切回 dev 分支，使用 `git stash pop` 来恢复之前的进度继续开发新功能。

下面是使用 `git stash` 时要遵循的顺序：

- 将修改保存到分支 A 工作区

- 运行 `git stash`

- 签出分支 B

- 修正 B 分支的 bug

- 提交并推送 (可选) 至远程

- 切回分支 A

- 运行 `git stash pop` 来恢复工作区暂存的改动

下面是 `git stash` 命令的常见用法

- 存储工作区和暂存区
  
  ```shell
  git stash
  # 可以为本次存储命名
  git stash save "test"
  ```

- 列出缓存区中保存的记录
  
  ```shell
  git stash list
  ```

- 恢复某一次的缓存，如果不指定 stash_id，则默认恢复最新的缓存
  
  ```shell
  git stash pop [stash_id]
  ```

- 将堆栈中的内容应用到当前工作区，且不会删除缓存栈中的记录，可将堆栈中某次缓存的内容多次应用到工作目录中，适用于多个分支的情况
  
  ```shell
  git stash apply [stash_id]
  ```

- 清除缓存栈中的指定记录，如果不指定 stash_id，则默认清除最新的缓存记录
  
  ```shell
  git stash drop [stash_id]
  # 清除所有的缓存记录
  git stash clear
  ```

- 查看堆栈中的缓存与当前目录的差异
  
  ```shell
  # 最新一次的缓存
  git stash show [-p]
  # 指定的某次缓存
  git stash show [stash_id] [-p]
  ```

- 如果一个分支和缓存中的变更有分歧，当我们试图重新应用缓存时，会造成冲突。一个简单的解决方法是使用 `git stash branch` 命令，它将根据缓存记录创建一个新分支，并将缓存中的修改弹出
  
  ```shell
  git stash branch dev_2 [stash_id]
  ```

#### find

```shell
find pathname -option [-print] [-exec|-ok command] {} \;
```

- `pathname`: 要查找的目录路径。例如用 **"."** 来表示当前目录，用 **"/"** 来表示系统根目录。

- `-print`: 将匹配的文件输出到标准输出。

- `-exec`: 对匹配的文件执行该参数所给出的 shell 命令。相应命令的形式为 **command {}**

- `-ok`: 和 -exec 的作用相同，不过会以一种更为安全的模式来执行该参数所给出的 shell 命令，在执行每一个命令 **command {}** 之前，都会给出提示，让用户来确定是否执行。

- `-name filename`: 查找名为 filename 的文件。

- `-iname filename`: 查找名为 filename 的文件，且忽略大小写。

- `-type [d/f]`: 按文件类型查找，d 表示目录，f 表示普通文件。 

使用 `find` 查找当前路径中匹配搜索关键词的文件/目录：

```shell
find . -name *cpp
```

使用 `find` 查找当前路径中匹配搜索关键词的目录：

```shell
find . -type d -name *cpp
```

查找匹配关键词的目录并列出其下的所有文件：

```shell
find . -type d -name *cpp -print -exec ls {} \;
```

查找一个名为 test.cpp 的文件并将其删除：

```shell
find . -name test.cpp -exec rm -rf {} \;
```

查找空文件或空目录：

```shell
find . -empty
```

查找更新时间在 1 天以内的所有文件/目录：

```shell
find . -mtime -1
# 查找更新时间在1天以上的所有文件
find . -type f -mtime +1
```

查找更新时间在 1 分钟以内的所有文件/目录：

```shell
find . -mmin -1
# 查找更新时间在1分钟以上的所有文件
find . -type f -mmin +1
```

查找当前路径下小于 2M 的文件/目录：

```shell
find . -size -2M
```

查找大于 50M 且小于 100M 的文件：

```shell
find . -size +50M -size -100M
```

查找大于 10K 且小于 10M 的文件：

```shell
find . -size +10k -size -10M
```

查找当前路径下超过 10M 的所有 .mp3 文件并删除它们：

```shell
find . -type f -name "*.mp3" -size +10M -exec rm {} \;
```

当需要进行基于 OR 运算的查找时，可以加上 -o 开关。查找当前路径下以 .h 或 .cpp 结尾的所有文件：

```shell
find . -name "*.h" -o -name "*.cpp"
```

#### grep

```shell
grep [options] [pattern] file
```

常用参数总结：

- `-i`: 不区分大小写

- `-n`: 显示匹配行与行号

- `-c`: 仅返回匹配到的行数

- `-v`: 反向选择，显示未匹配成功的行

- `-r`: 在当前目录及其子目录下进行匹配

- `-l`: 列出包含查找词的文件名

- `-L`: 列出不包含查找词的文件名

- `-E`: 使用 `egrep` 命令，扩展正则表达式使用

- `-F`: 不使用正则表达式

使用 `grep` 在文件中搜索字符串：

```shell
grep root /etc/passwd
# 不区分大小写
grep -i ROOT /etc/passwd
# 匹配多个查找词
grep -e root -e bash /etc/passwd
# 显示匹配行号
grep -n root /etc/passwd
# 仅显示匹配到的行数
grep -c root /etc/passwd
# 将匹配行的前两行和后三行显示出来
grep -A3 -B2 root /etc/passwd
# 显示不包含查找词的行
grep -v root /etc/passwd
```

使用 `grep` 在文件夹中搜索字符串：

```shell
grep -r malloc .
# 仅显示匹配到的文件名
grep -r -l malloc .
```

使用 `grep` 进行正则表达式的查找：

```shell
# 匹配字母数字开头的行并显示行号
grep -n '^[0-9a-zA-Z]' test.txt
# 匹配非英文字母开头的行
grep '^[^a-zA-Z]' test.txt
```

正则表达式主要依赖于元字符，以下是一些元字符的介绍：

| 元字符  | 描述                                              |
|:----:|:-----------------------------------------------:|
| .    | 匹配任意单个字符除了换行符                                   |
| ^    | 从起始位置进行匹配                                       |
| $    | 匹配最末端                                           |
| *    | 匹配前一个字符出现0次或1次以上                                |
| [ ]  | 匹配方括号内的任意字符                                     |
| [^ ] | 匹配除了方括号里的任意字符                                   |
| \    | 转义字符，用于匹配一些保留的字符，`[ ] ( ) { } . * + ? ^ $ \ \|` |

使用 `grep -E` 或 `egrep` 可以基于扩展正则表达式进行查找

| 元字符   | 描述                                   |
|:-----:|:------------------------------------:|
| ?     | 匹配前一个字符0次或1次                         |
| +     | 匹配前一个字符1次或多次                         |
| (xyz) | 字符集，匹配与 xyz 完全相等的字符串                 |
| |     | 或运算符，匹配符号前或后的字符                      |
| {n,m} | 匹配 num 个大括号之前的字符或字符集 (n <= num <= m) |

```shell
# 匹配含有123或boy的行
grep -E '123|boy' test.txt
# 匹配至少出现一个数字的行
grep -E '[0-9]+' test.txt
# 匹配至少出现2个o至多出现3个o的行
grep -E 'o{2,3}' test.txt
```

#### sort

`sort` 命令的基本语法如下：

```shell
sort [选项] [文件]
```

常用选项包括：

- `-r`：逆序排序（降序）

- `-n`：按数值进行排序

- `-k 列号`：按指定的列进行排序，默认列分隔符为制表符或空格

- `-t 分隔符`：指定列的分隔符

- `-u`：去除重复行，仅保留第一次出现的行并排序输出

- `-f`：忽略大小写进行排序

- `-b`：行略行首的空白字符进行排序

假设有一个文本文件 file.txt，内容如下：

```context
Tom 25 Male
Jerry 22 Female
Alice 27 Female
Lucy 9 Female
```

使用 `sort` 按字典顺序对每行进行排序：

```shell
sort file.txt
```

```context
Alice 27 Female
Jerry 22 Female
Lucy 9 Female
Tom 25 Malele
```

使用 `sort` 按第二列数值大小进行排序：

```shell
sort -k 2 -n file.txt
```

```context
Lucy 9 Female
Jerry 22 Female
Tom 25 Male
Alice 27 Female
```

使用 `sort` 先按第三列降序排序，再按第二列数值大小升序排序：

```shell
sort -k 3r -k 2n file.txt
```

```context
Tom 25 Male
Lucy 9 Female
Jerry 22 Female
Alice 27 Female
```

使用 `sort` 对文件进行去重并按第二列数值大小进行排序：

```shell
sort -u -k 2n file.txt
```

```context
Lucy 9 Female
Jerry 22 Female
Tom 25 Male
Alice 27 Female
```

使用 `sort` 按照文件大小进行排序：

```shell
du -sh * | sort -h
```

#### ln -s

使用 `ln -s` 命令为某一个文件/文件夹在另外一个位置建立一个同步的链接：

```shell
ln -s [源文件] [目标文件]
```

当多个目录用到相同的文件时，只需在一个固定的目录放上该文件，在其它目录下使用 `ln -s` 命令链接就可以，以避免重复占用磁盘空间。

#### lsb_release -a

使用 `lsb_release -a` 查看 Ubuntu 系统的详细信息：

```shell
lsb_release -a
```

```context
Distributor ID:    Ubuntu
Description:    Ubuntu 22.04 LTS
Release:    22.04
Codename:    jammy
```

使用 `cat /etc/*release*` 查看 Centos 系统的详细信息：

```shell
cat /etc/*release*
```

#### free

使用 `free` 命令查看服务器内存信息：

```shell
free -h
```

#### scp

使用 `scp` 将远程文件/文件夹复制到本地：

```shell
scp [-r] [远程用户名]@[远程IP地址]:[远程文件/目录] [本地目录]
```

```shell
scp ubuntu@10.41.157.40:/home/wangzejun/code/test.cpp /data/wangzejun/code
```

#### firewall-cmd

在 CentOS 系统上使用 `firewall-cmd` 开放 1369 端口：

```shell
firewall-cmd --permanent --add-port=1369/tcp
```

重新加载防火墙策略：

```shell
firewall-cmd --reload
```

查看 1369 端口是否被开启：

```shell
firewall-cmd --permanent --query-port=1369/tcp
```

#### iptables

在 Ubuntu 系统上使用 `iptables` 开放 1369 端口：

```shell
sudo iptables -A INPUT -p tcp --dport 1369 -j ACCEPT
```

查看端口开启情况：

```shell
sudo iptables -L
```

#### tar

使用 `tar` 解压 tar.gz 和 tar.bz2：

```shell
tar -xvf test.tar
tar -zxvf test.tar.gz
tar -jxvf test.tar.bz2
```

使用 `tar` 压缩文件夹为 tar.gz 和 tar.bz2：

```shell
tar -cvf test.tar test/
tar -zcvf test.tar.gz test/
tar -jcvf test.tar.bz2 test/
```

`-c` 表示创建新的归档文件， `-z` 表示对归档文件使用 `gzip` 压缩，`-j` 表示对归档文件使用 `bzip2` 压缩，`-v` 表示显示命令执行过程，`-f` 指定归档文件的名称，`-x` 表示释放归档文件中的文件及目录。

#### zip

使用 `zip` 压缩文件或文件夹：

```shell
# 压缩文件
zip test.zip test.txt
# 递归压缩文件夹，-q表示不显示压缩过程
zip -r -q test.zip test/
```

使用 `unzip` 解压：

```shell
unzip -q test.zip
```

#### gzip

使用 `gzip` 压缩文件为 .gz 形式：

```shell
# 删除原始文件
gzip test.txt
# 保留原始文件
gzip -k test.txt
```

使用 `gunzip` 或 `gzip -d` 解压：

```shell
# 删除.gz文件
gunzip test.txt.gz
gzip -d test.txt.gz
# 保留.gz文件
gunzip -k test.txt.gz
```

#### ps -aux

在 Linux 系统中，`ps -aux` 是用于查看系统进程信息的命令。

- `-a`: 显示所有用户的进程，而不仅仅是当前用户的

- `-u`: 显示详细的用户 / 拥有者 (user) 信息

- `-x`: 显示没有控制终端的进程

`ps -aux` 会列出当前系统中所有用户的所有进程，并显示详细的用户信息，包括没有控制终端的进程。示例输出如下：

```context
USER       PID  %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  16336  1108 ?        Ss   Dec01   0:02 /usr/lib/systemd/systemd
root         2  0.0  0.0      0     0 ?        S    Dec01   0:00 [kthreadd]
...
john       123  0.1  0.5 123456 7890 pts/0    R+   Dec01   1:23 /usr/bin/example-command
```

`USER` 表示进程的用户，`PID` 是进程的 `ID`，`%CPU` 和 `%MEM` 分别表示 CPU 和内存的占用百分比，`COMMAND` 是启动进程的命令。

查看 527706 进程的详细信息：

```shell
ps -aux | grep 527706
```

使用 `netstat -anp` 命令可以查看某个端口号对应的进程 ID：

```shell
# 查看9000端口对应的进程ID
netstat -anp | grep 9000
```
