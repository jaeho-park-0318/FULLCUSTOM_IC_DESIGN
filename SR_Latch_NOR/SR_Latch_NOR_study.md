# NOR 기반 SR Latch 설계 보고서

## 1. 설계 개요

### 1.1 설계명

**NOR-Based SR Latch**

### 1.2 설계 목적

NOR 기반 SR Latch는 두 개의 NOR Gate를 Cross-Coupled 형태로 연결하여 1-bit 정보를 저장하는 기본적인 순차 논리 회로이다.

NAND 기반 SR Latch와 달리 NOR 기반 SR Latch에서는 Set과 Reset 입력이 **Active-High** 방식으로 동작한다.

즉 입력이 1이 되었을 때 Set 또는 Reset 동작이 수행된다.

본 설계의 목표는 다음과 같다.

- NOR Gate 기반 SR Latch의 동작 원리 이해
- Cross-Coupled Feedback 구조 이해
- Active-High Set / Reset 동작 이해
- Hold / Set / Reset / Invalid 상태 분석
- NAND SR Latch와의 차이 비교
- Cadence Virtuoso Transient Simulation
- Full-Custom Schematic / Layout 설계
- DRC / LVS / PEX 검증

---

# 2. 기본 입출력

NOR 기반 SR Latch는 다음 입력을 가진다.

```text
S = Set

R = Reset
```

출력:

```text
Q

QB
```

정상 상태에서는:

```text
QB = NOT(Q)
```

관계를 만족해야 한다.

---

# 3. Active-High 입력

NOR SR Latch의 가장 중요한 특징은 입력이 **Active-High**라는 것이다.

즉:

```text
S = 1 → Set

R = 1 → Reset
```

이다.

반면 입력이 모두 0이면:

```text
S = 0
R = 0
```

기존 상태를 유지한다.

---

# 4. NOR SR Latch 구조

두 개의 NOR Gate를 Cross-Coupled 방식으로 연결한다.

```text
                   ┌───────────┐
S ────────────────>│           │
                   │    NOR    ├────────────> QB
              Q ──>│           │
                   └───────────┘
                        ▲
                        │
                        │ Cross-Coupled
                        │ Feedback
                        ▼
                   ┌───────────┐
R ────────────────>│           │
                   │    NOR    ├────────────> Q
             QB ──>│           │
                   └───────────┘
```

주의할 점은 NOR SR Latch에서 Set 입력 `S=1`일 때 최종적으로 `Q=1`이 되도록 **Q/QB 출력 이름을 올바르게 지정해야 한다는 것**이다.

---

# 5. 논리식

NOR 기반 SR Latch는 다음 관계로 표현할 수 있다.

```text
QB = NOT(S OR Q)

Q  = NOT(R OR QB)
```

두 출력이 서로 Feedback 입력으로 연결되어 상태를 유지한다.

---

# 6. Truth Table

| S | R | Q(next) | QB(next) | 동작 |
|---:|---:|---:|---:|---|
| 0 | 0 | Q(previous) | QB(previous) | Hold |
| 1 | 0 | 1 | 0 | Set |
| 0 | 1 | 0 | 1 | Reset |
| 1 | 1 | 0 | 0 | Invalid |

---

# 7. Hold 상태

입력:

```text
S = 0
R = 0
```

인 경우이다.

NOR Gate의 외부 입력이 모두 비활성화되어 있으며 출력은 Feedback에 의해 이전 상태를 유지한다.

따라서:

```text
Q(next)  = Q(previous)

QB(next) = QB(previous)
```

이다.

---

# 8. Set 상태

입력:

```text
S = 1
R = 0
```

으로 설정한다.

S 입력이 1이 되면 Set 측 NOR 출력은:

```text
QB = 0
```

이 된다.

QB가 0이고 R도 0이므로 다른 NOR Gate 출력은:

```text
Q = NOT(0 OR 0)

Q = 1
```

이 된다.

따라서:

```text
Q  = 1
QB = 0
```

으로 Set된다.

---

# 9. Reset 상태

입력:

```text
S = 0
R = 1
```

로 설정한다.

R이 1이므로 Q를 발생시키는 NOR Gate 출력은:

```text
Q = 0
```

이 된다.

Q가 0이고 S도 0이면:

```text
QB = NOT(0 OR 0)

QB = 1
```

이다.

따라서:

```text
Q  = 0
QB = 1
```

이 된다.

---

# 10. Invalid 상태

입력:

```text
S = 1
R = 1
```

이면 두 NOR Gate 모두 입력에 1을 받는다.

NOR Gate는 하나의 입력이라도 1이면 출력이 0이므로:

```text
Q  = 0
QB = 0
```

이 된다.

그러나 정상 상태에서는:

```text
QB = NOT(Q)
```

여야 한다.

따라서:

```text
Q = QB = 0
```

은 정상적인 Latch 상태가 아니다.

즉:

```text
S = 1
R = 1
```

은 **Invalid State**이다.

---

# 11. Invalid 상태 해제 문제

다음 상태:

```text
S = 1
R = 1
```

에서:

```text
S = 0
R = 0
```

으로 동시에 변경하면 두 NOR Gate가 동시에 상태를 결정하려고 한다.

실제 CMOS 회로에서는 다음 요소의 차이가 존재한다.

```text
Transistor Delay
Threshold Voltage
Parasitic Capacitance
Routing Delay
Device Mismatch
```

따라서 최종 Q 값이 예측하기 어려워질 수 있다.

이 때문에 Invalid 입력은 사용하지 않는 것이 원칙이다.

---

# 12. NAND SR Latch와의 핵심 차이

NAND SR Latch:

```text
Active-Low

S_B = 0 → Set
R_B = 0 → Reset

S_B = R_B = 1 → Hold

S_B = R_B = 0 → Invalid
```

NOR SR Latch:

```text
Active-High

S = 1 → Set
R = 1 → Reset

S = R = 0 → Hold

S = R = 1 → Invalid
```

---

# 13. 권장 Port Mapping

## Input

```text
S
R
```

## Output

```text
Q
QB
```

## Power

```text
VDD
VSS
```

권장 Symbol:

```text
          ┌───────────────┐
S ───────>│               │──────> Q
R ───────>│ NOR_SR_LATCH  │
          │               │──────> QB
          └───────────────┘
```

---

# 14. CMOS NOR2 구조

Static CMOS 2-input NOR Gate는:

```text
PMOS → Series

NMOS → Parallel
```

구조를 사용한다.

```text
                    VDD
                     │
                  PMOS A
                     │
                  PMOS B
                     │
                    OUT
                     │
              ┌──────┴──────┐
              │             │
            NMOS A        NMOS B
              │             │
              └──────┬──────┘
                     │
                    VSS
```

---

# 15. NAND와 NOR의 CMOS 구조 차이

## NAND

```text
PMOS = Parallel

NMOS = Series
```

## NOR

```text
PMOS = Series

NMOS = Parallel
```

이 차이는 Full-Custom Layout과 Delay 특성에도 영향을 준다.

---

# 16. NOR SR Latch의 Transistor 수

Static CMOS NOR2 하나는:

```text
PMOS = 2

NMOS = 2
```

총:

```text
4 Transistors
```

를 사용한다.

NOR SR Latch는 NOR2 두 개이므로:

```text
4 × 2 = 8 Transistors
```

이다.

---

# 17. 권장 Cell Hierarchy

```text
PMOS / NMOS
     ↓
    NOR2
     ↓
 NOR_SR_LATCH
```

하위 NOR2 Cell에서 먼저:

```text
Schematic
Simulation
Layout
DRC
LVS
```

를 완료한 후 SR Latch를 구성하는 것이 좋다.

---

# 18. Cadence Virtuoso Schematic 설계 순서

```text
1. NOR2 Cell 생성

2. PMOS × 2 배치
3. NMOS × 2 배치

4. PMOS Series 연결
5. NMOS Parallel 연결

6. VDD / VSS 연결

7. NOR2 Functional Simulation

8. NOR2 Symbol 생성

9. NOR_SR_LATCH Cell 생성

10. NOR2 Symbol × 2 배치

11. Q Output과 QB Output 지정

12. Q → 반대쪽 NOR 입력으로 Feedback

13. QB → 반대쪽 NOR 입력으로 Feedback

14. S Input Pin 생성

15. R Input Pin 생성

16. Q / QB Output Pin 생성

17. VDD / VSS 연결

18. Check and Save

19. Symbol 생성

20. Testbench 생성

21. Transient Simulation
```

---

# 19. Simulation 검증

Transient Simulation에서 다음 순서로 확인할 수 있다.

## Initial Reset

```text
S = 0
R = 1
```

예상:

```text
Q  = 0
QB = 1
```

---

## Hold

```text
S = 0
R = 0
```

이전 상태 유지

---

## Set

```text
S = 1
R = 0
```

예상:

```text
Q  = 1
QB = 0
```

---

## Hold

```text
S = 0
R = 0
```

Set 상태 유지

---

## Reset

```text
S = 0
R = 1
```

예상:

```text
Q  = 0
QB = 1
```

---

# 20. 초기 상태 문제

Cross-Coupled Feedback 구조이기 때문에 Simulation 시작 시 초기 상태가 불명확할 수 있다.

따라서 초기 구간에서 Reset을 인가하는 것이 좋다.

예:

```text
0 ns ~ 10 ns

S = 0
R = 1
```

결과:

```text
Q  = 0
QB = 1
```

로 초기화된다.

---

# 21. Propagation Delay

주요 Delay는 다음과 같다.

## Set Delay

```text
S : 0 → 1
      ↓
Q : 0 → 1
```

## Reset Delay

```text
R : 0 → 1
      ↓
Q : 1 → 0
```

일반적으로:

```text
Input 50% VDD Crossing

→

Output 50% VDD Crossing
```

사이의 시간을 측정한다.

---

# 22. NOR Gate의 Delay 특성

NOR Gate에서는 Pull-Up Network에 PMOS가 Series로 연결된다.

```text
VDD
 │
PMOS
 │
PMOS
 │
OUT
```

PMOS는 NMOS에 비해 동일 크기에서 일반적으로 Drive Strength가 낮기 때문에 NOR Gate의 LOW → HIGH 전환은 상대적으로 느려질 수 있다.

따라서 Full-Custom 설계에서는 PMOS Width를 조절하여 Rise / Fall Delay를 맞추는 방법을 사용할 수 있다.

---

# 23. Layout 구조

권장 Floorplan:

```text
VDD ────────────────────────────────

       ┌──────────┐
S ────>│ NOR_QB   │─────────────> QB
       └──────────┘
             ▲
             │ Q Feedback
             │
             ▼
       ┌──────────┐
R ────>│ NOR_Q    │─────────────> Q
       └──────────┘
             ▲
             │ QB Feedback
             │

VSS ────────────────────────────────
```

---

# 24. Layout 주요 고려사항

Feedback 경로:

```text
Q  → NOR_QB

QB → NOR_Q
```

를 가능한 한 짧게 유지한다.

특히 NOR2는 PMOS가 Series 구조이므로 PMOS Active / Diffusion 구조가 NAND와 다르다.

Layout에서는:

```text
PMOS Series Connection

NMOS Parallel Connection
```

을 정확하게 구현해야 한다.

---

# 25. DRC / LVS

## DRC

확인 항목:

```text
Active Width
Active Spacing
Poly Width
Poly Spacing
Metal Width
Metal Spacing
Contact Enclosure
Via Enclosure
N-Well Rule
Implant Rule
```

---

## LVS

논리 연결을 확인한다.

```text
S + Q  → NOR → QB

R + QB → NOR → Q
```

주요 오류 후보:

```text
Q / QB Port Mapping 오류

Feedback 방향 오류

S / R Input 반전

VDD / VSS 누락
```

---

# 26. Post-Layout Simulation

PEX 이후 Extracted View를 사용하여 Transient Simulation을 다시 수행한다.

비교 항목:

```text
Set Delay

Reset Delay

Rise Time

Fall Time

Power Consumption

Q / QB Feedback Stability
```

Schematic Simulation보다 Layout Simulation에서 Parasitic R/C 영향으로 Delay가 증가할 수 있다.

---

# 27. NAND형과 NOR형 비교

| 항목 | NAND SR Latch | NOR SR Latch |
|---|---|---|
| 기본 Gate | NAND2 × 2 | NOR2 × 2 |
| Set 입력 | Active-Low | Active-High |
| Reset 입력 | Active-Low | Active-High |
| Hold | S_B=1, R_B=1 | S=0, R=0 |
| Set | S_B=0, R_B=1 | S=1, R=0 |
| Reset | S_B=1, R_B=0 | S=0, R=1 |
| Invalid | S_B=0, R_B=0 | S=1, R=1 |
| Invalid Output | Q=1, QB=1 | Q=0, QB=0 |
| Transistor 수 | 8 | 8 |
| 입력 특성 | Active-Low | Active-High |

---

# 28. 기억하기 쉬운 방법

## NAND SR Latch

```text
0이 Active

00 → Invalid

11 → Hold
```

즉:

```text
NAND = Active-Low
```

---

## NOR SR Latch

```text
1이 Active

11 → Invalid

00 → Hold
```

즉:

```text
NOR = Active-High
```

---

# 29. 장점

- 매우 단순한 1-bit Storage 구조
- NOR Gate 2개만 필요
- Active-High Set / Reset으로 직관적인 제어 가능
- Latch 및 Flip-Flop 동작 이해에 적합
- Full-Custom Sequential Circuit 설계의 기본 구조

---

# 30. 단점

- S와 R을 동시에 1로 만들 수 없음
- Invalid State 존재
- 초기 상태가 불명확할 수 있음
- Setup / Reset 입력 전환 순서에 주의 필요
- Cross-Coupled Feedback 때문에 Simulation 초기화가 필요할 수 있음

---

# 31. 향후 확장

NOR SR Latch의 원리는 다음 회로를 이해하는 기초가 된다.

```text
SR Latch
   ↓
Gated SR Latch
   ↓
Gated D Latch
   ↓
D Flip-Flop
   ↓
Register
   ↓
Counter / MAC / Sequential Logic
```

---

# 32. 결론

NOR 기반 SR Latch는 두 개의 NOR Gate를 Cross-Coupled 형태로 연결하여 1-bit 정보를 저장하는 기본적인 Sequential Circuit이다.

NOR SR Latch는 Active-High 방식으로 동작한다.

```text
S = 1, R = 0 → Set

S = 0, R = 1 → Reset

S = 0, R = 0 → Hold

S = 1, R = 1 → Invalid
```

NAND 기반 SR Latch와 기본적인 저장 원리는 동일하지만 입력의 Active Level과 Invalid State의 출력이 반대이다.

Full-Custom 설계에서는 NOR Gate의 Series PMOS 구조, Cross-Coupled Feedback Routing, Initial State 설정, Propagation Delay 및 DRC/LVS 검증이 주요 설계 요소이다.
