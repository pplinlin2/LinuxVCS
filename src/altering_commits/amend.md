# Amend
## Introduction
* 重新提交一次commit到目前的HEAD
  1. 有檔案commit時忘記加入
  1. commit後發現有錯誤
  1. 修正commit message的錯字
* 簡單說就是把原本commit `A -> B -> C` 改成 `A -> B -> C'`

## Example
先建好實驗的環境
```
mkdir test
cd test
git init
echo 'A' > test.txt
git add test.txt
git commit -m "A"
echo 'B' >> test.txt
git commit -am "B"
echo 'C' >> test.txt
git commit -am "C"
```
目前git log的狀態
```console
# git log --oneline --graph --decorate
* a209a0f (HEAD -> master) C
* 4575cbd B
* c62a86e A
```
使用`git commit --amend`來修改最近的commit後，再查看一次git log，可以發現成功從`A -> B -> C` 改成 `A -> B -> C'`了
```console
# sed -i "s/C/C'/g" test.txt
# git commit --amend -m "change from C to C'"
[master 906e7af] change from C to C'
 Date: Mon Feb 6 18:14:04 2017 +0800
 1 file changed, 1 insertion(+)

# git log --oneline --graph --decorate
* 906e7af (HEAD -> master) change from C to C'
* 4575cbd B
* c62a86e A
```
