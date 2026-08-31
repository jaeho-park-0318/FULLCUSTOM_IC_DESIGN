# 2:1 Multiplexer (2MUX) 설계 보고서

## 1. 설계 개요

### 1.1 설계명

**2:1 Multiplexer (2MUX)**

### 1.2 설계 목적

2:1 Multiplexer는 두 개의 데이터 입력 중 하나를 선택하여 하나의 출력으로 전달하는 조합 논리 회로이다.

선택 신호 `S(Select)`의 값에 따라 입력 `A` 또는 `B` 중 하나가 출력 `Y`로 전달된다.

본 설계의 목표는 다음과 같다.

- Multiplexer의 기본 동작 원리 이해
- Select 신호에 따른 Data Path 선택 이해
- 2:1 MUX 논리식 분석
- CMOS Gate 기반 Schematic 설계
- NAND Gate 기반 MUX 구현
- Transmission Gate 기반 MUX 구조 이해
- Cadence Virtuoso 기반 Transient Simulation
- Full-Custom Layout
- DRC / LVS / PEX 검증
- Propagation Delay 및 Power 특성 분석

---

# 2. 2:1 MUX 기본 개념

2:1 Multiplexer는 두 개의 Data Input과 하나의 Select Input을 가진다.

## Input

```text
A
B
S
```

여기서:

```text
A = Data Input 0
B = Data Input 1
S = Select
```

이다.

## Output

```text
Y
```

---

# 3. 기본 동작

Select 입력 `S`에 따라 출력이 결정된다.

```text
S = 0 → Y = A

S = 1 → Y = B
```

즉 Select가 0이면 A가 선택되고, Select가 1이면 B가 선택된다.

---

# 4. Truth Table

| S | A | B | Y |
|---:|---:|---:|---:|
| 0 | 0 | X | 0 |
| 0 | 1 | X | 1 |
| 1 | X | 0 | 0 |
| 1 | X | 1 | 1 |

`X`는 Don't Care를 의미한다.

간단히 표현하면:

```text
S = 0 → Y = A

S = 1 → Y = B
```

이다.

---

# 5. 논리식

2:1 MUX의 기본 Boolean Equation은 다음과 같다.

```text
Y = (A AND NOT(S)) OR (B AND S)
```

즉:

```text
S_B = NOT(S)
```

라고 하면:

```text
Y = (A AND S_B) OR (B AND S)
```

이다.

---

# 6. 논리 구조

기본적인 Gate-Level 구조는 다음과 같다.

```text
                  ┌─────┐
S ───────────────>│ INV │──────> S_B
                  └─────┘
                                  │
                                  │
A ────────────────────────────────┼──┐
                                  │  │
                                  ▼  ▼
                               ┌───────┐
                               │ AND   │
                               └───┬───┘
                                   │
                                   │
                                   ├────────┐
                                            ▼
                                         ┌─────┐
                                         │ OR  │────> Y
                                            ▲
                                   ┌────────┘
                                   │
                               ┌───┴───┐
                               │ AND   │
                               └───────┘
                                  ▲ ▲
                                  │ │
B ────────────────────────────────┘ │
                                    │
S ──────────────────────────────────┘
```

논리적으로는:

```text
A + S_B → AND

B + S   → AND

두 AND 출력 → OR → Y
```

이다.

---

# 7. Select = 0 동작

다음과 같이 설정한다.

```text
S = 0
```

그러면:

```text
S_B = 1
```

이다.

논리식:

```text
Y = (A AND 1) OR (B AND 0)
```

따라서:

```text
Y = A
```

이다.

즉 A가 출력으로 전달된다.

---

# 8. Select = 1 동작

다음과 같이 설정한다.

```text
S = 1
```

그러면:

```text
S_B = 0
```

이다.

논리식:

```text
Y = (A AND 0) OR (B AND 1)
```

따라서:

```text
Y = B
```

이다.

즉 B가 출력으로 전달된다.

---

# 9. 권장 Port Mapping

## Input

```text
A
B
S
```

## Output

```text
Y
```

## Power

```text
VDD
VSS
```

권장 Symbol:

```text
                ┌─────────────┐
A ─────────────>│0            │
                │             │
B ─────────────>│1    2MUX    │────> Y
                │             │
S ─────────────>│SEL          │
                └─────────────┘
```

또는 일반적인 MUX Symbol:

```text
A ──────────────\
                 \
                  >────── Y
                 /
B ──────────────/

        S
        │
        └──────── Select
```

---

# 10. 내부 Net Label

권장 내부 Net Label은 다음과 같다.

```text
S_B

N0
N1

Y
```

예를 들어:

```text
S → INV → S_B

A + S_B → Logic → N0

B + S → Logic → N1

N0 + N1 → Logic → Y
```

와 같이 구성할 수 있다.

---

# 11. 기본 Gate를 이용한 구현

2MUX를 기본 Gate로 구현하면 다음 Cell이 필요하다.

```text
INV × 1

AND2 × 2

OR2 × 1
```

전체 구조:

```text
S
│
▼
INV
│
S_B
│
├────────────┐
│            │
A ── AND ────┤
             │
             ▼
            OR ─────> Y
             ▲
             │
B ── AND ────┤
      ▲
      │
      S
```

---

# 12. NAND Gate 기반 2MUX

Full-Custom CMOS에서는 AND와 OR을 각각 따로 만드는 것보다 NAND Gate를 이용하면 구조를 단순화할 수 있다.

기본식:

```text
Y = (A AND S_B) OR (B AND S)
```

De Morgan's Law를 이용하면:

```text
Y = NOT(
        NOT(A AND S_B)
        AND
        NOT(B AND S)
      )
```

따라서 NAND Gate만으로 MUX를 구성할 수 있다.

---

# 13. NAND 기반 논리식

먼저 Select의 Complement를 생성한다.

```text
S_B = NOT(S)
```

다음 두 중간 Node를 생성한다.

```text
N0 = NOT(A AND S_B)

N1 = NOT(B AND S)
```

최종 출력:

```text
Y = NOT(N0 AND N1)
```

따라서:

```text
Y = (A AND S_B) OR (B AND S)
```

가 된다.

---

# 14. NAND 기반 Schematic 구조

```text
                    ┌─────┐
S ─────────────────>│ INV │──────> S_B
                    └─────┘


A ────────────────┐
                  ▼
               ┌──────┐
S_B ──────────>│ NAND │──────> N0
               └──────┘


B ────────────────┐
                  ▼
               ┌──────┐
S ────────────>│ NAND │──────> N1
               └──────┘


N0 ───────────────┐
                  ▼
               ┌──────┐
N1 ───────────>│ NAND │──────> Y
               └──────┘
```

필요한 Gate는:

```text
INV   × 1
NAND2 × 3
```

이다.

---

# 15. NAND 기반 Port Mapping

각 Gate의 연결은 다음과 같다.

```text
INV0

IN  = S
OUT = S_B
```

```text
NAND0

A   = A
B   = S_B
OUT = N0
```

```text
NAND1

A   = B
B   = S
OUT = N1
```

```text
NAND2

A   = N0
B   = N1
OUT = Y
```

---

# 16. 권장 Cell Hierarchy

NAND 기반 2MUX를 사용할 경우:

```text
INV
NAND2
  │
  └────────┐
           ▼
          MUX2
```

구조를 권장한다.

이미 INV와 NAND2를 다른 회로에서 설계했다면 동일 Cell을 재사용할 수 있다.

---

# 17. CMOS NAND2 구조

Static CMOS NAND2의 구조는:

```text
PMOS → Parallel

NMOS → Series
```

이다.

```text
                    VDD
                     │
              ┌──────┴──────┐
             PMOS A        PMOS B
              │              │
              └──────┬───────┘
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

# 18. Transmission Gate 기반 2MUX

Full-Custom 설계에서는 2MUX를 **Transmission Gate(TG)**를 이용하여 매우 효율적으로 구현할 수도 있다.

기본 구조:

```text
              TG0
A ───────────[   ]─────┐
                       │
                       ├──────> Y
                       │
B ───────────[   ]─────┘
              TG1
```

Select 신호에 따라 하나의 Transmission Gate만 ON된다.

---

# 19. Transmission Gate 동작

A 경로의 TG:

```text
S = 0 → ON
S = 1 → OFF
```

B 경로의 TG:

```text
S = 0 → OFF
S = 1 → ON
```

따라서:

```text
S = 0 → A → Y

S = 1 → B → Y
```

가 된다.

---

# 20. Transmission Gate 제어

Transmission Gate는 PMOS와 NMOS를 병렬로 연결한다.

```text
         NMOS
IN ──────┤ ├────── OUT
    │
    └─────┤ ├──────
         PMOS
```

NMOS와 PMOS에는 Complementary Control Signal을 인가한다.

---

# 21. A 경로 TG

A를 선택하는 Transmission Gate는 `S = 0`일 때 ON되어야 한다.

따라서:

```text
NMOS Gate = S_B

PMOS Gate = S
```

로 구성할 수 있다.

즉:

```text
S = 0

S_B = 1

NMOS ON
PMOS ON

→ A 전달
```

이다.

---

# 22. B 경로 TG

B 경로는 `S = 1`일 때 ON되어야 한다.

따라서:

```text
NMOS Gate = S

PMOS Gate = S_B
```

로 구성한다.

즉:

```text
S = 1

S_B = 0

NMOS ON
PMOS ON

→ B 전달
```

이다.

---

# 23. Transmission Gate 기반 전체 구조

```text
                  ┌─────┐
S ───────────────>│ INV │──────> S_B
                  └─────┘


         S_B / S
A ────────[ TG0 ]────────┐
                         │
                         ├────────> Y
                         │
B ────────[ TG1 ]────────┘
          S / S_B
```

Control:

```text
TG0:

NMOS Gate = S_B
PMOS Gate = S
```

```text
TG1:

NMOS Gate = S
PMOS Gate = S_B
```

---

# 24. Transmission Gate 방식의 Transistor 수

Transmission Gate 하나는:

```text
NMOS = 1
PMOS = 1
```

총 2 Transistor이다.

TG 두 개:

```text
2 × 2 = 4 Transistors
```

Select Complement 생성을 위한 Inverter:

```text
2 Transistors
```

따라서 기본적인 TG 기반 MUX는:

```text
Total = 6 Transistors
```

정도로 구현할 수 있다.

출력 복원을 위한 Inverter를 추가하면 Transistor 수는 더 증가한다.

---

# 25. NAND 방식과 TG 방식 비교

| 항목 | NAND 기반 MUX | TG 기반 MUX |
|---|---|---|
| 구조 | INV + NAND3개 | INV + TG2개 |
| 기본 Transistor 수 | 약 14개 | 약 6개 |
| Logic Style | Static CMOS | Pass/Transmission |
| 구조 | 비교적 큼 | 단순 |
| Data Path | 여러 Gate 통과 | TG 하나 통과 |
| Full-Swing | 기본적으로 강함 | TG 사용 시 우수 |
| Layout | 규칙적 | 비교적 Compact |
| 설계 난이도 | 쉬움 | 약간 높음 |
| Select Complement | 필요 | 필요 |

※ Transistor 수는 사용한 구현 방식에 따라 달라질 수 있다.

---

# 26. 왜 Transmission Gate를 사용하는가

단일 NMOS Pass Transistor만 사용할 경우 Logic 1 전달 시 Threshold Voltage Drop 문제가 발생할 수 있다.

```text
NMOS:

Strong 0
Weak 1
```

반대로 PMOS는:

```text
Strong 1
Weak 0
```

의 특성을 가진다.

Transmission Gate에서는 PMOS와 NMOS를 병렬 사용하므로:

```text
NMOS + PMOS
```

가 서로의 약점을 보완한다.

따라서:

```text
Strong 0
Strong 1
```

을 모두 전달할 수 있다.

---

# 27. Cadence Virtuoso Schematic 설계 순서

NAND 기반 MUX를 기준으로:

```text
1. INV Cell 준비

2. NAND2 Cell 준비

3. MUX2 Cell 생성

4. INV × 1 배치

5. NAND2 × 3 배치

6. S → INV 연결

7. INV Output → S_B

8. A + S_B → NAND0

9. NAND0 Output → N0

10. B + S → NAND1

11. NAND1 Output → N1

12. N0 + N1 → NAND2

13. NAND2 Output → Y

14. A / B / S Input Pin 생성

15. Y Output Pin 생성

16. VDD / VSS 연결

17. Check and Save

18. Symbol 생성

19. Testbench 생성

20. Transient Simulation
```

---

# 28. Functional Simulation

MUX는 조합 논리 회로이므로 모든 Input Combination을 확인한다.

입력:

```text
A
B
S
```

총 경우의 수:

```text
2^3 = 8
```

이다.

---

# 29. 검증 Case

## Case 1

```text
A = 0
B = 1
S = 0
```

예상:

```text
Y = A = 0
```

---

## Case 2

```text
A = 1
B = 0
S = 0
```

예상:

```text
Y = A = 1
```

---

## Case 3

```text
A = 0
B = 1
S = 1
```

예상:

```text
Y = B = 1
```

---

## Case 4

```text
A = 1
B = 0
S = 1
```

예상:

```text
Y = B = 0
```

---

# 30. vpulse를 이용한 자동 검증

A, B, S의 Pulse Period를 서로 다르게 하면 8개의 모든 입력 조합을 자동으로 확인할 수 있다.

예를 들어 한 상태를 10 ns로 설정하면:

```text
A:

Delay       = 10 ns
Pulse Width = 10 ns
Period      = 20 ns
```

```text
B:

Delay       = 20 ns
Pulse Width = 20 ns
Period      = 40 ns
```

```text
S:

Delay       = 40 ns
Pulse Width = 40 ns
Period      = 80 ns
```

정도로 설정할 수 있다.

Simulation Time:

```text
80 ns 이상
```

으로 설정하면 전체 조합을 확인할 수 있다.

---

# 31. Propagation Delay

MUX에서는 Data Input뿐 아니라 Select Signal에 의한 Delay도 중요하다.

주요 Delay는 다음과 같다.

```text
A → Y

B → Y

S → Y
```

---

# 32. A-to-Y Delay

Select가 A를 선택하도록 고정되어 있을 때:

```text
S = 0
```

A를 변화시킨다.

```text
A 변화
 ↓
Y 변화
```

입력과 출력의:

```text
50% VDD Crossing
```

사이 시간을 측정한다.

---

# 33. B-to-Y Delay

다음 조건:

```text
S = 1
```

에서 B를 변화시킨다.

```text
B 변화
 ↓
Y 변화
```

Delay를 측정한다.

---

# 34. Select-to-Y Delay

A와 B를 서로 다른 값으로 설정한다.

예:

```text
A = 0
B = 1
```

그 상태에서:

```text
S : 0 → 1
```

로 변화시키면 출력도:

```text
Y : 0 → 1
```

로 변한다.

따라서:

```text
S → Y
```

Propagation Delay를 측정할 수 있다.

---

# 35. Critical Path

NAND 기반 MUX의 주요 Data Path는 다음과 같다.

A 입력 기준:

```text
A
↓
NAND0
↓
N0
↓
NAND2
↓
Y
```

B 입력 기준:

```text
B
↓
NAND1
↓
N1
↓
NAND2
↓
Y
```

하지만 Select 입력은 더 긴 경로를 가질 수 있다.

예:

```text
S
↓
INV
↓
S_B
↓
NAND0
↓
N0
↓
NAND2
↓
Y
```

따라서 NAND 기반 2MUX에서는 구조적으로:

```text
S → INV → NAND → NAND → Y
```

경로가 Critical Path 후보가 될 수 있다.

실제 Critical Path는 Transistor Sizing과 Layout Parasitic을 포함한 Simulation을 통해 확인해야 한다.

---

# 36. Layout 설계 방향

NAND 기반 MUX의 권장 Data Flow는:

```text
S → INV
      ↓
     S_B

A → NAND0 ─┐
           ├→ NAND2 → Y
B → NAND1 ─┘
```

이를 기준으로 Layout을 배치하면 Routing이 단순해진다.

---

# 37. 권장 Floorplan

```text
VDD ─────────────────────────────────────

             ┌───────┐
S ──────────>│  INV  │───> S_B
             └───────┘


A ───────>┌────────┐
S_B ─────>│ NAND0  │──── N0 ───┐
          └────────┘             │
                                 ▼
                              ┌────────┐
                              │ NAND2  │────> Y
                              └────────┘
                                 ▲
B ───────>┌────────┐             │
S ───────>│ NAND1  │──── N1 ────┘
          └────────┘


VSS ─────────────────────────────────────
```

---

# 38. Layout 주요 고려사항

## Select Routing

`S`는:

```text
INV
NAND1
```

을 구동하고 `S_B`는:

```text
NAND0
```

을 구동한다.

Select 신호는 MUX의 선택 속도에 직접 영향을 주므로 불필요하게 긴 배선을 피한다.

---

## Internal Node Routing

다음 Node를 짧게 연결한다.

```text
N0 → Final NAND

N1 → Final NAND
```

---

## Output Placement

최종 NAND Gate를 Output Pin `Y` 가까이에 배치하면 출력 배선을 줄일 수 있다.

---

# 39. DRC

Layout 완료 후 다음 항목을 확인한다.

```text
Active Width / Spacing

Poly Width / Spacing

Metal Width / Spacing

Contact Enclosure

Via Enclosure

N-Well Rule

Implant Rule
```

정확한 Rule 값은 사용하는 PDK의 Design Rule Manual을 따른다.

---

# 40. LVS

NAND 기반 MUX에서 다음 Mapping을 확인한다.

```text
S → INV → S_B

A + S_B → NAND0 → N0

B + S → NAND1 → N1

N0 + N1 → NAND2 → Y
```

주요 LVS 오류 후보:

```text
A / B 연결 반대

S / S_B 연결 반대

N0 / N1 Label 오류

Y Pin 오류

VDD / VSS 누락
```

---

# 41. Post-Layout Simulation

PEX 후 Extracted View에서 동일한 Simulation을 수행한다.

비교 항목:

```text
A-to-Y Delay

B-to-Y Delay

S-to-Y Delay

Rise Time

Fall Time

Average Power
```

Schematic과 Post-Layout Simulation 결과를 비교하여 Interconnect Parasitic 영향을 분석한다.

---

# 42. MUX의 활용

2:1 MUX는 다양한 Digital Circuit에서 사용된다.

```text
Data Selector

ALU

Register Input Selection

Feedback Selection

Counter

Shift Register

Processor Datapath

MAC

FSM
```

등에서 매우 자주 사용된다.

---

# 43. MAC에서의 활용 가능성

MAC 구조에서 MUX는 초기값과 Feedback 값을 선택하는 데 사용할 수 있다.

예를 들어:

```text
              ┌────────── 0
              │
              ▼
           ┌─────┐
ACC_Q ────>│ MUX │────> RCA Input
           └─────┘
              ▲
              │
           RESET /
           CONTROL
```

초기 연산에서:

```text
MUX → 0 선택
```

이후 Accumulation에서는:

```text
MUX → ACC_Q 선택
```

하도록 구성할 수 있다.

또는 Enable이 있는 Register 구조에서도 MUX를 이용해:

```text
Enable = 1 → 새로운 SUM 저장

Enable = 0 → 기존 ACC 값 유지
```

하도록 만들 수 있다.

---

# 44. 장점

- 구조가 단순함
- 두 Data 중 하나를 선택 가능
- Digital Datapath의 기본 Building Block
- NAND 기반으로 쉽게 구현 가능
- Transmission Gate 기반으로 Compact하게 구현 가능
- Register / ALU / MAC 등에 쉽게 적용 가능

---

# 45. 단점

- MUX 단계가 증가하면 Propagation Delay 증가
- Select 신호의 Fan-Out이 커질 수 있음
- 여러 MUX가 직렬 연결되면 Critical Path가 길어짐
- TG 방식에서는 Complementary Select Signal이 필요함

---

# 46. NAND 기반과 TG 기반 선택

Full-Custom 프로젝트에서 구조를 명확하게 분석하고 기존 NAND2 Cell을 재사용하려면:

```text
NAND 기반 2MUX
```

가 설계와 LVS 측면에서 편리하다.

반면 Area 및 Data Path Delay를 줄이고 싶다면:

```text
Transmission Gate 기반 2MUX
```

를 비교 대상으로 사용할 수 있다.

---

# 47. 결론

2:1 Multiplexer는 Select 입력 `S`에 따라 두 개의 Data Input `A`, `B` 중 하나를 출력 `Y`로 전달하는 대표적인 조합 논리 회로이다.

기본 동작은:

```text
S = 0 → Y = A

S = 1 → Y = B
```

이며 논리식은:

```text
Y = (A AND NOT(S)) OR (B AND S)
```

이다.

NAND Gate 기반 구현에서는:

```text
S → INV → S_B

A + S_B → NAND → N0

B + S → NAND → N1

N0 + N1 → NAND → Y
```

구조를 사용할 수 있다.

Full-Custom 관점에서는 NAND 기반 구조와 Transmission Gate 기반 구조를 모두 구현할 수 있으며, Delay / Area / Power 측면에서 비교할 수 있다.

특히 이후 Register, ALU 및 MAC 설계에서 2MUX는 Data Selection과 Feedback Control에 활용할 수 있는 중요한 기본 Cell이다.
