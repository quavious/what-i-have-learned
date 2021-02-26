+++
title = "분산처리 강의 기록 2020-11-09"
date = "2021-02-26T19:08:30+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["분산처리", "시스템", "HTTP", "SMTP", "Stateless", "Stateful", "Protocol", "FTP", "DNS", "Email"]
keywords = ["분산처리", "시스템", "HTTP", "SMTP", "Stateless", "Stateful", "Protocol", "FTP", "DNS", "Email"]
description = ""
showFullContent = false
+++

# distributed 2020-11-09

http protocol 특징 - 무상태 프로토콜  
Stateful, Stateless 비교할 것  
http 통신을 한 내용이 다음 통신에 영향을 미치지 않는다. - Cell Contained  
FTP - In band Signal, Out of band Signal 2차선 데이터(파일) 송수신 채널과 명령어 및 신호 송수신 채널이 각각 존재  
파일을 보내는 명령어 및 신호

### HTTP Stateless OR Stateful protocol

### FTP Signaling

- 전화를 걸 때 신호를 전송, 음성 데이터 자체를 전송하지 않음

### DHCP

broadcast - 타 네트워크로 브로드캐스팅하기 위해선 Agent 필요  
Unicast 메시지로 바꾸어 준 다음 에이전트에 전송 Encapsulation(캡슐화)

- Unicast

### DNS

분산 이름 서버 - 분산 DB  
Iterative OR Recursive

### E-Mail - Push OR Pull Protocols

원칙적으로 이메일 및 HTTP 서버 접속 후 메일 데이터를 가져옴(PULL)

## EMAIL을 통해 Push와 Pull 이해

비동기적 통신 매개체  
언제든지 보내고 읽을 수 있다. 타인의 일정을 신경쓸 필요가 없음

### EMAIL의 3가지 주요 요소

1. User-Agent
1. Mail Servers
1. Protocols - 프로토콜 특성 : 실제 프토토콜 방식
   - Push Protocol : SMTP
   - Pull Protocol : Mail Access Protocols
     - POP3
     - IMAP

#### User-Agent

메일 작성 및 발송 - Composing  
송신자가 소유한 메일 서버까지만 메일을 보낼 수 있다.  
예: 단국대 학생 - 단국대 메일 서버  
또한 메일 서버의 박스로부터 메일을 가져을 수 있다.

#### Mail Server

이메일 인프라의 핵심 부분  
Mailbox는 사용자로부터 들어오는 메일을 보관한다.

메일 송수신 순서

1. 송신자가 메일 전송
1. 송신자가 보낸 메일이 메일 서버 메시지 큐에 쌓임
1. 수신자의 메일서버, 메일 박스에 전달됨
1. 수신자의 유저 에이전트

#### SMTP Server

Client Side 클라이언트 사이드 - 송신자의 메일 서버  
Server Side 서버 사이드 - 수신자의 메일 서버

포트 25  
메일 전송 3단계

1. 핸드세이킹 - TCP 연결
1. 메시지 교환
1. 닫음  
   Commands 아스키 문자 - 7비트의 문자만 작성 가능  
   이미지는 아스키코드로 변경되어 전송됨 - 바이너리 멀티미디어 파일은 아스키코드로 인코딩된다.  
   Persistent Connections  
   모든 메시지는 같은 TCP 연결을 통해 전송된다.  
   HTTP - Pull, SMTP - Push

##### SMTP

Two Step procedure  
타 메일 서버와 직접적으로 연결되지 않음

#### Mail Retrieval

내 PC에는 Agent만 내장됨

### POP# Signal

TCP 110 포트
인증 절차

1.  Userorization "phase"

#### Phase update

POP3 단점 - 모든 장소에서 접속 가능하여 언제즌지 요구됨

33

#### Web Access protocls

User-Agent는 개인 웹 브라우내 존재 HTTP를 이용하여 원격 메일박스와 통신  
Send - 메시지 전송, Access - 이메일 메시지, 타 사용자로부터 받은 메일

### Video Streaming - Content Distridution Networks

Streaming Prerecorded Video : Netflix, YouTube  
On Demand - 소비자가 요구할 때 가지고 옴 - Prerecorded Videos  
Compressed Video

- Over 3 mbps
- Create Multiple Versions of the same video, with different quality level
- Which version they want to watch as a function of their current available bandwidth
  - 사용자의 대역폭 상태에 따라 화질 선택 가능

#### HTTP Streaming

Video is stored in HTTP Server as a file with URL  
User with TCP Connection and action GET request for that URL  
Server returns video file and HTTP Response

On client, Bytes are collected in client application buffer.  
bytes exceeds a predetermined threshold, app begins play-back.

### DASH

Dynamic Adaptive Streaming over HTTP.  
Encodes each video into several different versions. with different bit rate.

- Manifest File, providing a URL for each version with its bit rate.  
  전체 파일 정보의 버전을 담아놓음

#### Client

Request Manifest File including various versions  
different internet access rates at different encoding rates
dynamically chunk video segments of a few seconds in length.

Rate Determination Algorithm - 다음엔 어떤 길이로 동영상을 받아올까  
Clients adapt to the available bandwidth if the available end to end bandwidth changes

Reason not to provide streaming service in centralized way.
