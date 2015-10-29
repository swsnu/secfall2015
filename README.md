# Project 3: Man-in-the-middle attack

## Computer Security Fall 2015

### DUE: 11월 11일 11:59am KST

이번 과제는 웹 페이지 상에서 Man-in-the-middle (MITM) 공격을 수행하여 봄으로써 HTTPS에 대한 이해를 높이는 과제이다. 이 과제를 위해서 트래픽을 이해하기 위한 툴 중에 하나인 [Fiddler](http://www.telerik.com/fiddler)를 사용하고자 한다. Fiddler 는 프록시로서 웹 트래픽을 모니터링, 조사, 조작할 수 있는 확장 기능을 갖춘 프리웨어 디버깅 툴이다. Fiddler를 직접 사용해보며 수업 시간에 학습한 MITM 공격의 원리를 이해하고 어떻게 해야 안전하게 HTTPS를 사용할 수 있는지 이해한다.

1.  Fiddler 를 설치한다. Fiddler 는 Windows PC 에서만 설치가 가능하다.
2.  이번 과제를 위해 Fiddler 를 이용해 진행할 내용은 다음과 같다.
    *   과제는 다음 웹 페이지의 로그인 폼(Fake Login)에서 수행한다. ([웹 링크](https://147.46.248.177/))

    *   로그인 폼에 임의의 아이디와 비밀번호를 입력한 후, 서버로 전송되는 HTTPS 패킷을 Fiddler를 통해 확인한다.
    *   다음의 두 케이스를 관찰해야 한다.
        *   HTTPS 를 Decrypt 하지 않았을 때
        *   HTTPS 를 Decrypt 했을 때
    *   HTTPS 를 Decrypt 하기 위해서는 Fiddler 의 설정을 바꿔주어야 한다. ([링크](http://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/DecryptHTTPS))

### 리포트

리포트에는 다음의 내용이 포함되어 있어야 한다.

*   로그인 폼에서 전송되는 Decrypt되기 전/후 HTTPS 패킷정보 스크린 캡쳐
*   Decrypt된 패킷 스크린캡쳐에는 입력한 id 와 password 정보가 포함되어야한다.
*   HTTPS 통신에서 사용하는 SSL/TLS 의 동작 원리
*   프록시를 사용한 SSL/TLS Man-in-the-middle 공격 시나리오
*   Lessons Learned

### 제출 방식

리포트는 PDF 형식으로 {이름}.pdf 형태로 sec-tas@cmslab.snu.ac.kr 로 제출한다.

### 참고 문헌

[Download Fiddler](http://www.telerik.com/download/fiddler)
[Fiddler Documentation](http://docs.telerik.com/fiddler/configure-fiddler/tasks/configurefiddler)
