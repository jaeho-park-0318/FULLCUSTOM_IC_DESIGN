# Gated D Latch 설계 보고서

## 1. 설계 개요

### 1.1 설계명

**Gated D Latch**

### 1.2 설계 목적

Gated D Latch는 하나의 데이터 입력 `D`와 Enable 입력 `E`를 이용하여 1-bit 데이터를 저장하는 **Level-Sensitive Storage Element**이다.

Gated SR Latch에서는 `S`와 `R`이 동시에 활성화될 경우 금지 상태가 발생할 수 있다.

Gated D Latch는:

```text
S = D
R = NOT(D)
```

관계를 사용하여 Set과 Reset 입력이 동시에 활성화되지 않도록 만들어 Gated SR Latch의 금지 상태 문제를 제거한다.

본 설계의 목표는 다음과 같다.

- Gated D Latch의 데이터 저장 원리 이해
- Transparent / Hold 상태 이해
- Gated SR Latch와의 구조적 관계 이해
- NAND 및 Inverter 기반 Schematic 설계
- Cadence Virtuoso Transient Simulation
- Full-Custom Layout
- DRC / LVS / PEX 검증
- 향후 D Flip-Flop 및 Register 설계를 위한 기본 Storage Cell 확보

---

# 2. 기본 동작

Gated D Latch의 입력은 다음과 같다.

### Input

```text
D : Data
E : Enable
```

### Output

```text
Q
QB
```

기본 동작은:

```text
E = 1 → Q = D

E = 0 → Q(next) = Q(previous)
```

이다.

즉 Enable이 활성화되면 D가 Q로 전달되고 Enable이 비활성화되면 이전 Q 값을 유지한다.

---

# 3. Level-Sensitive 동작

D Latch는 Flip-Flop과 달리 **Edge-Triggered가 아니라 Level-Sensitive**이다.

## Enable = 1

```text
Q follows D
```

즉 Enable이 HIGH인 동안 D의 변화가 Q로 전달된다.

이를 **Transparent 상태**라고 한다.

## Enable = 0

입력 경로가 차단되고 기존 상태를 저장한다.

```text
Q(next) = Q(previous)
```

이를 **Hold 상태**라고 한다.

---

# 4. Truth Table

| E | D | Q(next) | 동작 |
|---:|---:|---|---|
| 0 | X | Q(previous) | Hold |
| 1 | 0 | 0 | Transparent / Reset |
| 1 | 1 | 1 | Transparent / Set |

`X`는 Don't Care를 의미한다.

---

# 5. Gated SR Latch에서 D Latch로 변환

Gated SR Latch에는 다음 입력이 존재한다.

```text
S
R
E
```

D Latch에서는:

```text
S = D

R = NOT(D)
```

로 설정한다.

따라서:

```text
D ───────────────→ S path

D ── INV ── D_B ─→ R path
```

이다.

여기서:

```text
D_B = NOT(D)
```

이다.

---

# 6. NAND 기반 입력식

NAND 기반 Gated D Latch의 내부 SR Latch 입력은 다음과 같다.

```text
S_B = NOT(D AND E)

R_B = NOT(D_B AND E)
```

여기서:

```text
D_B = NOT(D)
```

이다.

---

# 7. 전체 Schematic 구조

```text
                       Enable Gating                     SR Latch

D ─────────────────┐
                   ├── NAND ───── S_B ──────────┐
E ─────────────────┘                             │
                                                  ▼
                                             ┌─────────┐
                                       QB ──>│ NAND    │────> Q
                                             └─────────┘
                                                  ▲
                                                  │
                                                  │ Feedback
                                                  ▼
                                             ┌─────────┐
                                        Q ──>│ NAND    │────> QB
                                             └─────────┘
                                                  ▲
                                                  │
D ── INV ── D_B ──┐                              │
                   ├── NAND ───── R_B ───────────┘
E ─────────────────┘
```

필요한 기본 Gate는:

```text
INV   × 1
NAND2 × 4
```

이다.

---

# 8. Enable = 0 동작

`E = 0`이면:

```text
S_B = NOT(D AND 0) = 1

R_B = NOT(D_B AND 0) = 1
```

따라서 내부 NAND SR Latch는:

```text
S_B = 1
R_B = 1
```

인 Hold 상태가 된다.

결과:

```text
Q(next) = Q(previous)
```

이다.

즉 D가 변하더라도 출력 Q는 변화하지 않는다.

---

# 9. Enable = 1, D = 1

다음 조건:

```text
E = 1
D = 1
```

이면:

```text
D_B = 0
```

따라서:

```text
S_B = NOT(1 AND 1) = 0

R_B = NOT(0 AND 1) = 1
```

내부 SR Latch가 Set된다.

결과:

```text
Q  = 1
QB = 0
```

이다.

---

# 10. Enable = 1, D = 0

다음 조건:

```text
E = 1
D = 0
```

이면:

```text
D_B = 1
```

따라서:

```text
S_B = NOT(0 AND 1) = 1

R_B = NOT(1 AND 1) = 0
```

내부 SR Latch가 Reset된다.

결과:

```text
Q  = 0
QB = 1
```

이다.

---

# 11. 금지 상태가 없는 이유

Gated SR Latch에서는:

```text
S = 1
R = 1
E = 1
```

상태가 가능하며 이 경우 금지 상태가 발생한다.

하지만 Gated D Latch에서는:

```text
S = D

R = NOT(D)
```

이므로 D가 0 또는 1 중 어느 값이더라도 S와 R이 동시에 1이 될 수 없다.

### D = 0

```text
S = 0
R = 1
```

### D = 1

```text
S = 1
R = 0
```

따라서 Gated D Latch에서는 SR Latch의 금지 입력 조합을 구조적으로 제거할 수 있다.

---

# 12. Port Mapping

## Input

```text
D
E
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
          ┌────────────────┐
D ───────>│                │──────> Q
E ───────>│ GATED_D_LATCH  │
          │                │──────> QB
          └────────────────┘
```

---

# 13. 내부 Net Label

권장 내부 Net Label은 다음과 같다.

```text
D_B
S_B
R_B
Q
QB
```

정확한 Mapping:

```text
D → INV → D_B

D + E → NAND → S_B

D_B + E → NAND → R_B

S_B + QB → NAND → Q

R_B + Q → NAND → QB
```

이 Naming을 Schematic과 Layout에서 동일하게 유지하는 것을 권장한다.

---

# 14. Cell Hierarchy

권장 Virtuoso Cell 구조:

```text
INV
NAND2
  │
  └────────┐
           ▼
    GATED_D_LATCH
```

기본 Cell인 INV와 NAND2를 각각 Transistor-Level로 설계하고 Symbol로 생성한 뒤 상위 D Latch에서 사용한다.

---

# 15. CMOS Inverter

D의 반전 신호 `D_B`를 생성한다.

```text
          VDD
           │
          PMOS
           │
D ─────────┤────── D_B
           │
          NMOS
           │
          VSS
```

논리 관계:

```text
D_B = NOT(D)
```

이다.

---

# 16. CMOS NAND2

2-input NAND Gate는 다음 구조로 구성한다.

### Pull-Up Network

PMOS 두 개 병렬

### Pull-Down Network

NMOS 두 개 직렬

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

# 17. Cadence Virtuoso Schematic 설계 순서

```text
1. INV Cell 설계
2. INV Functional Simulation
3. INV Symbol 생성

4. NAND2 Cell 설계
5. NAND2 Functional Simulation
6. NAND2 Symbol 생성

7. GATED_D_LATCH Cell 생성
8. INV × 1 배치
9. NAND2 × 4 배치

10. D → INV → D_B 연결

11. D + E → NAND_S 연결
12. NAND_S Output → S_B Label

13. D_B + E → NAND_R 연결
14. NAND_R Output → R_B Label

15. S_B + QB → NAND_Q
16. NAND_Q Output → Q

17. R_B + Q → NAND_QB
18. NAND_QB Output → QB

19. Q와 QB Cross-Coupled Feedback 연결

20. D / E Input Pin 생성
21. Q / QB Output Pin 생성
22. VDD / VSS 연결

23. Check and Save
24. Symbol 생성
25. Testbench 구성
26. Transient Simulation
```

---

# 18. Testbench 구성

D Latch는 Transient Simulation으로 확인하는 것이 적합하다.

`D`와 `E`에 각각 `vpulse`를 사용한다.

두 입력의 주기를 다르게 설정하면 한 번의 Simulation에서 Transparent와 Hold 상태를 모두 확인할 수 있다.

예:

```text
D : 0 → 1 → 0 → 1 → ...

E : 0 → 1 → 0 → 1 → ...
```

---

# 19. Simulation 검증 Case

## Case 1

```text
E = 1
D = 0
```

예상:

```text
Q  = 0
QB = 1
```

---

## Case 2

```text
E = 1
D = 1
```

예상:

```text
Q  = 1
QB = 0
```

---

## Case 3

```text
E = 0
```

D가 변하더라도:

```text
Q(next) = Q(previous)
```

이어야 한다.

---

# 20. Waveform에서 확인할 핵심

D Latch Simulation에서는 다음 관계를 확인해야 한다.

```text
E = 1 → Q follows D

E = 0 → Q holds previous value
```

개념적인 Waveform:

```text
Time ─────────────────────────────────────────>

E      ______────────────______────────────____

D      ____────____────────────____────________

Q      ________────────────────____────________
             ↑                  ↑
        Transparent            Hold
```

실제 Waveform에서는 Gate Propagation Delay 때문에 D와 Q의 Edge가 완전히 동시에 발생하지 않는다.

---

# 21. 초기 상태 설정

Cross-Coupled NAND 구조에서는 Simulation 시작 시 Q와 QB의 초기 상태가 결정되지 않을 수 있다.

따라서 Simulation 초기에 Enable을 활성화한 상태에서 D 값을 일정 시간 유지하는 것을 권장한다.

예:

```text
0 ns ~ 10 ns

E = 1
D = 0
```

그러면:

```text
Q  = 0
QB = 1
```

로 초기화할 수 있다.

---

# 22. Timing Characteristic

Gated D Latch에서는 다음 Timing을 확인할 수 있다.

## D-to-Q Delay

Enable이 HIGH인 상태에서 D가 변화한 후 Q가 변화하기까지의 시간.

```text
D → Q Propagation Delay
```

---

## Enable-to-Q Delay

D가 고정된 상태에서 E가 활성화된 후 Q가 D 값을 반영하기까지의 시간.

```text
E → Q Propagation Delay
```

---

## Setup Time

Latch가 Hold 상태로 들어가기 전에 D가 안정되어 있어야 하는 최소 시간.

---

## Hold Time

Latch가 닫힌 직후 D가 일정 시간 유지되어야 하는 최소 시간.

---

# 23. D Latch와 D Flip-Flop 차이

## D Latch

**Level-Sensitive**

```text
E = 1인 동안 계속 D를 받아들임
```

즉:

```text
E HIGH → Transparent
```

이다.

## D Flip-Flop

**Edge-Triggered**

```text
Clock의 Rising Edge 또는 Falling Edge 순간에만 D 저장
```

따라서 Latch와 Flip-Flop은 동일하지 않다.

---

# 24. Master-Slave D Flip-Flop으로 확장

D Latch 두 개를 직렬 연결하면 Master-Slave D Flip-Flop을 만들 수 있다.

```text
D
│
▼
┌─────────────────┐
│ Master D Latch  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Slave D Latch   │
└────────┬────────┘
         │
         ▼
         Q
```

Master와 Slave에 서로 반대 Clock Phase를 적용하면 Edge-Triggered 동작을 구현할 수 있다.

따라서:

```text
Gated D Latch
      ↓
D Flip-Flop
      ↓
N-bit Register
      ↓
Accumulator
      ↓
MAC
```

으로 확장할 수 있다.

---

# 25. Layout 설계 방향

권장 Floorplan:

```text
VDD ─────────────────────────────────────

             ┌───────┐
D ──────────>│  INV  │────> D_B
             └───────┘

┌──────────┐               ┌──────────┐
│ NAND_S   │──────────────>│ NAND_Q   │────> Q
└──────────┘               └──────────┘
                                ▲
                                │
                                │ Feedback
                                ▼
┌──────────┐               ┌──────────┐
│ NAND_R   │──────────────>│ NAND_QB  │────> QB
└──────────┘               └──────────┘

VSS ─────────────────────────────────────
```

---

# 26. Feedback Routing

가장 중요한 내부 배선 중 하나이다.

```text
Q  → NAND_QB

QB → NAND_Q
```

두 경로를 가능하면 짧게 만든다.

Feedback Node의 Parasitic Capacitance가 커지면 State Transition Delay가 증가할 수 있다.

---

# 27. D / D_B Placement

Inverter는 Input Gating NAND 근처에 배치하는 것이 좋다.

```text
D ── INV ── D_B
```

이후:

```text
D   → NAND_S
D_B → NAND_R
```

로 분기한다.

---

# 28. Enable Routing

Enable `E`는 두 개의 NAND Gate를 동시에 구동한다.

```text
          ┌──> NAND_S
E ────────┤
          └──> NAND_R
```

즉 E의 Fan-Out은 기본적으로 2이다.

두 NAND에 대한 E 배선 길이를 지나치게 다르게 만들지 않는 것이 좋다.

---

# 29. Power Rail

하위 Cell의 VDD / VSS Rail 높이와 방향을 통일한다.

```text
VDD ─────────────────────────────

         PMOS Region

         NMOS Region

VSS ─────────────────────────────
```

이렇게 하면 Hierarchical Layout에서 Cell을 정렬하기 쉽다.

---

# 30. DRC

Layout 완료 후 다음 Design Rule을 검사한다.

```text
Active Width
Active Spacing
Poly Width
Poly Spacing
Metal Width
Metal Spacing
Via Enclosure
Contact Enclosure
N-Well
Implant
```

정확한 Minimum Rule은 사용하는 PDK의 Design Rule Manual을 따른다.

---

# 31. LVS

LVS에서는 다음 Mapping을 집중적으로 확인한다.

```text
D → INV → D_B

D + E → NAND_S → S_B

D_B + E → NAND_R → R_B

S_B + QB → NAND_Q → Q

R_B + Q → NAND_QB → QB
```

주요 오류:

```text
D / D_B 반대 연결

Q / QB 반대 연결

S_B / R_B Label 오류

Feedback 누락

VDD / VSS 연결 누락

Pin Name 불일치
```

---

# 32. Post-Layout Simulation

PEX 이후 Extracted View를 이용해 동일한 Transient Simulation을 다시 수행한다.

Schematic과 비교할 항목:

```text
D-to-Q Delay
E-to-Q Delay
Rise Time
Fall Time
Power Consumption
Feedback Stability
```

Layout의 Interconnect 및 Device Parasitic 때문에 일반적으로 Post-Layout Delay가 증가한다.

---

# 33. Gated SR Latch와 비교

| 항목 | Gated SR Latch | Gated D Latch |
|---|---|---|
| 입력 | S, R | D |
| Enable | E | E |
| Storage | 1 bit | 1 bit |
| 금지 상태 | 존재 | 없음 |
| 기본 구조 | NAND 4개 | NAND 4개 + INV |
| Data Input | 2개 | 1개 |
| 일반 Data 저장 | 불편 | 적합 |
| DFF 확장 | 가능 | 매우 적합 |

Gated D Latch는 Gated SR Latch의 금지 상태를 제거하고 일반적인 Data Storage에 사용하기 쉽도록 개선된 구조라고 볼 수 있다.

---

# 34. 장점

- 금지 입력 상태가 없음
- Data Input이 D 하나뿐임
- 구조가 비교적 단순함
- 1-bit Data Storage에 적합
- D Flip-Flop으로 쉽게 확장 가능
- Register 설계에 활용 가능
- Full-Custom 순차회로 설계 학습에 적합

---

# 35. 단점

- Level-Sensitive 구조
- Enable이 HIGH인 동안 D 변화가 Q에 전달됨
- Timing 관리가 필요한 회로에서는 Race 문제가 발생할 수 있음
- 완전한 Edge-Triggered 동작이 필요한 경우 DFF가 필요함

---

# 36. 향후 MAC 설계와의 관계

현재 설계 중인 MAC에서는 Multiplier와 RCA의 연산 결과를 저장하기 위한 Accumulator Register가 필요하다.

구조는:

```text
MUL4
 ↓
RCA
 ↓
Register
 ↓
ACC
```

이다.

Register는 여러 개의 D Flip-Flop으로 구성할 수 있으며, D Flip-Flop은 Gated D Latch를 기반으로 설계할 수 있다.

따라서:

```text
Gated D Latch
      ↓
D Flip-Flop
      ↓
10-bit Register
      ↓
MAC Accumulator
```

라는 설계 흐름으로 연결된다.

---

# 37. 결론

Gated D Latch는 Enable이 활성화된 동안 입력 D를 출력 Q로 전달하고, Enable이 비활성화되면 기존 Q 값을 저장하는 Level-Sensitive Storage Element이다.

Gated SR Latch에서:

```text
S = D
R = NOT(D)
```

관계를 적용함으로써 SR Latch의 금지 상태를 구조적으로 제거할 수 있다.

Full-Custom 설계에서는 NAND2 및 Inverter의 재사용, Cross-Coupled Feedback Routing, Enable Fan-Out, Transistor Sizing, D-to-Q 및 E-to-Q Delay, DRC/LVS, Post-Layout Parasitic 분석 등이 주요 설계 포인트이다.

또한 Gated D Latch는 이후 Master-Slave D Flip-Flop, N-bit Register 및 MAC Accumulator 설계로 확장하기 위한 핵심 기본 회로이다.
