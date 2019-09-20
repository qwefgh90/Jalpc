---
layout: post
title:  윈도우 커버로스 (Kerberos Explained ) 번역
date:   2016-08-15 00:00:00 +09:00
desc: ""
keywords: "Kerberos,Windows,Securiy,Authentication,커버로스"
categories: [Cs]
tags: [Kerberos]
icon: icon-html
---


Mark Walla의 <a target="_blank" href="https://msdn.microsoft.com/en-us/library/bb742516.aspx">"Kerberos Explained"</a> 를 번역한 글입니다.

## 커버로스

본 2000년 5월에 발행된 Windows 2000 Advantage magazine에 기고된 글임

이 글은 커버로스 인증의 입문용으로 알려졌지만 높은 수준의 테크니컬 리뷰입니다. 커버로스는 Windows 2000 Active Directory 구현의 필수적인 부분입니다. 그리고 Windows 2000 엔터프라이즈를 사용하려는 사람들은 보안과 관련된 관리상의 이슈 및 기초에 대해 반드시 알고 있어야 합니다.

많은 운영체제 벤더들은 MIT에서 고안된 인증 프로토콜을 사용하고 있습니다. 커버로스 버전5는 시간이 갈수록 엔터프라이즈 수준의 운영 환경에서 더욱 중심적인 역할을 할 것입니다. 커버로스는 안전한 사용자 인증을 위한 산업 표준을 제공하고 있습니다. Active Directory 도메인 컨트롤러는 사용자 계정 및 로그인 정보를 유지하고 있으며 커버로스 서비스를 뒷받침하고 있습니다.

시스템에 접근할때 사용자의 신원을 확인하는 과정은 가장 처음에 위치합니다. 도메인에 가입하지 않는 로컬 머신의 경우에는 시스템에 접근하기전에 Windows NT LAN Manager protocol은 여전이 사용자의 이름과 비밀번호를 확인하고 있습니다. 반면 도메인에 가입한 머신의 경우에는 Active Directory와 커버로스 인증을 사용하고 있습니다. 일단 접근이 허가되면 다른 시스템 자원에 접근을 허락하는 일련의 티켓들이 교환되게됩니다.

목차는 다음과 같습니다.

- [Kerberos 101](#kerberos-101)
- [Understanding Kerberos concepts](#understanding-kerberos-concepts)
- [Further Clarification of the Log-in Process](#further-clarification-of-the-log-in-process)
- [Postscript](#postscript)

### Kerberos 101

Windows 2000 보안의 핵심은 사용자 인증입니다. Active Directory Services는 네트워크상에서 로그인을 구현하기 위해 인증 프로토콜이 필요하게 되었습니다. RFC 1510에 기반을 둔 커버로스 버전5 프로토콜은 분산된 환경과 이기종 운영체제의 상호운영을 위해 진보된 인증을 제공하고 있습니다.

**Defaulting to Kerberos**

NT LAN Manager는 Windows NT와 Windows 2000 워크 그룹 환경에서 사용된 인증 프로토콜입니다. 또한 Windows 2000 Active Directory 도메인 환경에서 Windows NT 시스템을 인증하기 위해 사용되고 있습니다. 더 하위 수준의 Windows NT 도메인 컨트롤러가 존재하지 않는 상태에서는 NT LAN Manager는 비활성화됩니다. 그리고 커버로스는 기본적인 인증으로 엔터프라이즈 환경에서 사용되게됩니다.

### Understanding Kerberos concepts

커버로스 버전5는 모든 버전의 Windows 2000의 표준이며 높은 보안수준을 보장합니다. 커버로스 프로토콜에서 커버로스라는 이름은 그리스 신화에서 3개의 머리를 갖는개로 알려져있습니다. 실제로 커버로스에서 3개의 머리는 키 분배 센터(Key Distribution Center, KDC), 클라이언트, 접근하려는 서비스를 운영하는 서버로 구성되어 있습니다. KDC는 도메인  컨트롤러의 일부분이며 2가지 서비스를 제공합니다. 첫번째는 인증 서비스(Authentication Service, AS), 두번째는 티켓 승인 서비스(Ticket Granting Service, TGS)입니다. 그림 1은 클라이언트가 서버에 접근하는 것과 관련 있습니다.

1. AS Exchange

2. TGS Exchange

3. Client/Server (CS) Exchange

 
<figure>
	<a href="https://i-msdn.sec.s-msft.com/dynimg/IC37117.gif"><img src="https://i-msdn.sec.s-msft.com/dynimg/IC37117.gif" alt=""></a>
	<figcaption>그림 1</figcaption>
</figure>

#### AS Exchange

처음 네트워크를 통해 로그인을 하였을때 사용자는 KDC의 AS에 로그인 이름과 비밀번호를 제공함으로써 교섭을 시도합니다. 그리고 KDC는 Active Directory 사용자 계정 정보에 접근하게됩니다. 인증이 성공할 경우 사용자는 TGT(Ticket to Get Tickets)를 부여받게 됩니다. TGT는 기본적으로 10시간동안 유효하며 TGT는 비밀번호를 다시 입력받지 않고 세션을 통해 갱신될 수 있습니다. TGT는 로컬 머신의 메모리에 저장되며 또한 네트워크 상으로 서비스에 세션을 요청하는데 사용됩니다. 다음은 TGT 검색 과정에 대해 논의하고 있습니다.

#### Example AS Administration

AS(Authentication Service) 는 텍스트에서 클라이언트를 확인하게됩니다. 만약 사전인증(preauthentication)이 허용되어 있다면 사용자의 암호 해시값을 암호키로서 타임스탬프를 암호화하게 됩니다. Active Directory에 저장된 사용자 암호 해시를 이요하여 유효한 시간을 암호화하여 읽어들이고 KDC는 요청이 이전 요청의 replay이지 아닌지 확인할 수 있습니다. 몇몇 어플리케이션을 지원하기 위해 특정 사용자에 대한 사전인증은 해제될 수 있습니다. Active Directory 사용자의 계정 탭에 접근한 다음 "Do not require Kerberos" 옵션을 체크할 수 있습니다.

<figure><a href="https://i-msdn.sec.s-msft.com/dynimg/IC86356.gif">
<img src="https://i-msdn.sec.s-msft.com/dynimg/IC86356.gif"/></a>
<figcaption>그림 2: 커버로스 사전인증 해제</figcaption>
</figure>

만약 KDC가 클라이언트의 요청을 승인하게 된다면 2가지의 응답(AS 응답)을 보내게됩니다. 첫번째는 KDC의 비밀키로 암호화된 TGT와 사용자의 암호 해시로 암호화된 세션키 입니다. (이 세션키는 뒤에서 KDC와 통신할때 필요한 것) 
클라이언트 시스템은 TGT 티켓의 내용을 읽지 못하기 때문에 TGS에 TGT를 제공하여 서비스 티켓을 요구합니다. TGT는 TTL(time to live), 인증 데이터, 세션키를 포함합니다.

#### TGS Exchange
사용자는 TGT(Ticket to Get Tickets)를 KDC안에 있는 TGS(Ticket Granting Service)에 전달하게 됩니다. TGS는 사용자의 TGT를 인증한 뒤 티켓과 세션키를 생성합니다. 이는 클라이언트와 서버 서비스를 위한 것입니다. 이 정보는 서비스 티켓으로 알려져 있으며 클라이언트 머신에 저장되게 됩니다.

TGS는 클라이언트의 TGT를 자신의 키를 이용해 읽게됩니다. 만약 TGS가 클라이언트의 요구를 승인한다면 클라이언트와 타켓 서버를 양쪽을 위해 서비스 티켓을 생성합니다. 클라이언트는 TGS 세션키(AS 응답으로 받았던)를 사용하여 서비스 티켓의 일부분을 읽을 수 있습니다. 클라이언트는 접속하려는 서버에 TGS의 서버 부분을 전달하게됩니다.

#### Client/Server Exchange
일단 사용자가 클라이언트/서버 서비스 티켓을 갖게되면 서버 서비스와 세션을 맺을 수 있습니다. 서버는 KDC로 부터 받은 자신의 키를 이용해 클라이언트로 부터 전송된 TGS 정보를 복호화할 수 있습니다. 그리고 서비스 티켓은 사용자를 인증하게되고 서버와 클라이언트는 세션을 맺게 됩니다. 티켓의 수명이 다하게 되면 서비스 티켓은 반드시 갱신되어야 합니다.

#### Client/Server Exchange Detail
클라이언트는 세션을 맺기위해 서비스 티켓의 서버 부분을 서버에 전달하게 됩니다. 만약 상호 인증이 활성화 되어있다면 타겟 서버는 서비스 티켓의 세션키로 암호화된 타임스탬프를 반환합니다. 만약 타임 스탬프가 정상적으로 복호화 된다면 클라이언트는 스스로 서버에게 인증받았을 뿐 아니라 서버 또한 클라이언트에게 인증받은 것 입니다. 타겟 서버는 절대로 KDC와 직접 통신할 수 없으며 이것은 KDC의 다운타임과 압박을 줄일 수 있습니다.

