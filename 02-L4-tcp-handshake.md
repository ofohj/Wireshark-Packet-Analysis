# 주제: TCP 통신 연결/종료 과정 확인하기

## 🌐 학습목표
- 연결(3-way handshake), 데이터 전송과 종료(4-way handshake) 과정을 와이어샤크 패킷 상에서 어떻게 구현되는지 확인하기
- 해당 실습을 통해 실제 환경에서 세션 끊김, 재전송, 강제종료 등 문제가 발생했을 때 패킷단위로 문제를 추적하고 원인을 분석할 수 있음

---
## 🌐 실습과정
## 1. HTTP 사이트 접속
- HTTP 전용 사이트인 `http://neverssl.com`에 접속
- HTTPS 사이트에서 실습하지 않은 이유: 보안상의 이유로 HTTPS에서 발생하는 간섭 없이 순수한 TCP 패킷을 수집하기 위함

<img width="1470" height="985" alt="스크린샷 2026-09-19 214824" src="https://github.com/user-attachments/assets/c3401d27-b9d2-41f3-bec6-22919fb1999f" />

---

## 2. 와이어샤크 필터 설정하기
- 55718포트의 TCP의 동작을 보고싶기 때문에 필터에 `tcp.port == 55718` 입력하기
- 결과는 아래와 같다. 이 중 14255~15750 부분을 분석해보겠다.
  
  <img width="936" height="211" alt="image" src="https://github.com/user-attachments/assets/75ef0d5c-844b-4e20-88e0-b64e87d7eff6" />

💡 **패킷 번호 옆 대괄호 및 화살표 기호가 표시된 이유?**
- 와이어샤크가 현재 클릭된 패킷이 속한 단일 TCP 세션의 시작~진행~종료 범위를 시각화해주는 기능이다.
- 다른 패킷을 선택하면 그에 맞는 범위 표시가 활성화됨!

---

## 3. TCP 연결 과정 확인하기(3-way-handshake)

<img width="936" height="211" alt="image" src="https://github.com/user-attachments/assets/222cac91-dc85-4f7e-a33d-fdc57a0ba7b3" />

### 1) 14255번 패킷: (클라이언트 → 서버) SYN 세그먼트 전송

  <img width="567" height="222" alt="image" src="https://github.com/user-attachments/assets/94dfd13e-1db5-4f42-b734-b9cfdd0aa778" />

  🦈 **[확인해야할 부분]**
  - 클라이언트(Source Port: 55178)에서 출발해 서버(Destination Port: 80(http 포트번호))로 패킷 전달
  - Sequence Number: 152513721로 지정

### 2) 14264번 패킷: (서버 → 클라이언트) SYN, ACK로 응답

  <img width="658" height="223" alt="image" src="https://github.com/user-attachments/assets/c3c24305-7340-4223-bf61-2b2ea23aff27" />

  🦈 **[확인해야할 부분]**
  - 방금 전 SYN과는 반대로, 서버(Source Port: 80)에서 출발해 클라이언트(Destination Port: 55178)로 패킷 전달
  - Sequence Number: 182212534로 지정
  - Acknowledge Number: SYN의 Sequence number인 152513721의 다음 숫자 152513722로 지정됨

### 3) 14265번 패킷: (클라이언트 → 서버) ACK로 응답

  <img width="637" height="225" alt="image" src="https://github.com/user-attachments/assets/7033f58e-00fc-42a4-84b7-b89e4f4e7794" />
  
  🦈 **[확인해야할 부분]**
  - 다시 클라이언트(Source Port: 55178)에서 출발해 서버(Destination Port: 80)로 패킷 전달
  - Sequence Number: 서버의 Ack number(152513722)로 지정됨
  - Acknowledge Number: SYN의 Sequence number인 182212534의 다음 숫자 182212535로 지정됨

---

## 4. 데이터 송수신 과정 확인하기

<img width="930" height="208" alt="image" src="https://github.com/user-attachments/assets/764f1864-4cee-4106-a7f8-1768d48b19de" />

- 14266번 패킷: HTTP(웹 페이지 데이터 요청)
- 14268번 패킷: ACK(요청에 대한 서버의 응답 수신)
- 14269번 패킷: 웹 서버의 페이지 데이터 응답 성공(200)
- 14300번 패킷: ACK(데이터 잘 받았음을 서버에 알림)

---

## 5. TCP 종료 과정 확인하기(4-way-handshake)

<img width="930" height="212" alt="image" src="https://github.com/user-attachments/assets/e5073f53-7d93-451b-8ec7-f9e7a66f0319" />

### 1) 15492번 패킷: (클라이언트 → 서버) FIN, ACK로 종료 요청
- 이후 확인해야할 부분은 3번(3-way-handshake)와 유사하므로 하이라이트로만 표시

  <img width="837" height="419" alt="image" src="https://github.com/user-attachments/assets/98bb1527-590d-456c-914f-4ece9c510ae6" />

### 2) 15520번 패킷: (서버 → 클라이언트) ACK로 응답

  <img width="669" height="428" alt="image" src="https://github.com/user-attachments/assets/5f2b8e0a-d221-49f3-86b7-c4f287441c72" />
  
### 3) 15749번 패킷: (서버 → 클라이언트) FIN, ACK로 응답

  <img width="644" height="410" alt="image" src="https://github.com/user-attachments/assets/ce6c3878-3cc9-46d9-ba7d-0291c15efbf0" />

### 4) 15750번 패킷: (클라이언트 → 서버) ACK로 종료

  <img width="656" height="421" alt="image" src="https://github.com/user-attachments/assets/cf14f7f4-3dcb-4489-8b5c-a4aad9d5e27a" />

---
## 🌐 추가학습

💡 **이론상 4-way-handshake의 과정과 실제 패킷에 찍힌 과정 차이**
- 이론상 4-way-handshake는 다음과 같다.

  (1) 클라이언트 → 서버: FIN
  
  (2) 서버 → 클라이언트: ACK
  
  (3) 서버 → 클라이언트: FIN
  
  (4) 클라이언트 → 서버: ACK

- 그런데 실제 패킷엔 다음과 같이 찍혀있었다.

  (1) 서버 → 클라이언트: FIN, ACK
  
  (2) 클라이언트 → 서버: ACK

- **이유1: 자동 종료**
  - Keep-Alive Timeout(5초) 작동으로, 더이상 요청메시지가 들어오지 않아 서버가 먼저 연결 종료 선언
 
- **이유2: Piggybacking**
  - 네트워크 자원을 아끼고 전송 효율을 극대화하기 위해서 FIN과 ACK 응답 메시지를 동시에 보내는 기능
  - 패킷을 같이 보냄으로써, 패킷을 보낼 때 붙는 헤더의 중복 생성을 줄여 대역폭 낭비를 방지
 
