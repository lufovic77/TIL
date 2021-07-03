# 여러 Git 계정 관리하기
회사에 들어가게 되면 개인 깃 계정 뿐만 아니라 회사의 깃 계정도 사용하게 된다. 

이때 본인의 개인 계정을 global하게 정의한 경우 잘못된 정보로 커밋이 올라가기 십상이다. 

이런 경우를 디렉토리별 계정 분리를 위해 `.gitconfig` 파일을 수정하면 된다. 

## .gitconfig 파일 수정하기

```java
$ vim ~/.gitconfig
```

로 열어보면, 대부분의 사람들이 보통 아래의 양식을 가진 내용이 들어있을것이다. 

```java
[user]
		 name = lufovic77
     email = lufovic77@gmail.com
```

이런식으로 미리 깃헙 계정을 선언을 해두고 사용을 할 텐데, 

여기서 따로 회사 계정을 추가하는 방식으로 수정하면 된다. 

```java
[user]
		 name = lufovic77
     email = lufovic77@gmail.com
[includeIf "gitdir:{회사용작업폴더경로}"]
     path= .gitconfig-work
```

이런식으로 아래에 `includeIf`를 걸어두면 된다. 

- `{회사용작업폴더경로}`는 본인 로컬 환경에 있는 회사용 디렉토리 경로로 바꾸면 된다. 예를 들자면 아래와 같이 바꾸면 된다.

```java
[includeIf "gitdir:~/Desktop/work/"]
```

- `path= .gitconfig—work`는 위의 회사용 디렉토리에서 작업한 경우 기존의 `.gitconfig`가 아니라 새로 정의한 `.gitconfig-work` 파일로 넘어간다는 의미다.
- 

## .gitconfig-?? 파일 만들기

.gitconfig와 같은 경로에 새로운 .gitconfig-work 같은 파일을 만들어주자. 

내용은 같으며, 회사용 계정으로 변경해주면 된다. 

```java
[user]
     email = abcd@corp.com
     name = name
[github]
     user = username
```

- 이때, 맨 아래의 `user`는 github에서 사용되는 본인의 유저 이름이다.

## 증명 정보 추가하기

이렇게 만들어 두면 회사 디렉토리에서 작업한 내용을 커밋하고 푸시할 때 회사 계정으로 잘 올라가게 된다. 

다만, 이러면 푸시를 할 때마다 로그인을 해야하는 번거로움이 있다. 

그런경우 SSH 키를 이용해서 깃헙에 등록하여 사용하면 자동으로 자격 증명이 되어 푸시가 이어지는데, 이에 대한 자세한 내용은 아래 블로그를 참고해서 완료해보자. 

[https://yangeok.github.io/git/2020/03/08/ssh-multiple-account.html](https://yangeok.github.io/git/2020/03/08/ssh-multiple-account.html)
