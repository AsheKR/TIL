# Git / GitHub

## alias
```git
# git ci가 git commit을 한다.
git config --global alias.ci commit

# 명령어와 플래그가 같이 있을 경우 single quote를 사용
git config --global alias.graph 'log --graph'
```

## amend
```git
git commit --amend
```
마지막 커밋을 수정한다.
SHA-1값이 변경되기 때문에 PUSH한 사항에 대해 변경하는것은 피해야한다.

## reset
```git
# default, HEAD를 옮기고 Staging Area를 비운다. (Working dir 유지)
git reset --mixed

# HEAD를 옮긴다. (Staging Area와 Working dir 유지)
git reset --soft

# HEAD를 옮기고 Staging Area를 비우고, Work dir을 해당 스냅샷으로 되돌린다.
git reset --hard
```

## checkout
Branch를 옮길 때 사용
HEAD를 브랜치의 HEAD로 옮김

## revert
PUSH 사항을 바꾸고 싶을 때 해당 커밋을 삭제하는 커밋을 추가하는 것이 좋다.
```git
git revert {SHA}
```

## stash
하던 작업이 있는데 잠시 브랜치를 변경하거나 커밋할 일 이 생기면
`git stash`를 통해 잠시 모든 상황을 담아둘 수 있다.

```git
# 현재 상황 저장
git stash
# 이름으로 저장
git stash save {name}
# 스택 안 모든 스냅샷 리스트
git stash list
# 후입 선출 스냅샷 가져오기
git stash pop
# list로 확인한 것 중 필요한 것을 뽑음
git stash@{number}
# 마지막 Stash를 버림 @{number} 가능
git stash drop
# stash 스택 비우기
git stash clear
```

## branch
워크플로우 생성
```git
# 브랜치 생성
git branch {name}
# 브랜치 생성 후 checkout
git checkout -b {name}
# 브랜치 삭제
git branch -d {name}
# remote, local 브랜치 확인
git branch -a
# remote branch 삭제
git push --delete {remote name} {branch}
# 커밋을 포함한 branch 찾기
git branch --contains {SHA}
```

## REMOTE

```git
# url에 remotename을 지정하여 remote repo 저장
git remote add {remote name} {url}
# 모든 remote repo 보기
git remote -v
# remote repo의 URL 변경
git remote set-url {remote name} {url}
```

PUSH 명령으로 원하는 remote repo에 브랜치 상태를 올리거나, 태그를 올릴 수 있다.

## Tracking Branch(Upstream)
로컬 브랜치와 원격 브랜치를 연결해두면 push시 편하다.
```git
# 브랜치와 이에 해당하는 Tracking branch를 확인한다.
git branch -vv
# 현재 브랜치를 push하면서 remote 의 브랜치를 트래킹한다.
git push -u {remote name} {branch}
# 현재 브랜치가 remote의 브랜치를 트래킹한다.
git branch -u {remote name}/{branch}
```

## fetch
원격 브랜치의 정보를 가져온다.

## pull
원격 브랜치의 정보를 가져오지만 fetch 후 Merge까지 같이 해준다.
```git
#pull시 rebase를 하게할 수 있다.
git pull --rebase
```

## merge
Branch를 합칠 때 사용한다.

## rebase
커밋 히스토리를 깔끔하게 관리하기위해 사용됨
로컬에서만 사용하자. 이미 push한 branch끼리 rebase하는 것은 좋지 않다.

## show
특정 파일의 특정 커밋시 내용이나 특정 커밋의 변경사항을 본다.
```git
# 마지막 커밋의 상세내용 조회
git show
# 해당 커밋의 상세내용 조회
git show {SHA}
# remote repo branch의 파일 조회
git show {remote name}/{branch}:{file}
# 이전 커밋의 file 조회
git show HEAD~4:{file}
```

## Pull Request
자신이 포함되지 않은 GitHub에 자신의 코드를 반영하게 하는 방법
오픈소스를 clone 후, 자신의 branch를 따고, pull request를 통해 원래 브랜치에 자신의 브랜치를 Merge 해달라는 요청을 할 수 있다.

## Blame
각 파일의 마지막 커미터를 보여준다
```git
#파일의 모든 행의 마지막 수정자를 보여준다.
git blame {파일}
# 파일의 수정자의 이메일을 위주로 보여준다.
git blame -e {파일}
# 특정 라인부터 특정 라인까지 보여준다.
git blame -L {startline},{endline} {파일}
```

## tag
커밋에 릴리즈 로그를 붙이고 싶을 때
```git
#전체 태그를 확인
git tag

#glob 패턴 검색과 함께 확인
git tag -l 'v1.0*'

#Annotated tag를 메세지와 함께 남긴다.
git tag -a v1.0 -m "tag message"

# 특정 커밋에 태그를 남긴다.
git tag -a v1.0 -m "tag message" {hash}

# 태그 삭제
git tag -d v1.0
```

자동으로 태그를 push하지 않기 때문에, `git push {remote name} {tag name}`으로 올려준다.

## reflog

커밋을 잃어버렸을 경우 사용한다.
완전히 커밋을 유실되었다고 판단되는 경우 (.git/logs/내에서 삭제) git fsck --ful 명령어를 이용하여 Dangling commit을 보고 복구가능하다.

# Git 전략

## Git Flow
`feature > develop > release > hotfix > master`
메인은 master, develop 브랜치

## GitHub Flow
`release` 브랜치가 명확하지 않는 시스템에 사용에 맞게 되어있다.
`master` 브랜치는 항상 최신으로 배포 가능한 상태
`master`에서 브랜치를 따면 어떤일을 하는지 명확한 이름을 작성한다.
원격지 브랜치로 수시로 push한다.
머징 준비가 완료되었을 때 `pull request`
`master`로 머지되고 푸시되었을 때 즉시 배포되어야한다.
