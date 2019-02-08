# GIT의 내부



## Plumbing, Porcelain 명령

명령어 여러개를 Unix 스타일로 호출 가능한 저수준 명령어를 `Plumbing`

사용자에게 친숙한 사용자용 명령어는 `Porcelain`이라고 부른다.

`git init`은 `.git` 디렉터리를 생성한다.

```bash
HEAD
branches/  # Git 예전 버전에서만 사용됨
config  # 해당 프로젝트에만 적용되는 설정 옵션
description  # GitWeb에서 사용된다.
hooks/  # 클라이언트 훅이나 서버 훗을 넣는 곳
index
info/  # .gitignore처럼 무시할 파일 패턴을 적어두는 곳, 이는 Git으로 관리되지 않음
objects/
refs/
```

중요한것은 위 `HEAD`, `index`, `objects/`, `refs/`이다.

- `objects` 디렉터리는 모든 컨텐츠를 저장하는 데이터베이스
- `refs` 디렉터리는 커밋 개체의 포인터를 저장하는 데이터베이스
- `HEAD`파일은 현재 Checkout한 브랜치를 가리킴
- `index`파일은 Staging Area 정보를 저장한다.



## Git 개체

Git은 `Content-Addressable` , 즉 `Key-Value`파일시스템이다.

어떤 형식의 데이터라도 집어넣을 수 있고, 해당 Key로 언제든지 데이터를 가져올 수 있다.

Plumbing 명령어 `hash-object`에 데이터를 주면 `.git` 디렉토리에 저장하고 그 Key를 알려준다.



### objects 디렉터리

`.git/objects` 내부에는 `info`, `pack`의 빈 디렉터리를 생성한다.



#### Blob 개체

`hash-object` 명령으로 Git 데이터베이스에 텍스트파일을 저장하면 `.git/objects` 내 새 파일이 생긴다.

`hash-object`명령은 헤더 정보와 데이터 모두에 대한 SHA-1 해시값을 리턴한다. 이 40자의 해시값의 첫 두글자를 디렉터리 이름에 사용하고, 나머지 38글자를 파일 이름에 사용한다.

`has-object`로 생성된 파일을 `Blob` 개체라고 부른다.



#### Tree 개체

Tree는 디렉터리, Blob는 Inode나 일반 파일에 대응한다. 

Tree 개체는 한개 이상의 Blob, Tree 개체를 가지며 하위 개체들은 다음과 같은 속성을 가지고있다.

- SHA-1 포인터
- 파일 모드
- 개체 타입
- 파일 이름



#### 개체 저장소

Git은 개체 타입, 공백문자 하나, 내용의 크기, 마지막에 널문자를 추가하여 헤더를 만든다.

```python
>>> content = "what is up, doc?"
>>> header= f"blob {len(content)}\0"
"blob 16\000"
```

Git은 헤더와 원래 내용을 합쳐 `SHA-1` 체크섬을 계산한다.

```python
>>> store = header + content
"blob 16\000what is up, doc?"
>>> import hashlib
>>> sha1 = hashlib.sha1(store).hexdigest()
"bd9dbf5aae1a3862dd1526723246b20206e5fc37"
```

또 zlib으로 내용을 압축한다.

```python
>>> import zlib
>>> zlib_content = zlib.compress(str.encode(sha1))
```

마지막으로 zlib으로 압축한 내용을 개체로 저장한다. SHA-1 값중 맨 앞의 두자는 하위 디렉터리의 이름으로 사용하여 나머지 38자는 디렉터리 안에 있는 파일 이름으로 사용한다.

```python
>>> path = '.git/objects/' + sha1[:2] + '/' + sha1[2:]
>>> import pathlib, os
>>> pathlib.Path(os.path.dirname(path)).mkdir(parents=True)
>>> with open(path, 'w') as f:
    	f.write(zlib_content)
```

Git 개체는 모두 이 방식으로 저장하며 단지 종류만 다르다.

헤더가 `blob`가 아니라 `commit`이나 `tree`로 시작하게 되는 것 뿐이다.



## Git 레퍼런스

`git log 1a410e`라고 실행하면 전체 히스토리를 볼 수 있지만, `1a410e`를 기억해야한다.  SHA-1값을 사용하기보다 쉬운 이름으로 된 포인터를 사용하기위해 외우기 쉬운 이름으로 된 파일에 SHA-1값을 저장한다. GIT은 이러한 것을 `레퍼런스` 또는 `refs`라고한다. 

하위에 `heads`, `tags`의 디렉터리를 가진다.

레퍼런스를 사용하여 커밋의 위치를 저장할 수 있다.

```bash
$ echo "1a410efbd13591db07496601ebc7a059dd55cfe9" > .git/refs/heads/master
```

Git에는 좀더 안전하게 레퍼런스를 바꿀 수 있는 `update-ref` 명령이 있다.

```bash
$ git update-ref refs/heads/master 1a410efbd13591db07496601ebc7a059dd55cfe9
```

Git의 브랜치 역할이 바로 이것이다. 브랜치는 어떤 작업들 중 마지막 작업을 가리키는 포인터 또는 레퍼런스이다.

브랜치는 직접 가리키는 커밋과 그 커밋으로 따라갈 수 있는 모든 커밋을 포함한다.

`git branch (branchname)`을 사용하면 Git은 내부적으로 `update-ref` 명령을 실행한다. 입력받은 브랜치 이름과 현 브랜치 마지막 커밋의 SHA-1 값을 가져다가 `update-ref`를 실행한다.



### HEAD

HEAD는 현 브랜치를 가리키는 간접 레퍼런스이다. 간접 레퍼런스이기때문에 다른 레퍼런스와는 다르게 SHA-1 값을 가지고 있지 않는다.

```bash
$ cat .git/HEAD
ref: refs/heads/master
```

`git checkout test`를 실행하게되면 HEAD는 아래과 같이 바꾼다.

```bash
$ cat .git/HEAD
ref: refs/heads/test
```

`git commit`을 하게되면 새 커밋 개체가 만들어지는데, 지금 HEAD가 가리키고 있던 커밋의 SHA-1값이 커밋 개체의 부모로 사용된다.

이 파일도 손으로 직접 편집할 수 있지만 `symbolic-ref` 명령어로 좀더 안전하게 사용할 수 있다.

```bash
$ git symbolic-ref HEAD refs/heads/test
$ cat .git/HEAD
ref: fers/heads/test
```



### 태그

커밋 개체외 매우 비슷하다.  (브랜치랑 더 비슷한게 아닐까..?)

언제 태그를 달았는지, 태그 메세지, 어떤 커밋을 가리키는지에 대한 정보를 포함한다.

태그 개체는 Tree가 아니라 커밋 개체를 가리키는것이 그 둘의 차이이다. 브랜치처럼 커밋 개체를 가리키지만 옮길수는 없다.



태그는 `Annotated` 태그와 `LightWeight` 태그 두 종류로 나뉜다.

- LightWeight: 브랜치와 비슷하지만 브랜치처럼 옮길수는 없다.

```bash
$ git update-ref refs/tags/v1.0 cac0cab538b970a37ea1e769cbbde608743bc96d
```



- Annotated: 커밋을 직접 가리키지 않고 태그 개체를 가리킨다.

```bash
$ git tag -a v1.1 1a410efbd13591db07496601ebc7a059dd55cfe9 -m 'test tag'
```

태그 개체의 SHA-1 값의 내용의 내용을 조회해본다.

```bash
$ git cat-file -p 9585191f37f7b0fb9444f35a9bf50de191beadc2
object 1a410efbd13591db07496601ebc7a059dd55cfe9
type commit
tag v1.1
tagger Scott Chacon <schacon@gmail.com> Sat May 23 16:48:58 2009 -0700

test tag
```

`object` 부분에 실제로 태그가 가리키는 커밋이 있다. 커밋 개체뿐만아니라 모든 Git 개체에 태그를 달 수 있다.



## Packfile

현재 Git 데이터베이스에는 다음과 같은 파일들이 존재한다.

```bash
$ find .git/objects -type f
.git/objects/01/55eb4229851634a0f03eb265b69f5a2d56f341 # tree 2
.git/objects/1a/410efbd13591db07496601ebc7a059dd55cfe9 # commit 3
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a # test.txt v2
.git/objects/3c/4e9cd789d88d8d89c1073707c3585e41b0e614 # tree 3
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30 # test.txt v1
.git/objects/95/85191f37f7b0fb9444f35a9bf50de191beadc2 # tag
.git/objects/ca/c0cab538b970a37ea1e769cbbde608743bc96d # commit 2
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4 # 'test content'
.git/objects/d8/329fc1cc938780ffdd9f94e0d364e0ea74f579 # tree 1
.git/objects/fa/49b077972391ad58037050f2a75f74e3671e92 # new.txt
.git/objects/fd/f4fc3344e67ab068f836878b6c4951e3b15f3d # commit 1
```



Git은 zlib으로 파일 내용을 압축하기 때문에 저장공간이 많이 필요하지 않다.

크기가 큰 파일을 사용할 때 이 기능의 효과는 더욱 높아진다.

12K 정도의 파일을 받아 실험한다.

```bash
$ curl -L https://raw.github.com/mojombo/grit/master/lib/grit/repo.rb > repo.rb
$ git add repo.rb
$ git commit -m 'added repo.rb'

$ git cat-file -p master^{tree}
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob 9bc1dc421dcd51b4ac296e3e5b6e2a99cf44391e      repo.rb
100644 blob e3f094f522629ae358806b17daf78246c27c007b      test.txt

$ du -b .git/objects/9b/c1dc421dcd51b4ac296e3e5b6e2a99cf44391e
4102    .git/objects/9b/c1dc421dcd51b4ac296e3e5b6e2a99cf44391e

$ echo '# testing' >> repo.rb
$ git commit -am 'modified repo a bit'
[master ab1afef] modified repo a bit
 1 files changed, 1 insertions(+), 0 deletions(-)
```

기존 파일의 blob가 있고, 기존 파일에 몇줄의 코드를 추가한 파일의 blob가 존재한다고 했을 때 두가지는 완전히 새로운 Blob 개체이다. 이 두파일의 차이점만을 저장하는 기능을 지원한다.

```bash
$ git cat-file -p master^{tree}
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob 05408d195263d853f09dca71d55116663690c27c      repo.rb
100644 blob e3f094f522629ae358806b17daf78246c27c007b      test.txt

$ du -b .git/objects/05/408d195263d853f09dca71d55116663690c27c
4109    .git/objects/05/408d195263d853f09dca71d55116663690c27c
```



Git이 처음 개체를 저장하는 형식은 Loose 개체 포맷이라고 부른다. 이는 나중에 파일 하나로 압축(Pack)할 수 있다. 그래서 공간을 절약하고 효율을 높일 수 있다. 

Git이 이렇게 압축하는 때는 

- Loose 개체가 너무 많거나, 
- `git gc` 명령을 실행했을 때, 
- 그리고 리모트 서버로 Push 할 때 압축한다.

`git gc`를 실행하고 디렉터리를 열어보면 대부분의 개체가 사라지고 한쌍의 파일이 생긴다.

```bash
$ git gc
Counting objects: 17, done.
Delta compression using 2 threads.
Compressing objects: 100% (13/13), done.
Writing objects: 100% (17/17), done.
Total 17 (delta 1), reused 10 (delta 0)

$ find .git/objects -type f
.git/objects/71/08f7ecb345ee9d0084193f147cdad4d2998293
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
.git/objects/info/packs
.git/objects/pack/pack-7a16e4488ae40c7d2bc56ea2bd43e25212a66c45.idx
.git/objects/pack/pack-7a16e4488ae40c7d2bc56ea2bd43e25212a66c45.pack
```

압축되지 않은 Blob 개체는 어떤 커밋도 가리키지 않는 개체이다. 어떠한 커밋에도 추가되어있지않으면 `dangling` 개체로 취급되고 Packfile에 추가되지 않는다.

새로 생긴 파일은 `idex`파일과 `pack`파일이다. 파일시스템에서 삭제된 개체가 전부 이 Packfile에 저장된다. `Index` 파일은 빠르게 찾을 수 있도록 Packfile의 오프셋이 들어있다. `git gc` 명령을 실행하기 전 파일 크기는 8K정도였는데 새로 만들어진 Packfile은 겨우 4K에 불과하다.

어떻게 가능한일일까, 개체를 압축하면 Git은 먼저 이름이나 크기가 비슷한 파일을 찾는다. 그리고 두 파일을 비교해서 한 파일은 다른 부분만 저장한다. Git이 얼마나 공간을 절약해주는지 Packfile을 열어 확인할 수 있다.

`git verify-pack`명령으로 볼 수 있다.

```bash
$ git verify-pack -v \
  .git/objects/pack/pack-7a16e4488ae40c7d2bc56ea2bd43e25212a66c45.idx
0155eb4229851634a0f03eb265b69f5a2d56f341 tree   71 76 5400
05408d195263d853f09dca71d55116663690c27c blob   12908 3478 874
09f01cea547666f58d6a8d809583841a7c6f0130 tree   106 107 5086
1a410efbd13591db07496601ebc7a059dd55cfe9 commit 225 151 322
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a blob   10 19 5381
3c4e9cd789d88d8d89c1073707c3585e41b0e614 tree   101 105 5211
484a59275031909e19aadb7c92262719cfcdf19a commit 226 153 169
83baae61804e65cc73a7201a7252750c76066a30 blob   10 19 5362
9585191f37f7b0fb9444f35a9bf50de191beadc2 tag    136 127 5476
9bc1dc421dcd51b4ac296e3e5b6e2a99cf44391e blob   7 18 5193 1 \
  05408d195263d853f09dca71d55116663690c27c
ab1afef80fac8e34258ff41fc1b867c702daa24b commit 232 157 12
cac0cab538b970a37ea1e769cbbde608743bc96d commit 226 154 473
d8329fc1cc938780ffdd9f94e0d364e0ea74f579 tree   36 46 5316
e3f094f522629ae358806b17daf78246c27c007b blob   1486 734 4352
f8f51d7d8a1760462eca26eebafde32087499533 tree   106 107 749
fa49b077972391ad58037050f2a75f74e3671e92 blob   9 18 856
fdf4fc3344e67ab068f836878b6c4951e3b15f3d commit 177 122 627
chain length = 1: 1 object
pack-7a16e4488ae40c7d2bc56ea2bd43e25212a66c45.pack: ok
```

`9bc1d`가 `repo.rb`파일인데, 이 Blob는 두번째 버전인 `05408` Blob를 가리킨다. 개체에서 세번째는 압축된 개체의 크기를 나타낸다. 여기서 특이점은 원본을 그대로 저장하는 것이 첫번째가 아니라 두번째 버전이라는 것이다. 첫번째 버전은 차이점만 저장된다. 최신 버전에 접근할 때가 더 많고 속도가 더 빨라야하기 때문이다.

