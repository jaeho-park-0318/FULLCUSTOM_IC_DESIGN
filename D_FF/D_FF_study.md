# D Flip-Flop(DFF) 설계 보고서

## 1. 설계 개요

### 1.1 설계명

**D Flip-Flop (DFF)**

### 1.2 설계 목적

D Flip-Flop은 하나의 데이터 입력 `D`와 Clock 입력 `CLK`를 이용하여 **Clock Edge 순간의 입력 데이터를 저장하는 1-bit 순차 논리 회로**이다.

Gated D Latch는 Enable이 활성화된 전체 구간 동안 입력 D가 출력 Q로 전달되는 Level-Sensitive 구조인 반면, D Flip-Flop은 특정 Clock Edge에서만 입력 D를 저장하는 **Edge-Triggered Storage Element**이다.

본 설계의 목표는 다음과 같다.

- D Flip-Flop의 기본 동작 원리 이해
- Gated D Latch와 DFF의 차이 이해
- Master-Slave 구조 이해
- NAND 기반 또는 D Latch 기반 DFF 설계
- Rising-Edge Triggered 동작 검증
- Setup Time / Hold Time 개념 이해
- Clock-to-Q Delay 측정
- Cadence Virtuoso 기반 Transient Simulation
- Full-Custom Layout 및 DRC/LVS 검증
- 향후 Register 및 MAC Accumulator 설계를 위한 Storage Cell 확보

---

# 2. D Flip-Flop 기본 개념

D Flip-Flop의 기본 입력과 출력은 다음과 같다.

## Input

```text
D
CLK
```

## Output

```text
Q
QB
```

D Flip-Flop은 특정 Clock Edge에서 D 값을 저장한다.

Rising-Edge Triggered DFF의 경우:

```text
CLK Rising Edge 발생
        ↓
Q(next) = D
```

Clock Edge가 아닌 시간에는:

```text
Q(next) = Q(previous)
```

즉 이전 상태를 유지한다.

---

# 3. D Latch와 D Flip-Flop 차이

## D Latch

D Latch는 **Level-Sensitive**이다.

```text
E = 1 → Q follows D
E = 0 → Q holds previous value
```

Enable이 HIGH인 전체 구간 동안 D의 변화가 Q로 전달된다.

---

## D Flip-Flop

D Flip-Flop은 **Edge-Triggered**이다.

Rising-Edge DFF의 경우:

```text
CLK ↑ 순간에만 D 저장
```

그 외의 시간에는 D가 변해도 Q가 변하지 않는다.

---

## 비교

| 항목 | Gated D Latch | D Flip-Flop |
|---|---|---|
| 동작 방식 | Level-Sensitive | Edge-Triggered |
| 제어 입력 | Enable | Clock |
| 데이터 저장 | Enable HIGH 동안 | Clock Edge 순간 |
| 입력 변화 전달 | Enable HIGH 동안 가능 | Edge 순간만 |
| Timing 제어 | 상대적으로 어려움 | 상대적으로 명확 |
| Register 구성 | 가능 | 매우 적합 |
| 동기식 회로 | 제한적 | 일반적으로 사용 |

---

# 4. Rising-Edge Triggered DFF

본 설계에서는 **Rising-Edge Triggered DFF**를 기준으로 한다.

동작은 다음과 같다.

```text
CLK = 0 → Q 유지

CLK 0 → 1
        ↑
        Rising Edge
        ↓
        D 값 저장

CLK = 1 → Q 유지
```

예를 들어:

```text
Rising Edge 직전 D = 1
```

이면 Rising Edge 이후:

```text
Q = 1
```

이 된다.

---

# 5. Truth Table

Rising-Edge Triggered DFF의 동작은 다음과 같다.

| CLK | D | Q(next) | 동작 |
|---|---:|---|---|
| No Edge | X | Q(previous) | Hold |
| Rising Edge | 0 | 0 | Store 0 |
| Rising Edge | 1 | 1 | Store 1 |

여기서 `X`는 Don't Care를 의미한다.

---

# 6. Master-Slave DFF 구조

D Flip-Flop을 구현하는 대표적인 방법은 **두 개의 D Latch를 Master-Slave 구조로 연결하는 것**이다.

```text
D
│
▼
┌─────────────────┐
│  Master D Latch │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Slave D Latch  │
└────────┬────────┘
         │
         ▼
         Q
```

Master와 Slave는 서로 반대 Clock Phase에서 동작한다.

---

# 7. Clock Phase

Rising-Edge Triggered DFF를 구성하는 한 가지 방법은:

```text
Master Enable = NOT(CLK)

Slave Enable  = CLK
```

로 설정하는 것이다.

즉:

```text
CLK = 0
Master = Open
Slave  = Closed
```

```text
CLK = 1
Master = Closed
Slave  = Open
```

이다.

---

# 8. Master-Slave 동작 원리

## 8.1 CLK = 0

```text
Master Enable = 1
Slave Enable  = 0
```

따라서:

```text
Master → Transparent
Slave  → Hold
```

입력 D는 Master 내부에 전달되지만 최종 Q에는 아직 전달되지 않는다.

---

## 8.2 CLK Rising Edge

Clock이:

```text
0 → 1
```

로 변하면:

```text
Master → Hold
Slave  → Transparent
```

가 된다.

Master가 Rising Edge 직전에 저장하고 있던 값을 Slave가 받아 최종 Q로 전달한다.

따라서:

```text
CLK ↑ → Q(next) = D
```

가 된다.

---

## 8.3 CLK = 1

```text
Master = Closed
Slave  = Open
```

이므로 D가 변하더라도 Master 출력은 변하지 않는다.

따라서 Slave에 전달되는 값도 고정되어 Q가 안정적으로 유지된다.

---

# 9. 전체 구조

Gated D Latch 두 개를 이용한 Rising-Edge DFF는 다음과 같이 표현할 수 있다.

```text
                    CLK_B                     CLK

D ───────────────>┌──────────────┐
                  │ Master       │
                  │ D Latch      │
                  └──────┬───────┘
                         │
                         │ QM
                         ▼
                  ┌──────────────┐
                  │ Slave        │────────────> Q
                  │ D Latch      │
                  └──────────────┘
```

여기서:

```text
CLK_B = NOT(CLK)
```

이다.

---

# 10. 내부 Net Label 권장안

Schematic과 Layout을 관리하기 위해 다음 내부 Net Label 사용을 권장한다.

```text
CLK_B

QM
QMB

Q
QB
```

Master 내부까지 구체적으로 표시한다면:

```text
MASTER_D_B
MASTER_S_B
MASTER_R_B

SLAVE_D_B
SLAVE_S_B
SLAVE_R_B
```

등으로 확장할 수 있다.

---

# 11. 권장 Port Mapping

## Input

```text
D
CLK
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
D ───────>│               │──────> Q
CLK ─────>│      DFF      │
          │               │──────> QB
          └───────────────┘
```

Clock 입력에는 Edge-Triggered 회로임을 표시하기 위해 Symbol에 삼각형을 추가할 수도 있다.

```text
          ┌───────────────┐
D ───────>│ D           Q │──────>
CLK ─────>|               │
          └───────────────┘
```

---

# 12. 권장 Cell Hierarchy

이미 Gated D Latch를 만들어 둔 경우 가장 간단한 Hierarchy는:

```text
INV
NAND2
  ↓
GATED_D_LATCH
  ↓
DFF
```

이다.

DFF 내부에는:

```text
GATED_D_LATCH × 2
INV × 1
```

을 사용한다.

---

# 13. DFF 내부 연결

Rising-Edge Triggered DFF를 기준으로:

```text
CLK → INV → CLK_B
```

Master Latch:

```text
D input      = D
Enable input = CLK_B
```

Slave Latch:

```text
D input      = QM
Enable input = CLK
```

전체 Mapping:

```text
CLK → INV → CLK_B

D + CLK_B → MASTER

MASTER.Q → QM

QM + CLK → SLAVE

SLAVE.Q  → Q
SLAVE.QB → QB
```

---

# 14. Schematic 연결 예시

```text
                       ┌─────────┐
CLK ──────────────────>│  INV    │──────> CLK_B
                       └─────────┘


D ────────────────────────────────────────┐
                                          │
                                          ▼
                              ┌────────────────────┐
CLK_B ───────────────────────>│ MASTER D LATCH     │
                              │                    │
                              └─────────┬──────────┘
                                        │
                                       QM
                                        │
                                        ▼
                              ┌────────────────────┐
CLK ─────────────────────────>│ SLAVE D LATCH      │
                              │                    │
                              └─────────┬──────────┘
                                        │
                                        ├──────> Q
                                        │
                                        └──────> QB
```

---

# 15. Cadence Virtuoso Schematic 설계 순서

```text
1. INV Cell 준비
2. NAND2 Cell 준비
3. GATED_D_LATCH Cell 준비
4. 각 Cell의 Symbol 생성 확인

5. 새로운 DFF Cell 생성

6. GATED_D_LATCH Symbol 2개 배치
   - Master
   - Slave

7. INV Symbol 1개 배치

8. CLK → INV 연결
9. INV Output → CLK_B Label

10. D → Master.D 연결
11. CLK_B → Master.E 연결

12. Master.Q → QM Label

13. QM → Slave.D 연결
14. CLK → Slave.E 연결

15. Slave.Q → Q Output
16. Slave.QB → QB Output

17. D Input Pin 생성
18. CLK Input Pin 생성
19. Q / QB Output Pin 생성
20. VDD / VSS 연결

21. Check and Save
22. DFF Symbol 생성
23. Testbench 생성
24. Transient Simulation 수행
```

---

# 16. Simulation Testbench

DFF는 Transient Analysis로 검증한다.

주요 입력:

```text
D
CLK
```

두 입력 모두 `vpulse`를 사용할 수 있다.

예:

```text
CLK:
Period = 20 ns
Pulse Width = 10 ns

D:
CLK보다 느리거나 다른 주기로 설정
```

D와 CLK Edge가 항상 겹치지 않도록 처음에는 여유 있게 설정하는 것이 좋다.

---

# 17. Simulation에서 확인할 핵심

Rising-Edge Triggered DFF라면:

```text
CLK ↑ 순간의 D 값
      ↓
다음 Q 상태
```

를 확인한다.

예:

```text
CLK ↑
D = 0

→ Q = 0
```

다음 Rising Edge:

```text
CLK ↑
D = 1

→ Q = 1
```

Clock Edge 사이에서 D가 변해도 Q는 변하면 안 된다.

---

# 18. 예제 Waveform

개념적인 Waveform은 다음과 같다.

```text
Time ───────────────────────────────────────>

CLK     ____----____----____----____----
            ↑       ↑       ↑       ↑

D       ______----________------________

Q       __________----____________------_
            ↑       ↑       ↑
        CLK Edge에서만 변경
```

중요한 점은:

```text
D 변화 시점 ≠ Q 변화 시점
```

이라는 것이다.

Q는 D 자체가 변화할 때가 아니라 **Clock Edge에 맞추어 변화**해야 한다.

---

# 19. Clock-to-Q Delay

DFF의 중요한 Timing Parameter 중 하나이다.

Clock Rising Edge가 발생한 후 Q가 실제로 변화하기까지 일정 시간이 필요하다.

이를:

```text
Clock-to-Q Delay
```

라고 한다.

표기는 일반적으로:

```text
t_CQ
```

또는:

```text
t_CLK-Q
```

등으로 표현한다.

측정 방법:

```text
CLK 50% VDD Crossing
        ↓
Q 50% VDD Crossing
```

사이의 시간을 측정한다.

---

# 20. Setup Time

D 입력은 Clock Edge가 발생하기 직전에 일정 시간 동안 안정되어 있어야 한다.

이를 **Setup Time**이라고 한다.

```text
             CLK Rising Edge
                   ↑
                   │
D ───── stable ────┤
      <--- t_setup -->
```

즉:

```text
Clock Edge 직전에 D가 너무 늦게 변화하면
올바른 값이 저장되지 않을 수 있다.
```

---

# 21. Hold Time

Clock Edge 이후에도 D가 일정 시간 동안 유지되어야 한다.

이를 **Hold Time**이라고 한다.

```text
CLK Rising Edge
       ↑
       │
       ├──── D must remain stable
       │
       <--- t_hold --->
```

Setup/Hold 조건을 위반하면 DFF 내부 Node가 불안정해질 수 있다.

---

# 22. Metastability

D가 Clock Edge와 거의 동시에 변화하면 DFF 내부에서 0과 1 중 어느 상태로 결정될지 늦어지는 현상이 발생할 수 있다.

이를 **Metastability**라고 한다.

즉:

```text
Setup/Hold Violation
        ↓
Internal Node 불안정
        ↓
Q 결정 시간 증가
```

가 발생할 수 있다.

Full-Custom Simulation에서는 D와 CLK의 Edge 간격을 점점 줄여가며 이를 확인할 수 있다.

---

# 23. 초기 상태 문제

Reset이 없는 일반 DFF는 Simulation 시작 시 Q의 초기 상태가 정해지지 않을 수 있다.

특히 Cross-Coupled NAND 기반 Latch가 포함되어 있다면:

```text
Q = X
QB = X
```

또는 중간 전압에서 시작할 가능성이 있다.

해결 방법 중 하나는 Simulation 초기에 D 값을 고정하고 정상적인 Clock Edge를 한 번 이상 인가하는 것이다.

예:

```text
초기:

D = 0

첫 번째 CLK Rising Edge 발생

→ Q = 0
```

이후 정상적인 테스트를 시작한다.

---

# 24. Reset이 필요한 이유

실제 Register나 MAC Accumulator에서는 초기값이 중요하다.

예를 들어 MAC:

```text
ACC(next) = ACC + A × B
```

에서 ACC 초기값이 불명확하면 전체 연산 결과도 불명확해진다.

따라서 이후에는 일반 DFF에 Reset 기능을 추가한:

```text
DFF_RST
```

구조로 확장하는 것이 좋다.

---

# 25. DFF with Reset

Reset 기능을 추가하면:

```text
RESET 활성화 → Q = 0
```

으로 초기 상태를 강제로 결정할 수 있다.

예:

```text
RESET = 1 → Q = 0
RESET = 0 → Normal DFF Operation
```

또는 Active-Low Reset이라면:

```text
RESET_B = 0 → Q = 0
RESET_B = 1 → Normal
```

이다.

프로젝트 전체에서는 Active-High 또는 Active-Low 중 하나를 정해 일관되게 사용하는 것이 중요하다.

---

# 26. DFF와 Register 관계

DFF 하나는 1-bit를 저장한다.

따라서 N-bit Register는 DFF N개로 구성할 수 있다.

예:

```text
D<0> → DFF0 → Q<0>
D<1> → DFF1 → Q<1>
D<2> → DFF2 → Q<2>
...
```

모든 DFF의 Clock은 동일하게 연결한다.

---

# 27. 10-bit Register 예시

현재 MAC 설계에서는 10-bit Accumulator Register가 필요하므로:

```text
DFF × 10
```

으로 구성할 수 있다.

```text
SUM<0> ──> DFF0 ──> ACC<0>
SUM<1> ──> DFF1 ──> ACC<1>
SUM<2> ──> DFF2 ──> ACC<2>
...
SUM<9> ──> DFF9 ──> ACC<9>
```

Clock은 모두:

```text
CLK
```

에 연결한다.

---

# 28. MAC과 DFF의 관계

현재 MAC 구조는:

```text
A, B
 ↓
MUL4
 ↓
RCA10
 ↓
SUM<9:0>
 ↓
REG10
 ↓
ACC<9:0>
```

이다.

REG10 내부에 DFF 10개를 사용하면:

```text
RCA10.SUM<9:0>
       ↓
DFF × 10
       ↓
ACC_Q<9:0>
       ↓
RCA10 Feedback
```

구조가 된다.

즉 DFF는 MAC에서 **Accumulator의 상태를 저장하는 핵심 순차회로**이다.

---

# 29. Layout 설계 방향

Master-Slave DFF Layout은 크게:

```text
CLK INV
Master Latch
Slave Latch
```

세 영역으로 나눌 수 있다.

개념적 Floorplan:

```text
VDD ───────────────────────────────────────

┌─────┐   ┌────────────────┐   ┌────────────────┐
│ INV │   │ MASTER LATCH   │   │ SLAVE LATCH    │
│ CLK │──>│                │──>│                │──> Q
└─────┘   └────────────────┘   └────────────────┘

VSS ───────────────────────────────────────
```

Data Flow를:

```text
D → Master → Slave → Q
```

방향으로 배치하면 Routing이 단순해진다.

---

# 30. Clock Routing

DFF에서는 Clock 신호가 매우 중요하다.

Clock은:

```text
CLK
CLK_B
```

두 신호로 사용될 수 있다.

Clock Inverter를 Master와 Slave 근처에 배치하여 Clock 배선을 짧게 유지하는 것이 좋다.

특히:

```text
CLK → Slave
CLK_B → Master
```

경로의 Delay 차이가 너무 커지면 내부 Race 문제가 증가할 수 있다.

---

# 31. Master-Slave 배치

Master와 Slave Latch는 서로 가까이 배치한다.

```text
Master.Q → Slave.D
```

경로는 DFF 내부의 중요한 Data Path이므로 가능한 한 짧게 구성한다.

권장 방향:

```text
D → MASTER → SLAVE → Q
```

이다.

---

# 32. Feedback Routing

각 D Latch 내부에는 Cross-Coupled Feedback이 존재한다.

Master:

```text
QM → Master QB-side NAND
QMB → Master Q-side NAND
```

Slave:

```text
Q → Slave QB-side NAND
QB → Slave Q-side NAND
```

이 Feedback 경로들은 가능한 한 짧게 배치한다.

---

# 33. Power Rail

모든 하위 Cell에서 동일한 VDD/VSS Rail 방향을 사용하면 Hierarchical Layout에 유리하다.

```text
VDD ─────────────────────────────

        PMOS Region

        Logic

        NMOS Region

VSS ─────────────────────────────
```

Master와 Slave를 동일한 Cell Height로 구성하면 이후 Register Layout에도 재사용하기 쉽다.

---

# 34. Layout Hierarchy

권장 Layout 순서:

```text
INV Layout
   ↓
NAND2 Layout
   ↓
GATED_D_LATCH Layout
   ↓
DFF Layout
```

각 단계에서:

```text
DRC PASS
LVS PASS
```

후 상위 Cell로 올라가는 것이 좋다.

---

# 35. DRC

DFF Layout 완료 후 Design Rule Check를 수행한다.

주요 확인 항목:

```text
Poly Width / Spacing
Active Width / Spacing
Metal Width / Spacing
Via Enclosure
Contact Enclosure
N-Well Rule
Implant Rule
```

정확한 값은 사용하는 PDK Design Rule Manual을 따른다.

---

# 36. LVS

DFF에서 LVS 오류가 발생하기 쉬운 부분은 다음과 같다.

```text
Master / Slave 연결

CLK / CLK_B 연결

Master.Q → Slave.D

Q / QB Feedback

VDD / VSS

Pin Name
```

특히 Master와 Slave Clock Phase가 반대로 연결되어 있는지 반드시 확인한다.

---

# 37. Post-Layout Simulation

PEX 이후 Extracted View를 사용하여 다시 Transient Simulation을 수행한다.

비교할 항목:

```text
Clock-to-Q Delay

D-to-Q 동작

Rise Time

Fall Time

Setup Time

Hold Time

Dynamic Power
```

Layout의 Parasitic R/C로 인해 일반적으로 Clock-to-Q Delay가 증가할 수 있다.

---

# 38. Critical Path

Master-Slave DFF에서 중요한 Data Path는 대략:

```text
D
 ↓
Master Input Logic
 ↓
Master Storage Node
 ↓
Slave Input Logic
 ↓
Slave Storage Node
 ↓
Q
```

이다.

하지만 실제 DFF Timing에서는 전체 D-to-Q Combinational Delay보다:

```text
CLK Edge
 ↓
Slave Enable
 ↓
Internal Node
 ↓
Q
```

경로에 해당하는 **Clock-to-Q Delay**가 핵심 Timing Parameter가 된다.

---

# 39. 장점

- Clock Edge 기반 동작
- 동기식 회로 설계에 적합
- D 입력 하나만 사용
- SR Latch의 금지 상태 없음
- Register 구성에 적합
- Counter / FSM / MAC 등에 활용 가능
- Sequential Circuit의 핵심 기본 Cell

---

# 40. 단점

- D Latch보다 Transistor 수 증가
- Master/Slave 두 개의 Storage Stage 필요
- Clock Distribution 필요
- Clock Power 증가
- Setup/Hold Constraint 존재
- Clock Skew에 민감할 수 있음

---

# 41. Gated D Latch와 DFF 비교

| 항목 | Gated D Latch | D Flip-Flop |
|---|---|---|
| 입력 | D, E | D, CLK |
| 동작 | Level-Sensitive | Edge-Triggered |
| Storage | 1 bit | 1 bit |
| 기본 구성 | NAND4 + INV | D Latch ×2 + CLK INV |
| Transparent 상태 | 존재 | 외부에서 사실상 없음 |
| Timing 관리 | Enable Level | Clock Edge |
| Register 설계 | 가능 | 일반적 |
| MAC Accumulator | 가능 | 더 적합 |

---

# 42. 향후 확장

DFF를 기반으로 다음 회로를 설계할 수 있다.

```text
DFF
 ↓
DFF with Reset
 ↓
N-bit Register
 ↓
Accumulator
 ↓
MAC
```

또는:

```text
DFF
 ↓
Shift Register
 ↓
Counter
 ↓
State Machine
```

등의 다양한 순차회로로 확장 가능하다.

---

# 43. 현재 MAC 설계에서의 활용

현재 Full-Custom MAC의 전체 구조는 다음과 같다.

```text
A<3:0>
B<3:0>
   │
   ▼
  MUL4
   │
   ▼
P<7:0>
   │
   ▼
Zero Extension
   │
   ▼
 RCA10
   │
   ▼
SUM<9:0>
   │
   ▼
 REG10
   │
   ▼
ACC<9:0>
   │
   └──────────────> RCA10 Feedback
```

여기서:

```text
REG10 = DFF × 10
```

으로 구성할 수 있다.

따라서 DFF의 동작과 Layout 성능은 MAC 전체의 Clock Frequency, Power, Area에 직접 영향을 준다.

---

# 44. 최종 설계 흐름

권장 설계 순서는 다음과 같다.

```text
NAND2
 ↓
INV
 ↓
GATED_D_LATCH
 ↓
DFF
 ↓
DFF_RST
 ↓
REG10
 ↓
MAC4
```

각 Cell마다:

```text
Schematic
 ↓
Simulation
 ↓
Symbol
 ↓
Layout
 ↓
DRC
 ↓
LVS
 ↓
PEX
```

순서로 검증하는 것이 좋다.

---

# 45. 결론

D Flip-Flop은 Clock Edge 순간의 입력 D를 저장하고 다음 Clock Edge까지 해당 상태를 유지하는 대표적인 Edge-Triggered Sequential Circuit이다.

Gated D Latch 두 개를 Master-Slave 구조로 연결하고 서로 반대 Clock Phase를 적용하면 Rising-Edge 또는 Falling-Edge Triggered DFF를 구현할 수 있다.

본 Full-Custom 설계에서는:

```text
Master Enable = CLK_B
Slave Enable  = CLK
```

방식의 Rising-Edge Triggered DFF를 기본 구조로 사용할 수 있다.

설계 시 중요한 요소는 다음과 같다.

```text
Clock-to-Q Delay
Setup Time
Hold Time
Clock / CLK_B Routing
Master-Slave Data Path
Feedback Routing
Transistor Sizing
DRC / LVS
Post-Layout Parasitic
```

또한 본 DFF는 이후:

```text
DFF
 ↓
10-bit Register
 ↓
Accumulator
 ↓
MAC
```

구조로 확장되며, 현재 진행 중인 Full-Custom MAC 설계에서 Accumulator의 상태 저장을 담당하는 핵심 순차회로로 활용할 수 있다.
