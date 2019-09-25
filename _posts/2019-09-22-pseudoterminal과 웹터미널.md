---
layout: post
title: pseudoterminal과 웹터미널
date: 2019-09-22 00:00:00 +00:00
desc: ""
keywords: "pseudoterminal, bash, pts, ptm"
categories: [Research]
tags: [pseudoterminal]
icon: icon-html
---

## pseudoterminal

pseudoterminal(가상 터미널, pty)은 실제 터미널 하드웨어를 유사한 소프트웨어적인 터미널이며, master와 slave라는 장치들의 쌍(pair)입니다.
slave는 bash와 같은 쉘(shell)과 연결된 장치입니다. `/dev/pts` 하위에서 확인해볼 수 있습니다.
master는 slave와 연결된 장치입니다. master에 데이터를 쓰면 slave의 입력으로 들어가게되며 master는 slave에 쓰인 데이터를 읽을 수 있습니다.
이처럼 master와 slave는 한쌍으로 서로 입출력이 연결되어 하나의 큰 채널로 볼 수 있습니다.

<figure><img src="/static/assets/img/blog/pseudoterminal.jpg" style="max-width: 100%;"/>
<figcaption>pseudoterminal</figcaption>
</figure>

### slave를 이용해서 SSH에 메시지 보내보기

SSH를 이용해 원격에서 로그인해서 쉘을 이용할 때, 역시 pseudoterminal이 사용됩니다. SSH에 접속하면 다음과 같은 과정을 거치게 됩니다.

1. SSH는 SSHD와 TCP 세션으로 연결됩니다. 그리고 아이디와 패스워드를 받습니다.
2. 로그인에 성공할 경우, SSHD는 새로운 pseudoterminal을 열고 slave(`/dev/pts/0`)에 새로운 쉘을 연결합니다.
3. 사용자로 부터 받은 데이터는 slave와 연결된 master에 쓰게됩니다. 반대로 slave로 부터 받은 데이터는 master를 통해 읽게 됩니다.

그렇다면 실제로 slave에 쓰게된다면 사용자 화면에 보일까요?

<div style="padding-left:1em;">
<div>[root@localhost ~]# echo "This is a message sent to myself" > /dev/pts/1</div>
<div><b>This is a message sent to myself</b></div>
<div>[root@localhost ~]# echo "This is a message sent to myself" > /dev/pts/1</div>
<div><b>This is a message sent to myself</b></div>
</div>

위 예제에서 `/dev/pts/1`은 확실히 원격의 사용자 터미널(ssh 클라이언트)과 연결되어 있는 것을 확인할 수 있습니다.

### PDIP를 이용한 PTY 구조 살펴보기

PDIP는 파라미터로 받은 프로그램을 실행하면서 slave에 연결시켜서 대화할 수 있도록 도와주는 프로그램 입니다. PDIP는 [명령어](http://rachid.koucha.free.fr/tech_corner/pty_pdip.html)를 통해
데이터를 읽고 쓸 수 있습니다. recv와 정규식을 통해 원하는 텍스트만 읽을 수 있으며, send를 통해 slave로 데이터를 전송할 수 있습니다.


<div style="padding-left:1em;">
<div>[root@localhost \~]# pdip -- ssh localhost</div>
<div><b>recv "password:"</b></div>
<div>root@localhost's password:<b>send "testtest\n"</b></div>
<div><b>recv "]# "</b></div>
<div></div>
<div>Last login: Tue Sep 24 09:43:11 2019 from localhost</div>
<div>[root@localhost \~]# <b>send "ls\n"</b></div>
<div><b>recv "]# "</b></div>
<div>ls</div>
<div>#.bash_profile#                     pdip-2.4.7-1.x86_64.rpm</div>
<div>#prod.properties#                   test.properties</div>
<div>Dockerfile                          abcd.properties</div>
<div>build.sh                            tutorial</div>
<div>[root@localhost \~]# <b>send "exit\n"</b></div>
<div>Connection to localhost closed.</div>
<div>exit</div>
<div>logout</div>
</div>

위 프로그램을 설명하면 다음과 같습니다.

1. ssh는 실행되면서 특정 slave 장치(`/dev/pts/1`)에 붙게됩니다. 사용자가 제어할 수 있는 터미널 환경을 제공하기 위해 생성됩니다.
2. `pdip -- ssh localhost`을 실행하면 pdip는 ssh와 연결된 master 장치에 붙게됩니다.
3. pdip는 master를 통해 slave로 데이터를 보내게되고 이를 ssh가 수신합니다.
4. ssh는 네트워크를 통해 sshd에 데이터를 보내게됩니다.
5. 로그인이 성공할 경우, sshd는 **2)**의 pdip 처럼 master 장치를 통해 쉘(bash, `/dev/pts/2`)과 데이터를 주고 받게됩니다.

### 참조

- http://rachid.koucha.free.fr/tech_corner/pty_pdip.html