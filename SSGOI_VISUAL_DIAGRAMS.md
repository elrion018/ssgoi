# SSGOI 아키텍처 시각화 다이어그램

## 📊 전체 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Application Layer                            │
│  ┌─────────────────────┐        ┌─────────────────────┐            │
│  │   React/Next.js     │        │    Router (App)     │            │
│  │   Svelte/SvelteKit  │◄───────┤   - /home          │            │
│  │   Vue/Nuxt          │        │   - /item/[id]     │            │
│  └─────────────────────┘        └─────────────────────┘            │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      Framework Layer (@ssgoi/react)                  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  <Ssgoi config={ssgoiConfig}>                                │  │
│  │    ┌────────────────────────────────────────────────┐       │  │
│  │    │  <SsgoiTransition id="/">                     │       │  │
│  │    │    ┌──────────────────────────────────┐       │       │  │
│  │    │    │  Page Content                    │       │       │  │
│  │    │    │  - Hero Elements                 │       │       │  │
│  │    │    │  - DOM Transitions               │       │       │  │
│  │    │    └──────────────────────────────────┘       │       │  │
│  │    └────────────────────────────────────────────────┘       │  │
│  │  </Ssgoi>                                                    │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Components:                                                         │
│  • Ssgoi Provider ──────► createSggoiTransitionContext()           │
│  • SsgoiTransition ─────► transition() hook                        │
│  • React Context ───────► useSsgoi()                               │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        Core Layer (@ssgoi/core)                      │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │               TransitionContext (페이지 전환 조정)              │ │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐  │ │
│  │  │ PendingTransition│  │  Route Matching │  │ Scroll Offset│  │ │
│  │  │  - from: "/a"    │  │  - exact match  │  │  - x: 0      │  │ │
│  │  │  - to: "/b"      │◄─┤  - wildcard /*  │  │  - y: 120    │  │ │
│  │  │  - outResolve()  │  │  - universal *  │  └──────────────┘  │ │
│  │  │  - inResolve()   │  └─────────────────┘                    │ │
│  │  └─────────────────┘                                          │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │            TransitionCallback (애니메이션 상태 머신)            │ │
│  │                                                                │ │
│  │   Current State    Trigger    Action                          │ │
│  │   ─────────────    ───────    ──────                          │ │
│  │   No Animation  +  IN     →   Start IN (0→1)                  │ │
│  │   No Animation  +  OUT    →   Clone & Start OUT (1→0)         │ │
│  │   IN Running    +  OUT    →   Reverse IN                      │ │
│  │   OUT Running   +  IN     →   Reverse OUT                     │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                   Animator (물리 엔진)                         │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   │ │
│  │  │ Spring Config│  │ State Track  │  │ Velocity Preserve│   │ │
│  │  │ - stiffness  │  │ - position   │  │ - normalized     │   │ │
│  │  │ - damping    │  │ - velocity   │  │ - reversible     │   │ │
│  │  │ - mass       │  │ - from/to    │  │ - continuous     │   │ │
│  │  └──────────────┘  └──────────────┘  └──────────────────┘   │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    Transition Types                            │ │
│  │  ┌─────────────────────┐      ┌─────────────────────┐        │ │
│  │  │  View Transitions   │      │ Element Transitions │        │ │
│  │  │  • hero()           │      │  • fadeIn/Out()     │        │ │
│  │  │  • scroll()         │      │  • slideUp/Down()   │        │ │
│  │  │  • drill()          │      │  • scaleIn/Out()    │        │ │
│  │  │  • fade()           │      │  • bounce()         │        │ │
│  │  │  • pinterest()      │      │  • rotate()         │        │ │
│  │  └─────────────────────┘      └─────────────────────┘        │ │
│  └───────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     Animation Engine (Popmotion)                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  animate({                                                     │ │
│  │    from: 0,                                                    │ │
│  │    to: 1,                                                      │ │
│  │    onUpdate: (progress) => element.style.opacity = progress,   │ │
│  │    ...springConfig                                             │ │
│  │  })                                                            │ │
│  └───────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔄 페이지 전환 시퀀스 플로우

```
사용자 클릭 → 라우터 네비게이션
    │
    ├─────────────────────────────────────────┐
    ▼                                         ▼
Page A (Departing)                    Page B (Arriving)
    │                                         │
    │ 1. Unmount 트리거                       │ 1. Mount 트리거
    │                                         │
    ▼                                         ▼
TransitionContext에 등록              TransitionContext에 등록
{ from: "/pageA" }                    { to: "/pageB" }
    │                                         │
    └──────────────┬──────────────────────────┘
                   ▼
            checkAndResolve()
            경로 매칭 & 설정 찾기
                   │
    ┌──────────────┴──────────────────────────┐
    ▼                                         ▼
OUT Animation 시작                      IN Animation 시작
(1 → 0 페이드 아웃)                     (0 → 1 페이드 인)
    │                                         │
    │  ┌──────────────────────────────────┐   │
    │  │     동시 실행 (Parallel)           │   │
    │  │  • tick(progress) 매 프레임        │   │
    │  │  • Spring physics 적용            │   │
    │  │  • 60fps 목표                     │   │
    │  └──────────────────────────────────┘   │
    │                                         │
    ▼                                         ▼
완료 & 정리                             완료
(클론 요소 제거)                        (최종 상태)
```

## 🎯 Hero Transition 데이터 플로우

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
                    HERO TRANSITION FLOW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Page A (/home)                          Page B (/item/123)
┌────────────────────┐                  ┌────────────────────┐
│  ┌──────────────┐  │                  │  ┌──────────────┐  │
│  │ Hero Element │  │                  │  │ Hero Element │  │
│  │ data-hero-   │  │   ──────────►    │  │ data-hero-   │  │
│  │ key="box-1"  │  │    Transform     │  │ key="box-1"  │  │
│  └──────────────┘  │    Animation     │  └──────────────┘  │
│                    │                  │                    │
│  Position:         │                  │  Position:         │
│  {x: 100, y: 200}  │                  │  {x: 0, y: 0}      │
│  {w: 100, h: 100}  │                  │  {w: 400, h: 400}  │
└────────────────────┘                  └────────────────────┘
        │                                        │
        ▼                                        ▼
┌───────────────────────────────────────────────────────────┐
│                   TransitionContext                       │
│                                                           │
│  1. OUT Phase: Store source positions                     │
│     fromNode = { rect, element, key: "box-1" }            │
│                                                           │
│  2. IN Phase: Find matching elements                      │
│     toNode = findByKey("box-1")                           │
│                                                           │
│  3. Calculate Transform:                                  │
│     dx = 100 - 0 = 100                                    │
│     dy = 200 - 0 = 200                                    │
│     dw = 100 / 400 = 0.25                                 │
│     dh = 100 / 400 = 0.25                                 │
│                                                           │
│  4. Apply Animation (progress: 0 → 1):                    │
│     translate: (1-p)*100, (1-p)*200                       │
│     scale: p + (1-p)*0.25                                 │
└───────────────────────────────────────────────────────────┘
```

## 🎭 애니메이션 상태 머신

```
┌─────────────────────────────────────────────────────────────┐
│                  ANIMATION STATE MACHINE                    │
└─────────────────────────────────────────────────────────────┘

                        ┌──────────────┐
                        │ No Animation │
                        │   (Idle)     │
                        └──────┬───────┘
                               │
                ┌──────────────┼──────────────┐
                ▼                             ▼
         ╔═══════════╗                 ╔═══════════╗
         ║ IN Trigger║                 ║OUT Trigger║
         ╚═══════════╝                 ╚═══════════╝
                │                             │
                ▼                             ▼
        ┌──────────────┐               ┌──────────────┐
        │ IN Running   │               │ OUT Running  │
        │   (0 → 1)    │               │   (1 → 0)    │
        │              │               │ +Clone Element│
        └──────┬───────┘               └──────┬───────┘
               │                              │
               │         ┌────────┐           │
               │◄────────┤Reverse │────────── ┤
               │         └────────┘           │
               │                              │
         ╔═══════════╗                 ╔═══════════╗
         ║OUT Trigger║                 ║ IN Trigger║
         ╚═══════════╝                 ╚═══════════╝
               │                              │
               ▼                              ▼
        ┌──────────────┐               ┌──────────────┐
        │ Reverse IN   │               │ Reverse OUT  │
        │ Keep IN cfg  │               │ Keep OUT cfg │
        │ Backward     │               │ Backward     │
        └──────┬───────┘               └──────┬───────┘
               │                              │
               └──────────┬───────────────────┘
                          ▼
                   ┌──────────────┐
                   │  Complete &   │
                   │   Cleanup     │
                   └──────────────┘
```

## 📈 스프링 물리 계산 시각화

```
Spring Animation Progress
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1.0 │     ╭─────────────────── Target
    │    ╱
    │   ╱ ╱╲    ← Overshoot (low damping)
    │  ╱ ╱  ╲
0.8 │ ╱ ╱    ╲___
    │╱ ╱         ╲___
    │ ╱              ────────── Settle
0.6 │╱
    │
    │   Stiffness: 300 (Spring force)
0.4 │   Damping: 30 (Energy loss)
    │   Mass: 1 (Fixed)
    │
0.2 │   Velocity preservation on interrupt:
    │   v = (current - previous) / Δt / 1000
    │
0.0 └────────────────────────────────────► Time
    0ms  100ms  200ms  300ms  400ms  500ms

Configuration Examples:
─────────────────────────────────
Smooth:  { stiffness: 100, damping: 20 }  ← Gentle, flowing
Normal:  { stiffness: 300, damping: 30 }  ← Balanced
Fast:    { stiffness: 500, damping: 35 }  ← Snappy, quick
Bouncy:  { stiffness: 400, damping: 10 }  ← Elastic, playful
```

## 🔗 컴포넌트 상호작용 맵

```
┌────────────────────────────────────────────────────────────┐
│                    COMPONENT INTERACTION MAP                │
└────────────────────────────────────────────────────────────┘

User Action
    │
    ▼
┌─────────┐     Navigation      ┌──────────────┐
│  Link   │────────────────────►│    Router    │
└─────────┘                     └──────┬───────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    ▼                                     ▼
            ┌──────────────┐                    ┌──────────────┐
            │   Page A     │                    │   Page B     │
            │ (Unmounting) │                    │ (Mounting)   │
            └──────┬───────┘                    └──────┬───────┘
                   │                                    │
                   ▼                                    ▼
         ┌──────────────────┐                ┌──────────────────┐
         │ SsgoiTransition  │                │ SsgoiTransition  │
         │     id="/"       │                │  id="/item/123"  │
         └──────┬───────────┘                └──────┬───────────┘
                │                                    │
                ▼                                    ▼
         ┌──────────────────┐                ┌──────────────────┐
         │ transition()     │                │ transition()     │
         │     Hook         │                │     Hook         │
         └──────┬───────────┘                └──────┬───────────┘
                │                                    │
                └────────────┬───────────────────────┘
                            ▼
                 ┌──────────────────────┐
                 │ TransitionCallback   │
                 │  • State machine     │
                 │  • Animation control │
                 └──────────┬───────────┘
                            │
                ┌───────────┼───────────┐
                ▼                       ▼
        ┌──────────────┐       ┌──────────────┐
        │   Animator   │       │   Registry   │
        │ • Popmotion  │       │ • Cleanup    │
        │ • Spring     │       │ • Lifecycle  │
        └──────┬───────┘       └──────────────┘
                │
                ▼
        ┌──────────────┐       ┌──────────────┐
        │ DOM Original │       │ DOM Clone    │
        │   Element    │       │  (for exit)  │
        └──────────────┘       └──────────────┘
```

## 📊 데이터 플로우 매트릭스

```
┌──────────────────┬─────────────────┬──────────────────┬─────────────────┐
│   Component      │     INPUT       │    PROCESS       │     OUTPUT      │
├──────────────────┼─────────────────┼──────────────────┼─────────────────┤
│ Ssgoi Provider   │ • config        │ • Create context │ • Context value │
│                  │ • transitions   │ • Setup routes   │ • getTransition │
├──────────────────┼─────────────────┼──────────────────┼─────────────────┤
│ SsgoiTransition  │ • id (route)    │ • Get transition │ • DOM with ref  │
│                  │ • children      │ • Apply ref      │ • data-ssgoi-*  │
├──────────────────┼─────────────────┼──────────────────┼─────────────────┤
│ TransitionContext│ • from path     │ • Match routes   │ • transition    │
│                  │ • to path       │ • Find config    │   config        │
│                  │ • scroll offset │ • Resolve promise│ • animation def │
├──────────────────┼─────────────────┼──────────────────┼─────────────────┤
│ TransitionCallback│• IN/OUT trigger│ • State machine  │ • Start/Stop    │
│                  │ • Element ref   │ • Clone if needed│ • Reverse       │
│                  │ • Config        │ • Track state    │ • Cleanup       │
├──────────────────┼─────────────────┼──────────────────┼─────────────────┤
│ Animator         │ • Spring config │ • Physics calc   │ • Progress 0-1  │
│                  │ • from/to       │ • Velocity track │ • tick callback │
│                  │ • velocity      │ • Interpolate    │ • onComplete    │
├──────────────────┼─────────────────┼──────────────────┼─────────────────┤
│ Registry         │ • transition key│ • Store/Retrieve │ • Definition    │
│                  │ • definition    │ • Lifecycle mgmt │ • Auto cleanup  │
│                  │ • target object │ • Memory track   │ • GC handling   │
└──────────────────┴─────────────────┴──────────────────┴─────────────────┘
```

## 🚀 성능 최적화 포인트

```
Performance Optimization Points
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Animation Frame Optimization
   ┌────────────────────────────────────────┐
   │ requestAnimationFrame Loop             │
   │                                        │
   │ 60 FPS Target ──► 16.67ms per frame   │
   │                                        │
   │ • Batch DOM updates                   │
   │ • Use transform over position         │
   │ • GPU acceleration (will-change)      │
   └────────────────────────────────────────┘

2. Memory Management
   ┌────────────────────────────────────────┐
   │ FinalizationRegistry                   │s
   │                                        │
   │ Transition ──► Register ──► Auto GC   │
   │                                        │
   │ • Automatic cleanup                   │
   │ • No memory leaks                     │
   │ • Weak references                     │
   └────────────────────────────────────────┘

3. DOM Optimization
   ┌────────────────────────────────────────┐
   │ Minimal DOM Operations                 │
   │                                        │
   │ • Clone only on exit                  │
   │ • Batch style changes                 │
   │ • Use CSS transforms                  │
   │ • Avoid reflow/repaint                │
   └────────────────────────────────────────┘

4. Bundle Size
   ┌────────────────────────────────────────┐
   │ Tree Shaking                           │
   │                                        │
   │ Import only what you need:            │
   │ • import { hero } from '@ssgoi/core'  │
   │ • Not: import * as ssgoi              │
   └────────────────────────────────────────┘
```

## 📝 요약

이 시각화를 통해 SSGOI의 핵심 동작 원리를 한눈에 파악할 수 있습니다:

1. **계층 구조**: Application → Framework → Core → Animation Engine
2. **데이터 흐름**: 사용자 액션 → 라우터 → 페이지 전환 → 애니메이션
3. **상태 관리**: 4가지 애니메이션 상태와 자연스러운 전환
4. **최적화**: 성능, 메모리, DOM 조작 최적화 포인트

각 다이어그램은 ASCII 아트로 구성되어 마크다운 환경에서 바로 확인 가능합니다.