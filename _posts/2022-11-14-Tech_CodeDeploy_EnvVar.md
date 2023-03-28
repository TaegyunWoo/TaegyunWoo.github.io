---
category: Tech
tags: [Tech]
title: "AWS CodeDeploy 에서 시스템 환경변수 사용하기"
date:   2022-11-14 14:00:00 
lastmod : 2022-11-14 14:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

AWS CodeDeploy 를 사용해서 배포를 할 때, 배포할 서버의 환경변수를 사용해야 하는 경우가 있다.

나는 아래와 같은 상황이었다.

- Spring Boot 애플리케이션을 배포할 서버는 테스트, 프로덕션 총 2개이다.
- **`properties` 파일은 테스트 서버용과 프로덕션 서버용으로 구분된다.**
- "배포된 애플리케이션을 실행시키는 쉘 스크립트"를 CodeDeploy 가 소스코드·jar 배포 후 실행한다.

<br/>

이때 가장 큰 고민은 2번째 항목이었다.

애플리케이션을 실행시키는 쉘 스크립트는 하나이고, 하나의 스크립트로 테스트/프로덕션 서버를 구분하여 각자 다른 `properties` 파일을 사용하도록 해야했다.

예를 들면 아래와 같다.

```bash
# ---[애플리케이션 실행 스크립트]---

# 테스트 서버인 경우 실행해야하는 명령어
nohup java -jar \
  -Dspring.profiles.active="test" \
  /home/ec2-user/app/build/libs/*.jar

# 프로덕션 서버인 경우 실행해야하는 명령어
nohup java -jar \
  -Dspring.profiles.active="production" \
  /home/ec2-user/app/build/libs/*.jar
```

<br/>

그래서 나는 환경변수를 사용하여 문제를 해결하려고 했다.

아래와 같이 말이다.

```bash
# ---[애플리케이션 실행 스크립트]---
nohup java -jar \
  -Dspring.profiles.active="$SPRING_PROFILES_ACTIVE" \
  /home/ec2-user/app/build/libs/*.jar
```

스크립트를 위와 같이 작성하고, 테스트 서버의 환경변수로 `SPRING_PROFILES_ACTIVE=test` 를 저장하고, 프로덕션 서버의 환경변수로 `SPRING_PROFILES_ACTIVE=production` 을 설정했다.

그리고 CodeDeploy 로 배포를 시작했는데…

**CodeDeploy 가 이 배포 스크립트를 실행할 때, `$SPRING_PROFILES_ACTIVE` 환경변수를 읽어오지 못했다!**

그 이유는 환경변수를 설정한 사용자 (`ec2-user`)와 CodeDeploy가 ec2에 접근할때 사용하는 사용자가 다르기 때문으로 추정된다.

**즉 `ec2-user` 가 설정한 환경변수를 `codedeploy` 가 접근할 수 없기 때문이다.**

따라서 `codedeploy` 가 직접 환경변수를 설정하도록 해야한다.

<br/><br/>

## CodeDeploy 가 접근할 수 있는 환경변수

그렇다면 어떻게 해야, CodeDeploy가 환경변수를 설정하도록 할 수 있을까?

<br/>

### 그 답은 바로, `/etc/profile.d/codedeploy.sh`

`/etc/profile.d/codedeploy.sh` 은 소스코드 배포를 위한 CodeDeploy 사용자가 가장 먼저 실행하는 스크립트 파일이다.

주로 환경변수를 설정하기 위해 사용된다.

따라서 이 파일에 원하는 환경변수를 설정하면 된다.

```bash
# /etc/profile.d/codedeploy.sh 에 추가할 내용
export 환경변수명=값
```

나의 경우, 아래와 같이 작성했다.

```bash
# 테스트 서버의 /etc/profile.d/codedeploy.sh
export SPRING_PROFILES_ACTIVE=test
```

```bash
# 프로덕션 서버의 /etc/profile.d/codedeploy.sh
export SPRING_PROFILES_ACTIVE=production
```

그리고 ec2 에서 실행되고 있는 `codedeploy-agent` 를 재실행해야 한다.

아래 명령어를 사용하여 재실행하자.

```bash
$ sudo service codedeploy-agent restart
```

`codedeploy-agent` 가 재실행되고 나서, 다시 배포를 시도해보면 환경변수를 아주 잘 읽는 것을 확인할 수 있다!

<br/><br/>

## 마치며..

지금까지 AWS CodeDeploy 가 접근할 수 있는 환경변수를 설정하는 방법에 대해 알아봤다.

이 문제로 나는 꽤 많이 헤맸다. 하지만 이 글을 읽는 독자들은 그렇지 않길 바라며 마무리한다.

<br/><br/>

## References

- [https://security.stackexchange.com/questions/14000/environment-variable-accessibility-in-linux](https://security.stackexchange.com/questions/14000/environment-variable-accessibility-in-linux)
- [https://unix.stackexchange.com/questions/64258/what-do-the-scripts-in-etc-profile-d-do](https://unix.stackexchange.com/questions/64258/what-do-the-scripts-in-etc-profile-d-do)