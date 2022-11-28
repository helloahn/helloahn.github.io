---
layout: post
title: [CentOS] yum update, install시 cpio: rename 에러
subtitle: yum package install / update error
categories: CentOS
tags: [centos, yum, cpio, error]
---

CentOS는 기본 package manager로 yum을 사용하는데,

package install / update를 할 때 conflict 에러가 발생했다.

```shell
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for update
openssh-clients-7.4p1-22.el7_9.x86_64.rpm                                                              | 655 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : openssh-clients-7.4p1-22.el7_9.x86_64                                                                      1/2
Error unpacking rpm package openssh-clients-7.4p1-22.el7_9.x86_64
error: unpacking of archive failed on file /etc/ssh/ssh_config: cpio: rename
  Verifying  : openssh-clients-7.4p1-22.el7_9.x86_64                                                                      1/2
openssh-clients-7.4p1-13.el7_4.x86_64 was supposed to be removed but is not!
  Verifying  : openssh-clients-7.4p1-13.el7_4.x86_64                                                                      2/2

Failed:
  openssh-clients.x86_64 0:7.4p1-13.el7_4                       openssh-clients.x86_64 0:7.4p1-22.el7_9

Complete!
```

특정 환경설정 관련된 파일은 아래처럼 `immutable` 옵션이 붙는데,

리눅스 파일 시스템에서 시스템 보안 및 안정성을 유지하기 위한 파일 보호 옵션이다.

이 옵션이 붙어있다면 sudo 유저도 쉽게 파일을 건드릴 수 없게 된다.

파일 보호 옵션을 확인할 수 있는 명령어는 `lsattr`이며 아래와 같이 사용한다.

```shell
$ sudo lsattr /etc/ssh/ssh_config
----i----------- /etc/ssh/ssh_config
```

`----i-----------` 에서 i가 `immutable`옵션을 의미하고,

해당 파일을 수정하기 위해선 이 옵션을 제거해줘야 한다.

이를 위한 명령어는 `chattr`이며 사용법은 아래와 같다

```shell
$ sudo chattr -i /etc/ssh/ssh_config
```

`immutable`옵션을 제거한 뒤 다시 파일 옵션을 확인해보면
```shell
$ sudo lsattr /etc/ssh/ssh_config
---------------- /etc/ssh/ssh_config
```

`i`가 제거된 것을 확인할 수 있다.

이 상태에서 실패했던 package install / update를 수행할 수 있으며,

완료된 후에는 다시 `i`옵션을 추가해주어 파일을 보호해 준다.

```shell
$ sudo chattr +i /etc/ssh/ssh_config
$ sudo lsattr /etc/ssh/ssh_config
----i----------- /etc/ssh/ssh_config
```

fin.

<html>
  <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-5795250395612169"
      crossorigin="anonymous"></script>

  <script src="https://utteranc.es/client.js"
          repo="helloahn/helloahn.github.io"
          issue-term="pathname"
          theme="dark-blue"
          crossorigin="anonymous"
          async>
  </script>
</html>
