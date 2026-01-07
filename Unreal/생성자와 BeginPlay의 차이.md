# Unreal Engine 생성자(Constructor) vs BeginPlay 정리

---

## 1. 핵심 한 줄 요약

| 구분          | 생성자 (Constructor) | BeginPlay()      |
| ----------- | ----------------- | ---------------- |
| 호출 시점       | 클래스 로드 / CDO 생성 시 | 인스턴스가 월드에 배치된 직후 |
| 목적          | **클래스 구조 정의**     | **런타임 로직 시작**    |
| World 접근    | ❌                 | ⭕                |
| 컴포넌트 생성     | ⭕ (필수)            | ❌                |
| 다른 Actor 참조 | ❌                 | ⭕                |

---

## 2. 생성자(Constructor)의 역할

### ✔ 생성자의 본질

* **클래스의 구조와 기본값을 정의하는 단계**
* 모든 인스턴스의 원형이 되는 **CDO(Class Default Object)** 생성 시에도 호출됨
* 월드, 레벨, 게임 진행 상태와 **무관**

---

### ✔ 생성자에서 해야 하는 작업

```cpp
MyActor::MyActor()
{
    RootComponent = CreateDefaultSubobject<USceneComponent>(TEXT("Root"));
    Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
}
```

* `CreateDefaultSubobject`
* 컴포넌트 생성 및 계층 구조 설정
* 기본 변수 초기화
* 에셋 **정적 로딩** (`ConstructorHelpers::FObjectFinder`)

---

### ❌ 생성자에서 하면 안 되는 작업

```cpp
GetWorld();                 // ❌ World 없음
GetWorld()->SpawnActor();   // ❌ 크래시 가능
```

❌ 이유:

* 생성자는 **CDO 생성 시에도 호출**됨
* 이 시점에는 World, Level, GameMode 존재하지 않음

📌 **PIE에서는 우연히 동작하고, 패키징 후 크래시 나는 대표 원인**

---

## 3. BeginPlay()의 역할

### ✔ BeginPlay의 본질

* Actor 인스턴스가 **실제 월드에 들어온 시점**
* 게임 로직이 시작되는 첫 지점

```cpp
void AMyActor::BeginPlay()
{
    Super::BeginPlay();

    GetWorld()->GetTimerManager().SetTimer(...);
}
```

---

### ✔ BeginPlay에서 해야 하는 작업

* `GetWorld()` 사용
* 다른 Actor 참조
* 타이머 등록
* 입력 처리
* AI / 상태 초기화
* 네트워크 관련 초기화

---

### ❌ BeginPlay에서 하면 안 되는 작업

```cpp
CreateDefaultSubobject<UStaticMeshComponent>(); // ❌
```

❌ 이유:

* 컴포넌트 구조가 CDO와 달라짐
* 에디터/저장/복제/리플리케이션 깨짐
* 클라이언트/서버 구조 불일치

👉 **컴포넌트 생성은 반드시 생성자에서만**

---

## 4. 자주 발생하는 실무 버그 예시

### ❌ 생성자에서 World 접근

* PIE에서는 정상
* 패키징 후 크래시

### ❌ BeginPlay에서 컴포넌트 생성

* 네트워크 복제 안 됨
* 저장/로드 불일치

---

## 5. 면접용 정리 답변

> 생성자는 클래스 구조와 기본값을 정의하는 단계이기 때문에
> 월드나 레벨 정보에 의존하는 작업을 하면 안 됩니다.
> 특히 생성자는 CDO 생성 시에도 호출되므로
> `GetWorld()` 같은 함수는 nullptr를 반환할 수 있습니다.
> 반면 BeginPlay는 인스턴스가 실제 월드에 배치된 이후 호출되므로
> 다른 Actor 참조, 타이머 등록, 입력 처리 같은
> 런타임 로직을 처리하기에 적절합니다.
> 따라서 컴포넌트 생성이나 클래스 구조 정의는 생성자에서,
> 게임 로직 초기화는 BeginPlay에서 수행해야 합니다.
