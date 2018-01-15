# 커밋 합치기

깃으로 작업을 하다 보면 여러개의 커밋을 합쳐야 할 때가 있다.

그 경우 `git rebase -i HEAD~n`을 통해 여러 커밋을 합칠 수 있다.

> HEAD~n에는 합칠 커밋의 범위를 입력하면 된다.



<br>



```shell
pick 924a281 commit 1
pick 647aa3d commit 2

# Rebase 0dd24db..647aa3d onto 0dd24db (2 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

`git rebase -i HEAD~n`을 입력하면 해당 범위만큼의 커밋 정보가 나타내게 되는데, 다른 곳으로 병합할 커밋을 pick에서 squash로 변경하고 저장한다.



```she
pick 924a281 commit 1
squash 647aa3d commit 2
```



<br>



```shell
# This is a combination of 2 commits.
# This is the 1st commit message:

commit 1

# This is the commit message #2:

commit 2

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Mon Jan 15 21:32:29 2018 +0900
#
# interactive rebase in progress; onto 0dd24db
# Last commands done (2 commands done):
#    pick 924a281 [Effective JAVA] 규칙 3 - singleton pattern
#    squash 647aa3d [Effective JAVA] 규칙 3 - singleton pattern
# No commands remaining.
# You are currently rebasing branch 'master' on '0dd24db'.
#
# Changes to be committed:
#       new file:   Effective-JAVA/rule_03_singleton_pattern.md
#       deleted:    rule_02_builder_pattern.html
#
# Untracked files:
#       Git/
#
```

저장 할 경우 커밋 메시지를 수정 할 수 있다. squash 할 모든 커밋 메시지가 나타나며, 필요 없는 경우 주석 처리하여 저장한다.



```Shell
# This is a combination of 2 commits.
# This is the 1st commit message:

commit 1

# This is the commit message #2:

# commit 2
```



<br>

`git log` 를 통해 커밋 로그를 확인해보면 커밋이 합쳐졌음을 확인 할 수 있다.