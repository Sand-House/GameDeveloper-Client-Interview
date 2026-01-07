# Unreal Engine에서 Replication과 RPC의 차이


## 1. Replication이란?

### 정의

Replication은 **서버가 가진 Actor의 상태(State)를 클라이언트와 자동으로 동기화**하는 메커니즘이다.

* 서버가 Authority를 가짐
* 서버의 상태가 기준
* 클라이언트는 결과를 수신

```cpp
UPROPERTY(Replicated)
int32 HP;
```

---

### Replication의 특징

* **상태 기반(State-based)** 동기화
* 최신 값만 보장 (중간 과정은 중요하지 않음)
* Join In Progress(JIP)에 안전
* 패킷 유실이 발생해도 다음 업데이트로 복구 가능

---

### Replication을 써야 하는 대상

* HP, Ammo, Position
* 상태 플래그 (Alive, Dead, Stunned)
* 지속적으로 유지되어야 하는 값

> 핵심: **"지금 상태가 무엇인가"** 가 중요한 데이터

---

## 2. RPC란?

### 정의

RPC(Remote Procedure Call)는 **네트워크를 통해 원격에서 함수를 호출**하는 메커니즘이다.

```cpp
UFUNCTION(Server, Reliable)
void Server_Fire();
```

종류:

* Server RPC
* Client RPC
* Multicast RPC

---

### RPC의 특징

* **이벤트 기반(Event-based)**
* 호출 시점이 중요
* 과거 호출은 재전송되지 않음
* Join In Progress에 취약

---

### RPC를 써야 하는 대상

* 공격 요청
* 스킬 사용 요청
* 이펙트 / 사운드 재생
* UI 트리거

> 핵심: **"이 행동을 해달라"** 는 요청

---

## 3. Replication vs RPC 핵심 비교

| 구분     | Replication | RPC     |
| ------ | ----------- | ------- |
| 개념     | 상태 동기화      | 이벤트 전달  |
| 기준     | 현재 값        | 호출 순간   |
| JIP 대응 | 가능          | 불가능     |
| 패킷 유실  | 자동 복구       | 유실 시 소실 |
| 사용 대상  | 지속 상태       | 일회성 이벤트 |

---

## 4. 왜 HP는 Replication이고, 사운드는 RPC인가?

### HP

* 서버 authoritative 상태
* 현재 값이 중요
* JIP 대응 필요

👉 **Replication**

---

### 사운드 / 이펙트

* 일회성 연출
* 유실돼도 치명적이지 않음
* 늦게 들어온 클라이언트가 몰라도 됨

👉 **RPC (주로 Multicast)**

---

## 5. Reliable RPC를 써도 Replication이 필요한 이유

* Reliable RPC는 **이벤트 전달 보장**이지
  **현재 상태 보장**이 아님
* 과거 RPC는 JIP 클라이언트에게 전달되지 않음

> 상태는 항상 Replication이 정답

---

## 6. OnRep와 RPC의 관계

```cpp
UPROPERTY(ReplicatedUsing = OnRep_HP)
int32 HP;
```

* `OnRep`는 변수가 리플리케이트되었을 때 **클라이언트에서 호출되는 콜백**

---

## 7. Multicast RPC 사용 기준

Multicast는 다음 조건을 **모두 만족할 때만** 사용한다.

* 일회성 이벤트
* 늦게 들어온 클라이언트가 몰라도 되는 것
* 상태로 남지 않는 연출

### 예시

* 폭발 이펙트
* 총구 화염
* 환경 사운드

❌ HP 변경
❌ 위치 이동
❌ 상태 토글

---

## 8. 면접용 한 줄 요약

> "Replication은 상태 동기화이고,
> RPC는 이벤트 전달이다."

> "상태는 Replication,
> 연출과 요청은 RPC."
