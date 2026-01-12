# Unreal Engine PlayerController / PlayerState 정리

멀티플레이 환경에서 플레이어를 구성하는 핵심 클래스인  
**PlayerController**와 **PlayerState**의 역할과 책임을 정리한다.

---

## 1. PlayerController

### 개요
- **플레이어의 입력을 해석하고 게임 오브젝트에 명령을 전달하는 핵심 클래스**
- 키보드, 마우스, 게임 패드 등의 입력을 받아 처리
- 멀티플레이 환경에서는 **플레이어마다 개별 PlayerController가 생성**되어  
  여러 사용자의 입력을 충돌 없이 분리하여 관리한다

### 특징
- 서버와 클라이언트에 존재
- 각 PlayerController는 특정 플레이어에 1:1로 대응
- Pawn을 소유(Possess)하여 조작한다

---

### 입력 처리 흐름
1. 키보드, 마우스, 게임 패드 등 입력 장치로부터 사용자 조작 신호가 들어온다
2. PlayerController가 입력을 수신하고 해석한다
3. PlayerController가 현재 소유(Possess) 중인 Pawn에게  
   이동, 회전, 공격 등의 구체적인 명령을 전달한다

---

### 주요 기능
- 입력 처리
- 카메라 제어 로직
- HUD 및 UI와의 상호작용
- Pawn Possess / UnPossess 관리

---

## 2. PlayerState

### 개요
- **각 플레이어의 상태 정보를 저장하는 클래스**
- 서버에서 생성되며 모든 클라이언트에 **Replication**되어 공유된다
- 플레이어가 재스폰하거나 Pawn이 변경되어도 유지된다

### 역할 분담
- **PlayerController** : 플레이어의 *행동* (입력, 시점, 조작)
- **PlayerState** : 플레이어의 *정보* (상태, 점수, 메타데이터)

---

### 특징
- 서버 Authority
- 모든 클라이언트에 복제되어 조회 가능
- Pawn과 분리된 순수 데이터 컨테이너 역할

---

### 주로 관리하는 데이터
- 닉네임 및 유저 정보
- 점수
- 킬 / 데스 수
- 커스텀 외형 정보 (치장 아이템 등)

---

## 3. PlayerController vs PlayerState 요약

| 구분 | PlayerController | PlayerState |
|----|-----------------|------------|
| 주요 역할 | 입력 및 조작 | 플레이어 상태 정보 |
| 네트워크 | 서버 / 소유 클라이언트 | 서버 → 모든 클라이언트 복제 |
| 유지 여부 | Pawn 변경 시 유지 | Pawn 변경 / 리스폰 시 유지 |
| 성격 | 행동 중심 | 데이터 중심 |

---

## 4. 언제 무엇을 써야 하나?

- **입력 처리, 카메라, UI 제어** → `PlayerController`
- **점수, 닉네임, 킬/데스 등 공유 정보** → `PlayerState`
