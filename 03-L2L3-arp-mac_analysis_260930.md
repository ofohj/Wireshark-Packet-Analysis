#  주제: ARP Request로 MAC 주소 식별하기

## 🌐 학습목표
- L3 계층의 IP 주소를 바탕으로 L2 계층의 MAC 주소를 알아내는 ARP 동작 방식을 확인한다.
- ARP 요청/응답에 따른 패킷 내용을 확인한다.

---

## 🌐 실습 순서
### 1. 와이어샤크 필터 설정하기
- arp 트래픽만 볼 수 있도록 필터창에 `arp` 입력하기

---

### 2. (관리자권한 CMD) 게이트웨이 IP 확인하기
- CMD 창에 `ipconfig` 입력하여 확인

---

### 3. (관리자권한 CMD) ARP 캐시 테이블 초기화
- `arp -d *` 명령어로 테이블 전체 삭제
- 이후, `arp -a` 명령어로 목록이 비었는지 확인

---

### 4. (관리자권한 CMD) ARP 트래픽 유발
- 게이트웨이 IP로 핑 보내기: `ping [게이트웨이 IP]`

---

### 5. (와이어샤크) ARP 요청/응답 패킷 확인하기

<img width="612" height="77" alt="스크린샷 2026-10-01 001137" src="https://github.com/user-attachments/assets/3048e75a-6efa-43b6-8f2a-dfa7ab77baf6" />

---

### 6. 분석결과
#### (1) 2번째 줄(ARP Request)
  - Who has `192.168.0.15`? Tell `192.168.0.15` = `192.168.0.15` IP 쓰는 사람 누구야? `192.168.0.15`(나 = PC)한테 mac 주소 좀 알려줘.
  - 패킷을 클릭해서 확인해보면, Broadcast로 보내졌음을 알 수 있음

    <img width="321" height="36" alt="image" src="https://github.com/user-attachments/assets/8ab7ae51-4531-49a7-9418-c6e3a2acced8" />

#### (2) 3번째 줄(ARP Reply)
  - 192.168.0.1 is at xx:xx:xx:xx:xx:xx = `192.168.0.15`의 mac 주소는 xx:xx:xx:xx:xx:xx이야.
  - 패킷을 클릭해서 확인해보면, Unicast 보내졌음을 알 수 있음
  - 이번 패킷은 Broadcast와 다르게 Destination: Unicast가 아닌 HonHaiPrecis가 떴는데, Unicast의 경우 해당 랜카드의 제조사명이 뜨기 때문.
  
    <img width="528" height="69" alt="image" src="https://github.com/user-attachments/assets/7aa07561-3827-41a3-8cf4-c87423d4d935" />

#### (3) 결과
  - ARP 응답을 받는 순간 내 PC의 ARP 캐시 테이블에 192.168.0.15가 자동으로 기록
  - 이후 동일한 목적지로 발송되는 IP 패킷은 추가적인 ARP Request 없이 메모리의 캐시 테이블을 참조하여 L2 프레임을 즉시 생성할 수 있음

### 💡 가벼운 트러블슈팅

<img width="580" height="256" alt="image" src="https://github.com/user-attachments/assets/f8f1c694-da06-4350-b8eb-ea8ffc434573" />

- arp만 필터링해서 보면 이렇게 요청이 엄청많이 쏟아지는 걸 확인할 수 있었다.
- 순간 이게 바로 ARP 스푸핑인가. 했는데 그게 아니라 >> arp 캐시를 삭제한 후, PC에 있는 다른 프로그램들이 arp 요청을 보내서 저런 현상이 발생한 것.
- 오류가 아니라 정상적인 현상으로 판정!
