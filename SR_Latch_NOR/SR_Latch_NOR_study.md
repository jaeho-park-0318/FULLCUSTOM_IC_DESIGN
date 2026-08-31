# NAND 기반 SR Latch 설계 보고서

## 1. 설계 개요

### 1.1 설계명

**NAND-Based SR Latch**

### 1.2 설계 목적

SR Latch는 1-bit의 정보를 저장할 수 있는 가장 기본적인 순차 논리 회로이다.

NAND 기반 SR Latch는 두 개의 NAND Gate를 서로 Cross-Coupled 형태로 연결하여 구성하며, 출력 `Q`와 `QB`가 서로의 입력으로 Feedback되는 구조를 이용해 이전 상태를 저장한다.

본 설계의 목표는 다음과 같다.

- SR Latch의 기본 저장 원리 이해
- NAND Gate 기반 Cross-Coupled 구조 이해
- Active-Low Set / Reset 입력 동작 이해
- Set / Reset / Hold / Invalid 상태 분석
- Cadence Virtuoso 기반 Transient Simulation
- Full-Custom Schematic 및 Layout 설계
- DRC / LVS / PEX 검증

---

# 2. SR Latch 기본 개념

SR은 다음을 의미한다.

```text
S = Set
R = Reset
```

SR Latch는 기본적으로 다음 출력을 가진다.

```text
Q
QB
```

정상 상태에서는:

```text
QB = NOT(Q)
```

관계를 만족한다.

즉:

```text
Q = 1 → QB = 0

Q = 0 → QB = 1
```

이다.

---

# 3. NAND SR Latch의 입력

NAND 기반 SR Latch는 입력이 **Active-Low**이다.

따라서 일반적으로 입력을 다음과 같이 표시한다.

```text
S_B
R_B
```

여기서 `_B`는 Active-Low 또는 Inverted Signal임을 의미한다.

```text
S_B = 0 → Set

R_B = 0 → Reset
```

이다.

---

# 4. NAND SR Latch 구조

두 개의 NAND Gate를 Cross-Coupled 형태로 연결한다.

```text
                   ┌───────────┐
S_B ──────────────>│           │
                   │   NAND    ├────────────> Q
             QB ──>│           │
                   └───────────┘
                        ▲
                        │
                        │ Cross-Coupled
                        │ Feedback
                        ▼
                   ┌───────────┐
R_B ──────────────>│           │
                   │   NAND    ├────────────> QB
              Q ──>│           │
                   └───────────┘
```

출력 Q가 아래 NAND의 입력으로 Feedback되고, 출력 QB가 위 NAND의 입력으로 Feedback된다.

---

# 5. 논리식

위쪽 NAND Gate:

```text
Q = NOT(S_B AND QB)
```

아래쪽 NAND Gate:

```text
QB = NOT(R_B AND Q)
```

두 식이 서로 Feedback 관계를 형성하기 때문에 이전 출력 상태를 저장할 수 있다.

---

# 6. Truth Table

NAND SR Latch는 Active-Low 입력을 사용한다.

| S_B | R_B | Q(next) | QB(next) | 동작 |
|---:|---:|---:|---:|---|
| 1 | 1 | Q(previous) | QB(previous) | Hold |
| 0 | 1 | 1 | 0 | Set |
| 1 | 0 | 0 | 1 | Reset |
| 0 | 0 | 1 | 1 | Invalid |

---

# 7. Hold 상태

입력:

```text
S_B = 1
R_B = 1
```

이라고 하자.

이 경우 NAND Gate의 외부 입력이 모두 비활성화된 상태가 된다.

출력은 이전 상태에 의해 결정된다.

```text
Q(next)  = Q(previous)

QB(next) = QB(previous)
```

따라서 현재 값을 저장한다.

이를 **Hold State**라고 한다.

---

# 8. Set 상태

입력:

```text
S_B = 0
R_B = 1
```

위쪽 NAND Gate에서:

```text
Q = NOT(0 AND QB)

Q = 1
```

이 된다.

Q가 1이 되면 아래쪽 NAND Gate는:

```text
QB = NOT(1 AND 1)

QB = 0
```

이 된다.

따라서 최종 출력:

```text
Q  = 1
QB = 0
```

이다.

즉 **Set 상태**이다.

---

# 9. Reset 상태

입력:

```text
S_B = 1
R_B = 0
```

아래쪽 NAND Gate에서:

```text
QB = NOT(0 AND Q)

QB = 1
```

이 된다.

QB가 1이 되면 위쪽 NAND Gate에서는:

```text
Q = NOT(1 AND 1)

Q = 0
```

이 된다.

따라서:

```text
Q  = 0
QB = 1
```

이 된다.

즉 **Reset 상태**이다.

---

# 10. Invalid 상태

입력:

```text
S_B = 0
R_B = 0
```

이면 두 NAND Gate 모두 하나의 입력으로 0을 받게 된다.

따라서:

```text
Q  = 1
QB = 1
```

이 된다.

그러나 정상적인 SR Latch에서는:

```text
QB = NOT(Q)
```

이어야 하므로 Q와 QB가 동시에 1인 상태는 정상적인 저장 상태가 아니다.

따라서:

```text
S_B = 0
R_B = 0
```

은 **Invalid State 또는 Forbidden State**이다.

---

# 11. Invalid 상태 해제 문제

다음 상태에서:

```text
S_B = 0
R_B = 0
```

두 입력을 동시에:

```text
S_B = 1
R_B = 1
```

로 변경하면 두 NAND Gate가 거의 동시에 상태를 결정하려고 한다.

실제 CMOS 회로에서는:

```text
Gate Delay
Transistor Mismatch
Parasitic Capacitance
Routing Delay
```

등에 의해 최종적으로 Q와 QB 중 어느 쪽이 1이 될지 예측하기 어려울 수 있다.

따라서 Invalid 상태는 반드시 피해야 한다.

---

# 12. Active-Low 입력의 의미

NAND SR Latch에서 가장 중요한 특징은:

```text
입력 0이 동작을 발생시킨다.
```

는 것이다.

즉:

```text
S_B = 0 → Set Active

R_B = 0 → Reset Active
```

이다.

반대로:

```text
S_B = 1
R_B = 1
```

이면 두 입력이 비활성 상태이며 이전 값을 유지한다.

---

# 13. 권장 Port Mapping

## Input

```text
S_B
R_B
```

또는 schematic symbol에서:

```text
SN
RN
```

과 같이 표현할 수도 있다.

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
           ┌────────────────┐
S_B ──────>│                │──────> Q
R_B ──────>│ NAND_SR_LATCH  │
           │                │──────> QB
           └────────────────┘
```

---

# 14. CMOS NAND2 구조

2-input CMOS NAND Gate는:

```text
PMOS → Parallel

NMOS → Series
```

구조를 갖는다.

```text
                   VDD
                    │
             ┌──────┴──────┐
             │             │
           PMOS A        PMOS B
             │             │
             └──────┬──────┘
                    │
                   OUT
                    │
                  NMOS A
                    │
                  NMOS B
                    │
                   VSS
```

---

# 15. NAND SR Latch의 Transistor 수

일반적인 Static CMOS NAND2는:

```text
PMOS = 2
NMOS = 2
```

총:

```text
4 Transistors
```

를 사용한다.

NAND SR Latch는 NAND2 두 개를 사용하므로:

```text
2 × NAND2
```

따라서 총 Transistor 수는:

```text
8 Transistors
```

이다.

---

# 16. 권장 Cell Hierarchy

```text
PMOS / NMOS
     ↓
   NAND2
     ↓
NAND_SR_LATCH
```

NAND2를 먼저 Schematic 및 Layout까지 완성한 후 상위 SR Latch에서 두 개를 재사용하는 것이 좋다.

---

# 17. Cadence Virtuoso Schematic 설계 순서

```text
1. NAND2 Cell 생성

2. PMOS × 2 배치
3. NMOS × 2 배치

4. PMOS Parallel 연결
5. NMOS Series 연결

6. VDD / VSS 연결

7. NAND2 Simulation

8. NAND2 Symbol 생성

9. NAND_SR_LATCH Cell 생성

10. NAND2 Symbol × 2 배치

11. 첫 번째 NAND Output을 Q로 설정

12. 두 번째 NAND Output을 QB로 설정

13. Q → 두 번째 NAND 입력으로 Feedback

14. QB → 첫 번째 NAND 입력으로 Feedback

15. S_B Input Pin 생성

16. R_B Input Pin 생성

17. Q / QB Output Pin 생성

18. VDD / VSS 연결

19. Check and Save

20. Symbol 생성

21. Testbench 구성

22. Transient Simulation
```

---

# 18. Simulation 검증

Transient Simulation에서 다음 상태를 순서대로 확인한다.

## Initial Reset

```text
S_B = 1
R_B = 0
```

예상:

```text
Q  = 0
QB = 1
```

---

## Hold

```text
S_B = 1
R_B = 1
```

예상:

```text
Q = previous value
```

---

## Set

```text
S_B = 0
R_B = 1
```

예상:

```text
Q  = 1
QB = 0
```

---

## Hold

```text
S_B = 1
R_B = 1
```

이전 Set 상태를 유지해야 한다.

---

## Reset

```text
S_B = 1
R_B = 0
```

예상:

```text
Q  = 0
QB = 1
```

---

# 19. 초기 상태 문제

Cross-Coupled 구조는 Feedback을 이용하기 때문에 Simulation 시작 시 초기 상태가 명확하지 않을 수 있다.

따라서 처음 몇 ns 동안 Set 또는 Reset 입력을 강제로 인가하는 것이 좋다.

예:

```text
0 ns ~ 10 ns

S_B = 1
R_B = 0
```

으로 설정하면:

```text
Q  = 0
QB = 1
```

상태로 초기화할 수 있다.

---

# 20. Propagation Delay

NAND SR Latch에서는 다음 Delay를 측정할 수 있다.

## Set Delay

```text
S_B : 1 → 0
       ↓
Q   : 0 → 1
```

## Reset Delay

```text
R_B : 1 → 0
       ↓
Q   : 1 → 0
```

일반적으로 입력과 출력의:

```text
50% VDD Crossing Point
```

를 기준으로 Propagation Delay를 측정한다.

---

# 21. Layout 구조

권장 Floorplan:

```text
VDD ─────────────────────────────

     ┌──────────┐
S_B →│ NAND_Q   │────────────→ Q
     └──────────┘
           ▲
           │ QB Feedback
           │
           ▼
     ┌──────────┐
R_B →│ NAND_QB  │────────────→ QB
     └──────────┘
           ▲
           │ Q Feedback
           │

VSS ─────────────────────────────
```

---

# 22. Layout 핵심 포인트

Cross-Coupled Feedback 경로:

```text
Q  → NAND_QB

QB → NAND_Q
```

를 최대한 짧게 배치한다.

Feedback Node의 Parasitic Capacitance가 커지면 출력 전환 속도가 느려질 수 있다.

두 NAND의 Layout 구조를 가능한 한 유사하게 설계하면 지나친 Delay 불균형을 줄일 수 있다.

---

# 23. DRC / LVS

## DRC

다음 항목을 확인한다.

```text
Active Width / Spacing
Poly Width / Spacing
Metal Width / Spacing
Contact Enclosure
Via Enclosure
N-Well
Implant
```

---

## LVS

다음 연결을 집중적으로 확인한다.

```text
S_B + QB → NAND → Q

R_B + Q → NAND → QB
```

주요 오류:

```text
Q / QB 연결 반대

Feedback 누락

S_B / R_B Pin 오류

VDD / VSS 누락
```

---

# 24. NAND SR Latch 특징

## 장점

- NAND2 두 개만으로 구현 가능
- 구조가 간단함
- 1-bit 저장 가능
- Active-Low Control 회로에 적합
- D Latch / DFF 설계의 기본 구조

## 단점

- Active-Low 입력이므로 논리 해석에 주의 필요
- Invalid State 존재
- 초기 상태가 불명확할 수 있음
- Set과 Reset을 동시에 활성화하면 안 됨

---

# 25. 결론

NAND 기반 SR Latch는 두 개의 NAND Gate를 Cross-Coupled 형태로 연결하여 1-bit 정보를 저장하는 가장 기본적인 순차 논리 회로 중 하나이다.

NAND SR Latch의 가장 중요한 특징은 Set과 Reset이 Active-Low라는 점이다.

```text
S_B = 0 → Set

R_B = 0 → Reset

S_B = R_B = 1 → Hold

S_B = R_B = 0 → Invalid
```

Full-Custom 설계에서는 Cross-Coupled Feedback Routing, NAND Gate의 Delay, 초기 상태 설정, Invalid State 방지 및 DRC/LVS 검증이 주요 설계 요소이다.
