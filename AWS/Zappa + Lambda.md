# AWS Lambda

zappa를 통해 Django를 배포해보자. IAM Credential만 잘 설정하면 참 쉽다..



## 람다란?

**Serverless**, 말 그대로 서버가 존재하지 않는다는 뜻이다. 실제로 작업을 처리하는 서버가 존재하지만 이를 신경쓰지 않아도 된다. 관리자가 서버의 사양, 확장성 등 모두 고려하지 않아도 **모든것을 자동으로 처리해준다.**



### Lambda, 장점

- 서버 확장성, 가용성에 대해 관리할 필요가 없다.
- 코드가 실행될때만 비용이 청구된다.



### Lambda, 단점

- 실행시간이 5분 이상 걸릴경우 자동으로 폐기되므로 오랜 시간을 요구하는 작업에는 적합하지 않는다.
- 초기 지연시간이 발생할 수 있다.



## Guide to using Django with Zappa

[원본](https://edgarroman.github.io/zappa-django-guide/)



### 순서

1. AWS Account Credentials
2. Environment Settings



### 1. AWS Account Credentials

사전작업으로는 다음이 필요하다.

- S3 Bucket 생성
  - region에 주의하여 생성하도록하자.
  - 이름은 `zappatest-code`라고 한다.
- IAM 유저 생성
  - 자세한 내용은 아래 섹션에서 설명한다.



#### 1-1. IAM 설정

1. https://console.aws.amazon.com/iam/home#/users 로 이동한다.
2. 사용자 추가
3. 사용자 이름 자유, 프로그래밍 방식 엑세스
4. 권한은 아무것도 설정하지 않는다.
5. 태그 설정도 없다.
6. 사용자 만들기를 완료한다.
7. `ACCESS_KEY`, `SECRET_ACCESS_KEY`를 **1-2** 항목을 참고해 채워넣는다.
8. 이후 다시 https://console.aws.amazon.com/iam/home#/users 로 이동한다.
9. 생성한 사용자를 클릭한다.
10. 인라인 정책 추가를 누른다. JSON 탭을 클릭한다.
11. 다음 내용에 꺾쇠부분을 채워넣어 붙여넣는다.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "lambda:CreateFunction",
                "lambda:ListVersionsByFunction",
                "logs:DescribeLogStreams",
                "events:PutRule",
                "route53:GetHostedZone",
                "s3:CreateBucket",
                "iam:CreateRole",
                "lambda:GetFunctionConfiguration",
                "cloudformation:DescribeStackResource",
                "iam:AttachRolePolicy",
                "iam:PutRolePolicy",
                "apigateway:DELETE",
                "events:ListRuleNamesByTarget",
                "apigateway:PATCH",
                "cloudformation:UpdateStack",
                "events:ListRules",
                "lambda:DeleteFunction",
                "events:RemoveTargets",
                "logs:FilterLogEvents",
                "lambda:GetAlias",
                "apigateway:GET",
                "events:ListTargetsByRule",
                "cloudformation:ListStackResources",
                "iam:GetRole",
                "events:DescribeRule",
                "apigateway:PUT",
                "lambda:GetFunction",
                "route53:ListHostedZones",
                "lambda:UpdateFunctionConfiguration",
                "route53:ChangeResourceRecordSets",
                "cloudformation:DescribeStacks",
                "lambda:UpdateFunctionCode",
                "events:DeleteRule",
                "events:PutTargets",
                "lambda:AddPermission",
                "cloudformation:CreateStack",
                "cloudformation:DeleteStack",
                "apigateway:POST",
                "lambda:RemovePermission",
                "lambda:GetPolicy"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "iam:PassRole",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::<버킷이름, 여기서는 zappateset-code>",
                "arn:aws:iam::<사용자 ID, 모르겠으면 아래 참고>:role/*-ZappaLambdaExecutionRole"
            ]
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:ListBucketMultipartUploads",
                "s3:AbortMultipartUpload",
                "s3:DeleteObject",
                "s3:ListMultipartUploadParts"
            ],
            "Resource": "arn:aws:s3:::<버킷이름, 여기서는 zappateset-code>/*"
        }
    ]
}
```

​	사용자 ID는 https://console.aws.amazon.com/iam/home?region=ap-northeast-2#/home 에 접속했을 때 `IAM 사용자 로그인 링크: `의 링크에 12자리의 숫자이다. (`https://<사용자 ID>.signin.aws.amazon.com/console`)

다른 권한이 필요하게되면 그때 하나씩 추가하는것을 권장한다.





#### 1-2. 로컬에 Credentials, Config 설정

`~/.aws/credentials`파일을 수정한다. 기존에 키가 있다면 뒤에 작성하면 된다.

레이벨의 이름은 자유롭게 설정하면 된다.

```shell
[zappa]
aws_access_key_id = your_access_key_id_specific_to_zappa
aws_secret_access_key = your_secret_access_key_specific_to_zappa
```

`~/.aws/config` 파일을 수정한다.

```shell
[default]
region = ap-northeast-2
output = json
```





### 2. Environment Settings



#### 2-1. Baseline packages

- Python 3.6 or 2.7 (AWS lambda의 지원 목록을 따른다.)
- 최신 장고
- 최신버전의 zappa
- zappa가 동작할 수 있는 가상환경
  - 여기에는 아래 두가지 방법 중 하나를 사용할 수 있다.



##### 2-1-1. Approach #1 - Local Machine

1. Python과 pip 및 virtualenv를 설치한다.
2. 프로젝트를 생성한다.

```shell
mkdir zappatest
cd zappatest
virtualenv ve
source ve/bin/activate
pip install django zappa
```



##### 2-1-2. Approach #2 - Docker with zappa (recommaned)

Docker사용이 동일한 환경을 팀의 다른 사람들과 공유하는것이 더 쉽기때문에 추천된다.

버전에 따라 아래 두개중 하나를 설치하면 된다.

- Python 2.7 ([lambci/lambda:build-python2.7](https://hub.docker.com/r/lambci/lambda/tags/))
- Python 3.6 ([lambci/lambda:build-python3.6](https://hub.docker.com/r/lambci/lambda/tags/))



###### Initial Setup

- [Install Docker](https://docs.docker.com/engine/installation/)
- zappa 이미지를 로컬에 설치

```shell
# For Python 2.7 projects
docker pull lambci/lambda:build-python2.7
# For Python 3.6 projects
docker pull lambci/lambda:build-python3.6
```



###### Project Setup

- 배포환경세팅을위한 Docker파일 생성

```dockerfile
FROM lambci/lambda:build-python3.6

LABEL maintainer="<your@email.com>"

WORKDIR /var/task

# Fancy prompt to remind you are in zappashell
RUN echo 'export PS1="\[\e[36m\]zappashell>\[\e[m\] "' >> /root/.bashrc

# Additional RUN commands here
# RUN yum clean all && \
#    yum -y install <stuff>

CMD ["bash"]
```

- Docker Build

```dockerfile
$ cd /your_zappa_project
$ docker build -t myzappa .
```



###### Create zappashell alias

빠른 Docker 실행을위해 명령어로 저장한다.

`AWS_PROFILE`은 credential에 Lambda 배포를 위해 작성한 레이벨 이름을 지정한다.

```shell
$ alias zappashell='docker run -ti -e AWS_PROFILE=zappa -v "$(pwd):/var/task" -v ~/.aws/:/root/.aws  --rm myzappa'
$ alias zappashell >> ~/.bash_profile
```

필자의 경우 Ubuntu + Zsh을 사용하기 때문에 조금 다르게 설정했다.

```shell
$ alias zappashell='sudo docker run -ti -e AWS_PROFILE=zappa -v "$(pwd):/var/task" -v ~/.aws/:/root/.aws  --rm myzappa'
$ alias zappashell | { read s; echo "alias ${s}" } >> ~/.zshrc
```



###### Create the Virtual Environment

가상환경에 필요한 라이브러리 설치

```python
$ zappashell
zappashell> python -m venv ve
zappashell> source ve/bin/activate 
(ve) zappa> pip install -r requirements.txt
```

가상환경의 현재 디렉터리는 로컬 시스템의 프로젝트에 매핑되어있다. `virtualenv`가 현재 폴더에 `ve` 폴더를 만들고 그 안에 Python 라이브러리들이 설치되어 컨테이너가 종료되어도 유지될 수 있지만 , 시스템라이브러리의 경우 컨테이너가 종료되면 손실된다. 해결책은 Dockerfile에 `RUN` 명령어를 사용하여 설치를 유지하는것이다.



###### Using your Environment

```shell
$ cd /your_zappa_project
$ zappashell
zappashell> source ve/bin/activate
(ve) zappashell> 
```



### 3. Django Setup

```shell
(ve) zappashell> django-admin startproject config
(ve) zappashell> mv config app
```

이후 config 설정에서 바꾸어주어야할것이 있다. Lambda가 SQLite3의 지원을 하지 않는다. 그러므로 PostgreSQL을 사용하던가, zappa-django-utils를 사용하여 S3에 SQLite3를 두어야한다.

AWS RDS - PostgreSQL을 사용하도록 한다.



> **주의 사항**
>
> 프로젝트의 구성에따라 Lambda 배포 전략을 다르게 가져야한다.
>
> ```
> ~/your_zappa_project/app/
> 
> app/       # project dir (django-admin.py로 생성도니 프로젝트)
> config/
> 	settings/         # 설정을 패키지로 분리했을 때
>      __init__.py
>      production.py
>      development.py
>      ...
>  __init__.py
>  urls.py
>  wsgi.py
> users/       # django-admin.py로 생성된 앱
> ```
>
> 필자가 사용한 프로젝트 구성은 다음과 같다. 이 때 `your_zappa_project`에서 `zappa deploy`를 수행하였을 때 setting에서 프로젝트의 모듈의 위치를 불러오려하면 `ModuleNotFoundError`가 발생한다. 대표적으로 `ROOT_URLCONF`의 `config.urls`를 불러오지 못한다. 이를 성공적으로 작동시키려면 `app.config.urls`로 작성해야한다. 
>
> 이 때문에 필자는 `app`에서 `zappa deploy`를 수행한다. 프로젝트에 반드시 필요한 설정들을 `app`안에 넣고 배포에 필요하지 않은 자원, 테스트들을 `projects_root_dir`에 놓고 사용한다.



### 4. Zappa init

주의사항을 참고해 **app**에서 배포를 하도록한다.

```shell
(ve) zappashell> mv app
(ve) zappashell> zappa init
```



Zappa 명령어로 간단하게 **zappa_setting** 파일을 생성할 수 있다.

```
# Lambda는 다양한 배포환경을 지원해준다.
Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
What do you want to call this environment (default 'dev'): dev

# credentials를 확인해 자동으로 프로필을 읽어온다. 사용하고자하는 프로필을 적는다.
AWS Lambda and API Gateway are only available in certain regions. Let's check to make sure you have a profile set up in one that will work.
We found the following profiles: default, and zappa. Which would you like us to use? (default 'default'): zappa

# 배포에 사용될 Bucket의 이름을 작성한다.
Your Zappa deployments will need to be uploaded to a private S3 bucket.
If you don't have a bucket yet, we'll create one for you too.
What do you want to call your bucket? (default 'zappa-qu0msfhly'): 

# Django Settings의 위치를 적는다.
It looks like this is a Django application!
What is the module path to your projects's Django settings?
(This will likely be something like 'your_project.settings')
Where are your project's settings?: config.settings.production

# Global Service를 위해 모든 region에 배포할지 물어본다.
You can optionally deploy to all available regions in order to provide fast global service.
If you are using Zappa for the first time, you probably don't want to do this!
Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]: 
```



설정을 완료하면 해당 디렉터리에 `zappa_settings.json`이 생기게된다. 만약 그 json 파일에 **aws_region**이 없다면 명시하도록하자.

```json
{
    "dev": {
        "aws_region": "ap-northeast-2",
        "django_settings": "config.settings.production",
        "profile_name": "default",
        "project_name": "django-base",
        "runtime": "python3.6",
        "s3_bucket": "zappa-qu0msfhly"
    }
}
```



### Zappa Deploy

위의 작업을 모두 완료했으면 선택한 배포환경으로 배포할 수 있다.

```
(ve) $ zappa deploy dev
Calling deploy for environment dev..
Downloading and installing dependencies..
100%|███████████████████████████████████████████████████████████████████████████████████████████| 27/27 [00:07<00:00,  3.91pkg/s]
Packaging project as zip..
Uploading zappatest-dev-1482425936.zip (13.1MiB)..
100%|████████████████████████████████████████████████████████████████████████████████████████| 13.8M/13.8M [00:25<00:00, 603KB/s]
Scheduling..
Scheduled zappatest-dev-zappa-keep-warm-handler.keep_warm_callback!
Uploading zappatest-dev-template-1482425980.json (1.5KiB)..
100%|███████████████████████████████████████████████████████████████████████████████████████| 1.58K/1.58K [00:00<00:00, 2.08KB/s]
Waiting for stack zappatest-dev to create (this can take a bit)..
100%|█████████████████████████████████████████████████████████████████████████████████████████████| 4/4 [00:18<00:00,  4.69s/res]
Deploying API Gateway..
Deployment complete!: https://x6kb437rh.execute-api.us-east-1.amazonaws.com/dev
```

**Deploy complete**가 뜬다면 성공적으로 완성한것이다. 만약 **Access**관련 문구가 보였다면 IAM에 가서 추가하도록한다.



주어진 링크로 접속하게되면 `ALLOWED_HOST`관련 에러가 발생할 것이다. 이 문제를 해결하기위해 settings에 `ALLOWED_HOST`를 추가하였으면 재 배포를 수행한다. 배포된 상태에서 재 배포는 **update**를 사용한다.

```
(ve) $ zappa update dev
```



만약 다른 에러가 발생한다면 아래 명령어를 통해 확인 할 수 있다.

```
zappa tail
```



## AWS RDS 접근 허용

기본적으로 RDS inbound에 Lambda는 허용되지 않는다.

Lambda의 보안그룹을 생성하여 RDS inbound에 포함될 수 있도록한다.

```
{
    "dev": {
        "django_settings": "frankie.settings", 
        "s3_bucket": "zappatest-code",
        "aws_region": "us-east-1",
        "vpc_config": {
            "SubnetIds": [ "subnet-f3446aba", "subnet-c5b8c79e"],
            "SecurityGroupIds": [ "sg-9a9a1dfc"]
        }
     }
}
```

이후 RDS inbound에 이곳에서 명시한 SecurityGroup을 추가하면 RDS에 추가될 수 있다.





## 로그에 관련하여

장고의 로그는 장고 내부에서 모두 처리되므로 Lambda 설정의 `Exception Handler`로 받아볼 수 없다.

https://github.com/Miserlou/Zappa/issues/1609#issuecomment-422623554

`zappa tail`시에는 Django에서 설정한 로그가 나오게된다.
