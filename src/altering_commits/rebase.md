# Rebase
## Introduction
* rebase = re + base = 重新 + 基礎版本，也就是把目前branch的基礎commit重新替換成別的branch的HEAD

## Example
### Rebase
建立實驗環境
```console
mkdir test
cd test
git init
echo 'A' >> file1
git add file1
git commit -m 'A'
echo 'B' >> file1
git commit -am 'B'

git checkout -b topic
echo 'W' >> file2
git add file2
git commit -m 'W'
echo 'X' >> file2
git commit -am 'X'
echo 'Y' >> file2
git commit -am 'Y'
echo 'Z' >> file2
git commit -am 'Z'

git checkout master
echo 'C' >> file1
git commit -am 'C'
echo 'D' >> file1
git commit -am 'D'
```
查看一下目前git的狀態
```console
# git log --oneline --graph --decorate --branches
* 9ebbd8e (HEAD -> master) D
* c301958 C
| * 37c6a38 (topic) Z
| * 21568cc Y
| * fcba829 X
| * a9dad28 W
|/
* eb2847f B
* 1d99fb9 A
```
將branch topic使用`git rebase`接回branch master，從結果可以看到原本的branch topic的基礎commit從B換成了D
```console
# git checkout topic
Switched to branch 'topic'
# git rebase master
First, rewinding head to replay your work on top of it...
Applying: W
Applying: X
Applying: Y
Applying: Z
# git log --oneline --graph --decorate --branches
* 2ebfcea (HEAD -> topic) Z
* 2e2bf5c Y
* d7f76c4 X
* f53f32a W
* 9ebbd8e (master) D
* c301958 C
* eb2847f B
* 1d99fb9 A
```
### Rebase + Fast-forward
將rebase後的branch topic使用fast-forward的方法merge回branch master，可以看到結果branch master的HEAD直接移到branch topci的HEAD
```console
# git checkout master
Switched to branch 'master'
# git merge topic
Updating 9ebbd8e..2ebfcea
Fast-forward
 file2 | 4 ++++
 1 file changed, 4 insertions(+)
 create mode 100644 file2
 # git log --oneline --graph --decorate --branches
* 2ebfcea (HEAD -> master, topic) Z
* 2e2bf5c Y
* d7f76c4 X
* f53f32a W
* 9ebbd8e D
* c301958 C
* eb2847f B
* 1d99fb9 A
```
先復原回還沒有merge的狀態
```console
# git reset --hard HEAD@{1}
HEAD is now at 9ebbd8e D
```
### Rebase + no-ff
這次將rebase後的branch topic使用關閉fast-forward的方法merge回branch master，可以看到新增了一個commit將branch master和branch topic合併起來
```console
# git merge topic --no-ff -m "Merge branch 'topic'"
Merge made by the 'recursive' strategy.
 file2 | 4 ++++
 1 file changed, 4 insertions(+)
 create mode 100644 file2
# git log --oneline --graph --decorate --branches
*   121f0ec (HEAD -> master) Merge branch 'topic'
|\
| * 2ebfcea (topic) Z
| * 2e2bf5c Y
| * d7f76c4 X
| * f53f32a W
|/
* 9ebbd8e D
* c301958 C
* eb2847f B
* 1d99fb9 A
```
