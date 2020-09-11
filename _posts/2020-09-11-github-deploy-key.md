---
layout: post
title:  "github 의 deploy key"
date:   2020-09-11 17:53:57 +0900
categories: jekyll update
comments: true
---
[이 포스트](https://blog.leocat.kr/notes/2017/11/27/github-deploy-key)와 [공식 문서](https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys)를 많이 참고했습니다. 

Github의 private repository는 허용된 사용자만이 접근가능합니다. 따라서 로컬 환경에서 pull을 받거나 push를 할 때 github 서버로부터 인증을 받는 과정이 필수입니다. 

하지만 여러 사람들이 사용하는 공용 서버일 경우 개인의 로그인 정보를 남기는 것이 사실상 꺼려집니다. 이럴때는 각각의 개인을 인증하는 대신, 공용 서버 자체를 인증하는 방법이 있습니다. 바로 Deploy key를 이용하는 방법입니다. 

Github이 해당 repository에 public키(공개키)를 등록하면, 공용 서버에 있는 private 키(개인키)로 인증을 할 수 있습니다. public-private 한 쌍의 비대칭키로 인증하는 방식입니다. 

- 비대칭 키란, 암호화와 복호화에 쓰이는 키가 각기 다른 암호 방식인데요, 사람들에게 공개되어 있어서 정보를 암호화 할 수 있는 **공개키**와, 본인만 알고 있어서 암호를 해석할 수 있는 **개인키**가 쌍을 이루고 있습니다. 공개키로 암호화를 했다면 개인키외에는 복호화를 할 수 없으므로 **보안**에 중점을, 개인키로 암호화를 했다면 공개키로 누구나 복호화가 가능하기 때문에 **내가 누구다** 라고 인증하는 용도로 쓰이게 됩니다. (공개키로 복호화가 가능하다는 뜻은 누구에게나 공개되지 않은 개인키로 암호화가 되었다는 뜻이니까요)

- 따라서 github에게 내가 이 사람이다! 라고 **인증**을 하기 위해선 공개키를 deploy key에 넘겨주는 작업이 필요합니다. 그 간단한 과정을 한번 돌아보겠습니다. 

# 1. RSA 키 생성
- RSA는 소인수분해를 이용한 공개키(비대칭키) 알고리즘입니다. 다음과 같이 생성할 수 있습니다. 
```javascript
    ssh-keygen -t rsa
```
- cat ~/.ssh/id_rsa.pub 으로 공개키를 확인할 수 있습니다.

# 2. Github Deploy Key에 공개키 등록
- Github > Repository > Settings 의 Deploy key 로 갑니다. 
- title에는 태그하고 싶은 이름을, key에는 1에서 얻은 공개키를 입력합니다. 

# 3. 접속 확인
- git clone (pull)을 받고자 하는 디렉토리에서 다음 명령어를 실행합니다. 나오는 질문들은 yes로 넘어가면 됩니다. 
```
ssh -T git@github.com
```
- **이 때, git clone은 http인증이 아닌 ssh 인증으로 받아야 합니다.** 즉, https://github.com/repo/myproject.git 형식이 아닌 git@github.com:repo/myproject.git 형식으로 받아야 합니다. 

처음에는 많이 헤맸는데 이렇게 하고보니 매우 간단하네요! 인증을 위해서라면 공개키를 건넨다! 라는 부분만 잘 이해하면 되는 것 같습니다.
