# Reset
## Introduction
* 三種模式
  1. Soft reset: `git reset --soft`
  1. Mixed reset: `git reset --mixed`
  1. Hard reset: `git reset --hard`
* 在commit B的狀態下執行指令`git reset --type A`的效果，其中type為soft、mixed或者hard

|type        |soft|mixed|hard|
|------------|:--:|:---:|:--:|
|Working dir.|B   |B    |A   |
|Index       |B   |A    |A   |
|Obj. store  |A   |A    |A   |
* 可以使用`git dff`系列指令來驗證`git reset`執行後的效果

## Example
先把實驗環境建好
```
mkdir test
cd test
git init
echo 'A' > test.txt
git add test.txt
git commit -m "A"
echo "B" >> test.txt
git commit -am "B"
```
目前的git log狀態
```console
# git log --oneline --graph --decorate
* 906d4ba (HEAD -> master) B
* 3d3cdb7 A
```
### Soft reset
使用`git reset --soft`回到上一個commit，用`git diff`系列指令來驗證結果
```console
# git reset --soft HEAD^
# git diff --stat
# git diff --cached --stat
 test.txt | 1 +
 1 file changed, 1 insertion(+)
# git diff HEAD --stat
 test.txt | 1 +
 1 file changed, 1 insertion(+)
```
回到一開始的實驗狀態，用`git log`確認一下
```console
# git reset --hard HEAD@{1}
HEAD is now at 906d4ba B
# git log --oneline --graph --decorate
* 906d4ba (HEAD -> master) B
* 3d3cdb7 A
```
### Mixed reset
使用`git reset --mixed`回到上一個commit，並驗證結果
```console
# git reset --mixed HEAD^
Unstaged changes after reset:
M       test.txt
# git diff --stat
 test.txt | 1 +
 1 file changed, 1 insertion(+)
# git diff --cached --stat
# git diff HEAD --stat
 test.txt | 1 +
 1 file changed, 1 insertion(+)
```
再回到一開始的實驗狀態
```console
# git reset --hard HEAD@{1}
HEAD is now at 906d4ba B
```
### Hard reset
使用`git reset --hard`回到上一個commit，並驗證結果
```console
# git reset --hard HEAD^
HEAD is now at 3d3cdb7 A
# git diff --stat
# git diff --cached --stat
# git diff HEAD --stat
```
