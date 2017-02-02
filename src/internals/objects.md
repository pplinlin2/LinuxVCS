# Git objects
## Introduction
* Plumbing command: 低階指令
* Porcelain command: 高階指令

![Blueprint](https://github.com/pplinlin2/LinuxVCS/blob/master/src/internals/objects.png)

## Blob object
```console
# mkdir test
# cd test
# git init
Initialized empty Git repository in /tmp/test/.git/
# find .git/objects/
.git/objects/
.git/objects/pack
.git/objects/info
# find .git/objects/ -type f
#
```
使用plumbing command `hash-object`來根據file內容產生object，`-w`代表將object寫入database，
執行完後可以發現`.git/object/`底下已經儲存了資料，可以透過`cat-file`來查看object，`-p`代表顯示object內容
```console
# echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
# find .git/objects/ -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
# git cat-file -p d670
test content
```
因此只要透過`hash-object`的方式，就可以達到對內容版本的管理，下面展示做法
```console
# echo 'version 1' > test.txt
# git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30
# echo 'version 2' > test.txt
# git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
```
可以發現兩個object都已經儲存在database中
```console
# find .git/objects/ -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
```
先回復到第一版，再回復到第二版
```console
# git cat-file -p 83ba > test.txt
# cat test.txt
version 1
# git cat-file -p 1f7a > test.txt
# cat test.txt
version 2
```

## Tree object
使用plumbing command `update-index`將test.txt加入tree object之中
```console
# git update-index --add --cacheinfo 100644 83baae61804e65cc73a7201a7252750c76066a30 test.txt
# git write-tree
d8329fc1cc938780ffdd9f94e0d364e0ea74f579
```
創建完成後，檢查一下object
```console
# git cat-file -p d832
100644 blob 83baae61804e65cc73a7201a7252750c76066a30    test.txt
# git cat-file -t d832
tree
```
新增一個new.txt，並將test.txt切換回第二版
```console
# echo 'new file' > new.txt
# git cat-file -p 1f7a > test.txt

# git update-index --add new.txt
# git update-index test.txt

# git write-tree
0155eb4229851634a0f03eb265b69f5a2d56f341
# git cat-file -p 0155
100644 blob fa49b077972391ad58037050f2a75f74e3671e92    new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a    test.txt
```
也可以將tree object做為一個子目錄加入另一個tree object
```console
# git read-tree --prefix=bak d832
# git write-tree
3c4e9cd789d88d8d89c1073707c3585e41b0e614
# git cat-file -p 3c4e
040000 tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579    bak
100644 blob fa49b077972391ad58037050f2a75f74e3671e92    new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a    test.txt
```

## Commit object
使用plumbing command `commit-tree`來建立commit object，`-p`指定parent commit object
```console
# cat git.env
export GIT_AUTHOR_NAME="author"
export GIT_AUTHOR_EMAIL="author@gmail.com"
export GIT_AUTHOR_DATE="1970-01-01T00:00:00"
export GIT_COMMITTER_NAME="committer"
export GIT_COMMITTER_EMAIL="committer@gmail.com"
export GIT_COMMITTER_DATE="1970-01-01T00:00:00"

# (source git.env && echo 'first commit' | git commit-tree d832)
c2233e7ace988f0f5c72f35edc4890d567e69226
# (source git.env && echo 'second commit' | git commit-tree 0155 -p c223)
e1e1a722502dafa81f3aa20ca6956bba2f3ecc42
# (source git.env && echo 'third commit' | git commit-tree 3c4e -p e1e1)
374fdbdcfa0805ec77d12ea8dca6bb9b1c5a7b80

# git log --stat 374f --pretty=oneline
374fdbdcfa0805ec77d12ea8dca6bb9b1c5a7b80 third commit
 bak/test.txt | 1 +
 1 file changed, 1 insertion(+)
e1e1a722502dafa81f3aa20ca6956bba2f3ecc42 second commit
 new.txt  | 1 +
 test.txt | 2 +-
 2 files changed, 2 insertions(+), 1 deletion(-)
c2233e7ace988f0f5c72f35edc4890d567e69226 first commit
 test.txt | 1 +
 1 file changed, 1 insertion(+)
```
