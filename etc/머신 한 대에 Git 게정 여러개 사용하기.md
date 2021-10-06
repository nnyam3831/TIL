## 로컬 머신에서 Git 계정 여러 개 사용하기

회사 git 계정, 개인 git 계정 분리

#### ssh 키 생성

```zsh
$ ssh-keygen -t rsa -b 4096 -C [메일]
$ ssh-keygen -t rsa -b 4096 -C "nnyam3831@github.com"
```

```zsh
Enter file in which to save the key (~/.ssh/id_rsa): id_rsa_personal
Enter passphrase (empty for no passphrase):
Enter same passphrase again:

Your identification has been saved in ~/.ssh/id_rsa_personal.
Your public key has been saved in ~/.ssh/id_rsa_personal.pub.
The key fingerprint is:
SHA256:7guBGmNZUorbRfiwhpePtkcawmO1TjAVikzCl5EXAA8 yangwookee@gmail.com
The key's randomart image is:
+---[RSA 4096]----+
|oE.=O..          |
|=o*O .           |
|o+=*+            |
|.==*..           |
|ooXoo . S        |
|.=+*o  o         |
|.o==  . .        |
|  o..  o         |
|   .    o.       |
+----[SHA256]-----+
```

이러면 id_rsa_personal key 쌍이 하나 생김, 동일한 방법으로 id_rsa_work도 생성

#### ssh key 등록

위에서 생성한 public ssh key를 복사해서 각각 개인 github, 회사 github의 ssh key에 등록해준다.

```zsh
$ pbcopy < ~/.ssh/id_rsa_personal.pub
```

github -> settings -> SSH and GPG keys 에 private key값 입력

#### ssh-agent에 키 등록

매번 git 명령어 쳐줄때마다 권한 로그인하기 귀찮으니까 ssh-agent 등록해줘야 한다.

```zsh
$ eval "$(ssh-agent -s)"
$ ssh-add -K ~/.ssh/id_rsa_personal
```

#### ssh config

각각의 git 계정을 사용하려면 ~/.ssh 디렉토리에 ssh 설정파일을 만들어줘서 명시해야 한다.

```sh
# 개인용
Host personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_personal

# 회사용
Host work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_work
```

이 때, 회사용 HostName은 회사 도메인을 사용해야 함

config 파일을 작성했으면 아래와 같이 테스트 해볼 수 있다.

```zsh
$ ssh -T git@personal
$ ssh -T git@work
```

#### gitconfig, gitconfig-\*

git@{Host} 를 통해서 각각의 git 계정을 사용할 수 있지만, 아예 폴더별로 설정파일을 작성해주면 따로 git 계정을 입력하지 않고 default로 사용할 수 있다.

아래와 같이 Projects 폴더를 한다고 하면

```
├── work # 회사 코드
└── seongwon # 개인 코드
```

.gitconfig 파일에 아래와 같은 구문을 추가

```
[includeIf "gitdir:{폴더명}/"]
        path = .gitconfig-{프로파일명}

```

개인, 회사를 각각 personal, work라는 이름의 profile로 만들었으니 다음과 같이 작성하면 된다. 이 때, 디렉토리가 git 저장소가 아니여도 하위 디렉토리의 git 저장소들은 설정한 .gitconfig-\*의 config를 따라간다.

```
[includeIf "gitdir:~/work/"]
    path = .gitconfig-work
[includeIf "gitdir:~/seongwon/"]
    path = .gitconfig-personall
```

이제 gitconfig-work, gitconfig-personal에 해당 정보를 입력해준다.

```zsh
[user]
        email = {이메일 주소}
        name = {이름}
[github]
        user = {유저명}
```

참고로, .gitconfig 파일은 Users/users에 있음
