
# Intro

1. 매일 Commit ( **T** oday  **I** **L** earn, 혹은 코드 따라쓰기, 튜토리얼 )
2. IT 책 출간 코드리뷰
3. 스프린트
4. 정적 블로그 운영
***

# git 사용 이유

파일변경점을 파악하기 쉽다.
네트워크에 연결되어 있지 않아도 커밋이 가능하고
PUSH를 사용하여 원격 저장소에 올릴 수 있다.

# 실습

1. Create Repository

MIT License?
상업적으로 사용해도 상관 없는 것

2. 둘러보기

- OverView
	private contribute?
	비공개했던 커밋들도 보여준다.
- Contribute
	동업자들을 표시해준다.
- Wiki
	HTML 파일로 생성할 수 있다

3. Create new File

기본적으로 md 파일로 작성되고
Commit log 를 필수로 적어준다.
Commit log는 Verb + Noun

이곳에서 생성한 파일을 Commit new file 하는 경우
Add + Commit + Push
세 과정을 한번에 해준다.

~~프로필 사진은 등록해놓는게 좋다~~

4. LOCAL PC ~ 원격지간의 통신 (SSH)

Settings에서 SSH 키를 생성가능하다.
generating SSH Key

```powershell
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
passphrase는 SSH 암호에 포함될 비밀번호와 같다.
그 후 rsa 내부 파일을 GIT에 등록하기위해 xclip을 사용하여 등록한다.

```powershell
$ sudo apt-get install xclip
$ xclip -sel clip < ~/.ssh/id_rsa.pub
```

그 후 아무 TITLE을 정해준 후 등록하면 된다.

5. 로컬에 레포지토리 등록

Clone or Download에서 Use SSH를 사용하고 링크를 복사한다.

ZSH 기준 아래 내용을 사용하면
git clone --recursive 가 작동하게 된다.

```powershell
gcl "[LINK]"
```

해당 레포지토리 폴더에 들어가서 
git remote -v 를 사용하게되면
~~내용 추가 바람~~

ZSH에서는 다음과 같다.
```powershell
gr
```

만약 레포지토리에서 수정사항이 있다면
ZSH에서는 *노란 불*이 들어오고
수정사항이 없다면 *초록 불*이 들어온다.

스테이터스를 통해 레포지토리에서 변경사항을 확인할 수 있다.

git status

```powershell
gst
```

올리기 전 ADD를 통해 올릴 변경사항을 설정할 수 있다.

git add "[FILENAME]"

```powershell
ga "[FILENAME]"
```

COMMIT을 통해 원격 저장소에 올릴것을 설정할 수 있고

git commit -m "COMMENT"

```powershell
gcm "COMMENT"
```

```
기본적인 이메일 설정이 되어있지 않다면
fatal: unable to auto-detect email address
라는 오류가 발생한다.

git config --global user.email "[EMAIL]"
을 설정해주면 이메일이 바뀌게된다.
```

마지막으로 원격저장소에 PUSH 해주면 올라간다.

git push -u origin master
```powershell
ggp
```

만약 원격 저장소에서 무언가 변경되었을 경우

git pull origin master
```powershell
ggl
```

***
## FETCH, MERGE, PULL의 차이
```
git fetch
git merge
git pull(git fetch + merge)
git pull --rebase (git fetch + merge + add+ commit)
```

원래 저장소를 upstream이라고 부른다.
원래 저장소에서 내 저장소로 가져오는 것을 FORK 라고 하고 저장소로 가져오고 LOCAL로 가져와야한다.

수정 후 내 저장소에 PUSH 하고 원래 저장소에 pull request를 보내 원격 저장소에 허락을 맡아야한다.

## Branch?

Master Branch는 기본
~~?~~
협업시 발생하는 *문제점*들 때문에 사용하게 된다.

Branch 명은
이슈번호 혹은 JIRA 등 협업 툴에 있는 티켓번호를 포함한다.

## Git Stash

최신 소스를 가지고 오고싶은데 내 저장소의 내용이 커밋되지 않았다면 임시로 내 내용물들을 저장해둔 뒤 git statsh pop 으로 꺼낸다.

## 커밋 메세지 작성법

1. 제목과 본문을 빈 행으로 분리한다.
2. 제목 행을  50자로 제한
3. 제목 행 첫 글자는 대문자로
4. 제목 행 끝에 마침표를 넣지 않는다.
5. 제목 행에 명령문을 사용한다.
6. 본문은 72자로 제한
7. 무엇과 왜 를 설명한다.

## 커밋 메세지 수정
마지막 커밋을 수정
$ git commit -- amend

커밋 메세지 여러개 수정
~~git rebase -~~

rebase 종료하기
$ git rebase --continue

## git-flow

브랜치 전략
~~MASTER, DEVELOP, FEATYRE, RELEASE, HOTFIX~~
추후 설명 추가

## Pull Request 보내기

## 성공한 프로젝트 관리를 보고 배우기

## github를 이용한 정적 페이지

## 공동 번역 프로젝트
https://guides.github.com/

## 파일 최신 상태로 유지하기

외부에서 포크해서 가져왔기 때문에 REMOTE를 등록해야한다.
```
git remote -v
git remote add upstream "[LINK]"
```

최신상태의 파일을 가져오기 위해 로컬로 우선 가져오고, 내 레포지토리에 업데이트한다.
```
git pull --rebase upstream master
git push -u origin master
```

## PULL REQUEST

New Pull Request

Conflict 가 발생하면 파일 안에 들어가서 수정한 후 Merge 해주면 된다.
