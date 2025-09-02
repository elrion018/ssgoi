# SSGOI 애니메이션 상태 머신 - 시나리오 기반 설명

## 🎯 핵심 개념

SSGOI의 애니메이션 상태 머신은 **4가지 시나리오**를 처리합니다:

1. **첫 등장** - 요소가 처음 나타날 때
2. **퇴장** - 요소가 사라질 때  
3. **들어오다가 나가기** - 애니메이션 중 반대 동작
4. **나가다가 들어오기** - 애니메이션 중 반대 동작

## 📖 시나리오 1: 첫 등장 (Initial Mount)

### 상황
홈페이지에서 상품 상세 페이지로 이동할 때, 상품 카드가 처음 나타나는 경우

### 실제 동작
```
현재 상태: 애니메이션 없음 (No Animation)
트리거: IN (페이지 마운트)
동작: 0 → 1 애니메이션 시작

예시:
1. 사용자가 "/products/123" 페이지 진입
2. 상품 카드 opacity: 0 (안 보임)
3. IN 애니메이션 시작
4. opacity: 0 → 0.2 → 0.5 → 0.8 → 1.0
5. 완전히 보이는 상태로 완료
```

### 코드에서 일어나는 일
```typescript
// create-transition-callback.ts
if (!currentAnimation && trigger === 'IN') {
  // 새로운 IN 애니메이션 생성
  currentAnimation = new Animator({
    from: 0,
    to: 1,
    onUpdate: (progress) => {
      element.style.opacity = progress;
    }
  });
  currentAnimation.forward(); // 0에서 1로 진행
}
```

## 📖 시나리오 2: 퇴장 (Unmount with Clone)

### 상황
상품 상세 페이지에서 홈으로 돌아갈 때, 현재 페이지가 사라지는 경우

### 실제 동작
```
현재 상태: 애니메이션 없음
트리거: OUT (페이지 언마운트)
동작: 요소 복제 후 1 → 0 애니메이션

예시:
1. 사용자가 "뒤로가기" 클릭
2. React가 컴포넌트를 제거하려 함
3. SSGOI가 DOM 요소를 복제 (클론 생성)
4. 원본은 사라지고, 클론이 남아서 애니메이션
5. opacity: 1.0 → 0.8 → 0.5 → 0.2 → 0
6. 애니메이션 완료 후 클론도 제거
```

### 왜 복제가 필요한가?
```typescript
// React가 컴포넌트를 제거하면 DOM도 사라짐
// 하지만 우리는 퇴장 애니메이션을 보여주고 싶음
// 해결책: DOM 복제본을 만들어 애니메이션 실행

if (!currentAnimation && trigger === 'OUT') {
  const clone = element.cloneNode(true);
  const parent = element.parentNode;
  
  // 원본 자리에 클론 삽입
  parent.insertBefore(clone, element.nextSibling);
  
  // 클론으로 OUT 애니메이션
  currentAnimation = new Animator({
    from: 1,
    to: 0,
    onUpdate: (progress) => {
      clone.style.opacity = progress;
    },
    onComplete: () => {
      clone.remove(); // 애니메이션 끝나면 클론 제거
    }
  });
}
```

## 📖 시나리오 3: 들어오다가 나가기 (IN → OUT Reversal)

### 상황
모달이 열리는 중(50% 진행)에 사용자가 ESC를 눌러 닫는 경우

### 실제 동작
```
현재 상태: IN 애니메이션 진행 중 (0.5 지점)
트리거: OUT
동작: IN 애니메이션을 역방향으로 (0.5 → 0)

예시:
1. 모달 열기 시작 (opacity: 0 → 0.5 진행 중)
2. 사용자가 ESC 키 누름
3. 새로운 OUT 애니메이션을 만들지 않음!
4. 현재 IN 애니메이션을 그대로 역전
5. opacity: 0.5 → 0.3 → 0.1 → 0
6. 자연스럽게 닫힘
```

### 핵심: 왜 역전인가?
```typescript
// ❌ 나쁜 방법: 새 애니메이션 생성
if (inAnimation && trigger === 'OUT') {
  inAnimation.stop();
  outAnimation = new Animator({
    from: 0.5, // 현재 위치
    to: 0,
    spring: outConfig // 다른 설정 사용
  });
  // 문제: 갑자기 속도가 바뀜, 부자연스러움
}

// ✅ 좋은 방법: 기존 애니메이션 역전
if (inAnimation && trigger === 'OUT') {
  // 같은 설정, 같은 속도 유지
  // 단지 방향만 반대로
  inAnimation.reverse(); 
  // 속도가 유지되어 자연스러움
}
```

### 시각적 비교
```
나쁜 방법 (끊김):
IN:  ●━━━━━━→ (빠른 속도)
           ↓ (갑자기 정지)
OUT:       ←━━━━━● (다른 속도)

좋은 방법 (자연스러움):
IN:  ●━━━━━━→
           ↓ (부드럽게 방향 전환)
OUT:      ←━━━━━━●  (같은 속도 유지)
```

## 📖 시나리오 4: 나가다가 들어오기 (OUT → IN Reversal)

### 상황
탭이 닫히는 중에 사용자가 마우스를 다시 올려 탭을 여는 경우

### 실제 동작
```
현재 상태: OUT 애니메이션 진행 중 (0.3 지점)
트리거: IN
동작: OUT 애니메이션을 역방향으로 (0.3 → 1)

예시:
1. 마우스가 벗어나 탭 닫기 시작 (opacity: 1 → 0.3)
2. 사용자가 다시 마우스 올림
3. OUT 애니메이션을 역전
4. opacity: 0.3 → 0.5 → 0.8 → 1.0
5. 다시 완전히 열림
```

### 속도 보존의 중요성
```typescript
// 애니메이션 역전 시 속도 보존
const velocity = animator.getVelocity(); // 현재 속도
animator.reverse(); // 방향 반대로

// 예: 빠르게 닫히던 중 다시 열기
// 닫히는 속도: -500px/s
// 역전 후 열리는 속도: +500px/s (같은 크기, 반대 방향)
```

## 🎮 실제 사용자 경험 예시

### 예시 1: 드롭다운 메뉴
```
시간축: 0ms ─────────────────────────────────────► 1000ms

사용자 행동:  클릭(열기)     다시클릭(닫기)    호버(열기)
             ↓              ↓                ↓
상태:        [없음] ───→ [IN 실행] ───→ [IN 역전] ───→ [OUT 역전]
             
애니메이션:   0 ────→ 0.7 ────→ 0.3 ────→ 1.0
             (열리기)   (닫히기)    (다시 열기)
```

### 예시 2: 페이지 전환
```
페이지 A → 페이지 B (중간에 뒤로가기)

1. 페이지 A에서 링크 클릭
   - 페이지 A: OUT 시작 (fade out)
   - 페이지 B: IN 시작 (fade in)

2. 0.5초 지점에서 뒤로가기 버튼
   - 페이지 A: OUT 역전 → 다시 나타남
   - 페이지 B: IN 역전 → 사라짐

3. 결과: 페이지 A로 자연스럽게 복귀
```

## 🔑 핵심 설계 원칙

### 1. **상태 연속성**
```
애니메이션은 항상 현재 위치에서 시작
- 갑작스런 점프 없음
- 부드러운 전환 보장
```

### 2. **속도 보존**
```
방향이 바뀌어도 속도는 유지
- 물리적으로 자연스러움
- 사용자가 예측 가능
```

### 3. **설정 일관성**
```
IN/OUT 역전 시 원래 설정 유지
- IN 중 OUT: IN 설정으로 역전
- OUT 중 IN: OUT 설정으로 역전
- 일관된 애니메이션 느낌
```

## 💡 개발자를 위한 팁

### 언제 어떤 상태가 발생하나?

| 사용자 동작 | 현재 상태 | 트리거 | 결과 |
|------------|----------|--------|------|
| 페이지 진입 | 없음 | IN | 새 IN 애니메이션 |
| 페이지 떠남 | 없음 | OUT | 클론 + OUT 애니메이션 |
| 열다가 닫기 | IN 진행중 | OUT | IN 역전 |
| 닫다가 열기 | OUT 진행중 | IN | OUT 역전 |

### 디버깅 체크포인트

```typescript
// 상태 확인
console.log({
  hasAnimation: !!currentAnimation,
  isRunning: currentAnimation?.isRunning(),
  currentValue: currentAnimation?.getCurrentValue(),
  velocity: currentAnimation?.getVelocity(),
  direction: currentAnimation?.getDirection()
});

// 트리거 추적
console.log(`Trigger: ${trigger}, Current State: ${state}`);
```

## 🎬 결론

SSGOI의 애니메이션 상태 머신은 **사용자의 즉각적인 반응**에 대응하도록 설계되었습니다:

- ✅ **중단 가능**: 언제든 방향 전환 가능
- ✅ **자연스러움**: 속도와 가속도 보존
- ✅ **예측 가능**: 일관된 동작 패턴
- ✅ **효율적**: 불필요한 애니메이션 생성 없음

이 설계로 네이티브 앱처럼 부드럽고 반응적인 웹 경험을 제공합니다.