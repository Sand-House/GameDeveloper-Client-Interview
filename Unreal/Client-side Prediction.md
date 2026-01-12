# Client-side Prediction

## 1. 개요

Client-side Prediction(CSP)은 멀티플레이 환경에서 **입력 지연(latency)** 문제를 해결하기 위한 기법이다.  
서버 권위(Server Authoritative) 구조를 유지하면서도, 클라이언트가 서버 응답을 기다리지 않고  
**입력 결과를 미리 시뮬레이션**하여 즉각적인 조작감을 제공한다.

---

## 2. 왜 필요한가?

서버 권위 구조에서 입력 처리 흐름은 다음과 같다.

1. 클라이언트가 입력 전송
2. 서버가 입력 처리
3. 서버 결과를 클라이언트에 전송

이 방식만 사용하면 네트워크 지연만큼 입력 반응이 늦어지며,  
조작감이 매우 나빠진다.

Client-side Prediction은 이 문제를 해결하기 위해  
**클라이언트에서 입력을 즉시 반영**한다.

---

## 3. 기본 개념

- 서버는 항상 **최종 권위(Authority)** 를 가진다.
- 클라이언트의 이동은 **예측 결과(Predicted State)** 이다.
- 서버 결과와 다를 경우 **보정(Correction)** 이 발생한다.

> Client-side Prediction은 클라이언트 권한을 의미하지 않는다.

---

## 4. 동작 흐름

### 4.1 프레임 단위 흐름

[Client]
입력 발생
→ 로컬에서 즉시 이동 (Prediction)
→ 입력을 서버로 전송

[Server]
입력 수신
→ 서버 기준 이동 계산
→ 결과 상태를 클라이언트로 전송

[Client]
서버 상태 수신
→ 예측 결과와 비교
→ 불일치 시 보정

---

## 5. Correction (보정)

서버 결과가 클라이언트 예측과 다를 경우 발생한다.

### 5.1 나쁜 보정
- 서버 위치로 즉시 덮어쓰기
- 순간이동처럼 튐 발생

### 5.2 좋은 보정
- 일정 시간 동안 보간(Lerp)
- 플레이어에게 자연스럽게 보이도록 처리

---

## 6. Rewind & Replay

언리얼을 포함한 대부분의 CSP 구현은  
**Rewind & Replay** 방식을 사용한다.

### 동작 방식

1. 서버가 특정 시점의 상태를 승인
2. 클라이언트는 해당 시점으로 되돌아감(Rewind)
3. 그 이후 저장된 입력들을 다시 적용(Replay)

이를 통해:
- 입력 손실 방지
- 부드러운 보정 가능

---

## 7. 언리얼 엔진에서의 Client-side Prediction

### 7.1 CharacterMovementComponent

언리얼에서는 `CharacterMovementComponent`가  
Client-side Prediction을 내부적으로 구현한다.

포함 기능:
- 입력 패킷 저장 (`SavedMove`)
- 서버 승인 처리
- 위치 보정
- Rewind & Replay

따라서 `ACharacter`를 사용할 경우 별도 구현이 거의 필요 없다.

---

## 8. 핵심 요약

- Client-side Prediction은 **지연을 숨기기 위한 기법**
- 서버는 항상 최종 판단자
- 차이는 **Correction + Rewind & Replay**로 해결
- 언리얼에서는 `CharacterMovementComponent`가 이를 담당

> **Client-side Prediction은 조작감을 위해 클라이언트가 먼저 움직이되,  
정답은 항상 서버가 결정하는 구조이다.**
