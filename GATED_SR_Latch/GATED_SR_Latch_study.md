# Gated SR Latch 설계 보고서

## 1. 설계 개요

### 1.1 설계명

**Gated SR Latch**

### 1.2 설계 목적

Gated SR Latch는 기본 SR Latch에 **Enable(E)** 신호를 추가하여 특정 시간에만 `S(Set)`와 `R(Reset)` 입력이 내부 Latch에 전달되도록 만든 **1-bit 저장 회로**이다.

기본 SR Latch는 입력 변화에 따라 즉시 상태가 변할 수 있지만, Gated SR Latch에서는 `E`가 활성화된 경우에만 상태 변경이 가능하다.

따라서 Gated SR Latch는 순차 논리 회로와 Level-Sensitive Storage Element의 기본 구조를 이해하는 데 중요한 회로이다.

본 설계의 목표는 다음과 같다.

- SR Latch의 기억 원리 이해
- Enable 신호의 역할 이해
- NAND Gate 기반 Gated SR Latch 구성
- Set / Reset / Hold 동작 검증
- 금지 상태(Invalid State) 확인
- Cadence Virtuoso 기반 Transient Simulation
- Full-Custom Layout 및 DRC/LVS 검증

---

# 2. SR Latch 기본 개념

SR Latch는 두 개의 입력과 두 개의 출력을 가진다.

### Input

- `S` : Set
- `R` : Reset

### Output

- `Q`
- `QB`

정상적인 상태에서는 다음 관계를 만족한다.

`QB = NOT(Q)`

즉 `Q`와 `QB`는 서로 반대 값을 갖는다.

SR Latch는 출력 신호를 다시 입력으로 Feedback하는 **Cross-Coupled 구조**를 이용하여 이전 상태를 기억한다.

---

# 3. NAND 기반 SR Latch

NAND Gate 두 개를 Cross-Coupled 형태로 연결하면 SR Latch를 구성할 수 있다.

```text
              ┌─────────┐
S_B ─────────>│ NAND    │──────> Q
              │         │
       QB ───>│         │
              └─────────┘
                   ▲
                   │
                   │ Feedback
                   ▼
              ┌─────────┐
R_B ─────────>│ NAND    │──────> QB
              │         │
        Q ───>│         │
              └─────────┘
```

NAND 기반 SR Latch에서는 입력이 **Active-Low**이다.

따라서:

```text
S_B = 0 → Set
R_B = 0 → Reset

S_B = 1
R_B = 1 → Hold
```

---

# 4. Gated SR Latch 구조

기본 SR Latch 앞에 Enable을 이용한 NAND Gate를 추가한다.

입력단의 NAND 출력은 다음과 같다.

```text
S_B = NOT(S AND E)

R_B = NOT(R AND E)
```

전체 구조는 다음과 같다.

```text
                Enable Logic                   SR Latch

S ───────┐
         ├── NAND ───── S_B ───────┐
E ───────┘                          │
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
R ───────┐                          │
         ├── NAND ───── R_B ───────┘
E ───────┘
```

따라서 Gated SR Latch는 기본적으로

```text
NAND2 × 4
```

로 구현할 수 있다.

---

# 5. 동작 원리

## 5.1 Enable = 0

`E = 0`이면 입력 NAND의 출력은 다음과 같다.

```text
S_B = NOT(S AND 0) = 1

R_B = NOT(R AND 0) = 1
```

따라서 내부 SR Latch의 입력은

```text
S_B = 1
R_B = 1
```

이 되어 Hold 상태가 된다.

즉:

```text
E = 0 → Q(next) = Q(previous)
```

S와 R이 변해도 출력 Q는 기존 상태를 유지한다.

---

## 5.2 Enable = 1

`E = 1`이면:

```text
S_B = NOT(S)

R_B = NOT(R)
```

이므로 S와 R의 상태가 내부 SR Latch에 전달된다.

---

# 6. Truth Table

| E | S | R | Q(next) | 동작 |
|---:|---:|---:|---|---|
| 0 | X | X | Q(previous) | Hold |
| 1 | 0 | 0 | Q(previous) | Hold |
| 1 | 1 | 0 | 1 | Set |
| 1 | 0 | 1 | 0 | Reset |
| 1 | 1 | 1 | Invalid | 금지 상태 |

`X`는 Don't Care를 의미한다.

---

# 7. Set 동작

다음 조건을 적용한다.

```text
E = 1
S = 1
R = 0
```

입력 NAND 출력:

```text
S_B = 0
R_B = 1
```

따라서 출력은:

```text
Q  = 1
QB = 0
```

이 된다.

---

# 8. Reset 동작

다음 조건을 적용한다.

```text
E = 1
S = 0
R = 1
```

입력 NAND 출력:

```text
S_B = 1
R_B = 0
```

따라서:

```text
Q  = 0
QB = 1
```

이 된다.

---

# 9. Hold 동작

다음 두 경우에는 기존 출력 상태가 유지된다.

### Enable이 0인 경우

```text
E = 0
S = X
R = X
```

### Enable은 1이지만 S와 R이 모두 0인 경우

```text
E = 1
S = 0
R = 0
```

두 경우 모두:

```text
Q(next) = Q(previous)
```

이다.

---

# 10. 금지 상태

다음 입력은 피해야 한다.

```text
E = 1
S = 1
R = 1
```

이 경우:

```text
S_B = 0
R_B = 0
```

NAND 기반 SR Latch에서 두 입력이 동시에 0이 되면:

```text
Q  = 1
QB = 1
```

이 될 수 있다.

이는 정상적인 상보 관계:

```text
QB = NOT(Q)
```

를 위반한다.

또한 S와 R이 동시에 비활성화될 경우 최종 상태가 Gate Delay나 Transistor 특성 차이에 의해 결정될 수 있다.

따라서:

```text
E = 1일 때 S = R = 1 상태는 금지
```

이다.

---

# 11. Port Mapping

## Input Pin

```text
S
R
E
```

## Output Pin

```text
Q
QB
```

## Power Pin

```text
VDD
VSS
```

권장 Symbol:

```text
          ┌────────────────┐
S ───────>│                │──────> Q
R ───────>│ GATED_SR_LATCH │
E ───────>│                │──────> QB
          └────────────────┘
```

---

# 12. 내부 Net Label

Schematic의 가독성과 Layout/LVS 검증을 위해 다음 Net Label을 사용하는 것을 권장한다.

```text
S_B
R_B
Q
QB
```

연결 관계는:

```text
S + E → NAND → S_B

R + E → NAND → R_B

S_B + QB → NAND → Q

R_B + Q → NAND → QB
```

이다.

---

# 13. Cell Hierarchy

권장 Virtuoso Cell 구조:

```text
NAND2
  ↓
GATED_SR_LATCH
```

NAND2를 Transistor-Level에서 먼저 설계하고 Symbol을 생성한 뒤 Gated SR Latch에서 4개를 반복 사용한다.

---

# 14. CMOS NAND2 구조

2-input CMOS NAND는 다음 구조를 갖는다.

### Pull-Up Network

PMOS 두 개를 병렬 연결한다.

### Pull-Down Network

NMOS 두 개를 직렬 연결한다.

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

Gated SR Latch에서 NAND Gate는 반복적으로 사용되므로 NAND2의 Delay와 Transistor Sizing이 전체 회로의 성능에 영향을 준다.

---

# 15. Cadence Virtuoso Schematic 설계 순서

```text
1. NAND2 Cell 생성
2. NAND2 Transistor-Level Schematic 설계
3. NAND2 Simulation
4. NAND2 Symbol 생성
5. GATED_SR_LATCH Cell 생성
6. NAND2 Symbol 4개 배치
7. S와 E를 첫 번째 NAND에 연결
8. R과 E를 두 번째 NAND에 연결
9. 두 출력 NAND를 Cross-Coupled 형태로 연결
10. S_B / R_B Label 지정
11. Q / QB Output Pin 생성
12. S / R / E Input Pin 생성
13. VDD / VSS 연결
14. Check and Save
15. Symbol 생성
16. Testbench 구성
17. Transient Simulation
```

---

# 16. Simulation 계획

Gated SR Latch는 **Transient Analysis**로 검증한다.

입력은 `analogLib/vpulse` 등을 사용할 수 있다.

확인해야 할 주요 Case는 다음과 같다.

### Case 1: Initial Reset

```text
E = 1
S = 0
R = 1
```

예상:

```text
Q  = 0
QB = 1
```

### Case 2: Hold

```text
E = 0
```

S와 R을 변화시켜도 Q가 유지되어야 한다.

### Case 3: Set

```text
E = 1
S = 1
R = 0
```

예상:

```text
Q  = 1
QB = 0
```

### Case 4: Hold

```text
E = 1
S = 0
R = 0
```

이전 상태 유지

### Case 5: Reset

```text
E = 1
S = 0
R = 1
```

예상:

```text
Q  = 0
QB = 1
```

---

# 17. 초기 상태 문제

Cross-Coupled NAND Latch는 Feedback 구조이므로 Simulation 시작 시 초기 상태가 명확하게 결정되지 않을 수 있다.

이 경우 Q와 QB가 X 상태 또는 중간 전압에서 시작할 가능성이 있다.

따라서 Simulation 시작 구간에서 Reset 또는 Set 상태를 강제로 입력해주는 것이 좋다.

예:

```text
0 ns ~ 10 ns

E = 1
S = 0
R = 1
```

그러면:

```text
Q  = 0
QB = 1
```

로 초기화할 수 있다.

---

# 18. Propagation Delay

측정할 수 있는 주요 Delay는 다음과 같다.

### Set Delay

```text
S 변화 → Q : LOW → HIGH
```

### Reset Delay

```text
R 변화 → Q : HIGH → LOW
```

일반적으로 입력과 출력의 `50% VDD` Crossing Point를 기준으로 측정한다.

예:

```text
t_PLH = Q가 LOW에서 HIGH로 변하는 Propagation Delay

t_PHL = Q가 HIGH에서 LOW로 변하는 Propagation Delay
```

---

# 19. Layout 설계 방향

NAND2 Layout Cell을 먼저 완성하고 이를 4개 배치하는 Hierarchical Layout 방식을 권장한다.

개념적인 Floorplan:

```text
VDD ─────────────────────────────────

┌────────┐         ┌────────┐
│ NAND_S │────────>│ NAND_Q │────> Q
└────────┘         └────────┘
                       ▲
                       │
                       │ Feedback
                       ▼
┌────────┐         ┌─────────┐
│ NAND_R │────────>│ NAND_QB │────> QB
└────────┘         └─────────┘

VSS ─────────────────────────────────
```

---

# 20. Layout 주요 고려사항

## Feedback Routing

다음 Feedback 경로를 가능한 한 짧게 배치한다.

```text
Q  → NAND_QB
QB → NAND_Q
```

Feedback Wire가 길어지면 Parasitic R/C가 증가하여 Delay가 증가할 수 있다.

## Enable Routing

Enable `E`는 두 개의 Input NAND를 동시에 구동한다.

```text
E → NAND_S
E → NAND_R
```

따라서 두 경로의 배선 길이가 지나치게 달라지지 않도록 구성하는 것이 좋다.

## Power Rail

각 NAND Cell의 VDD/VSS 방향을 동일하게 만든다.

```text
VDD ─────────────────

Logic Cells

VSS ─────────────────
```

## Well / Substrate Contact

PMOS 영역에는 적절한 N-Well Contact를 배치하고 NMOS 영역에는 Substrate Contact를 배치한다.

---

# 21. DRC

Layout 완료 후 Design Rule Check를 수행한다.

주요 확인 항목:

```text
Active Width / Spacing
Poly Width / Spacing
Metal Width / Spacing
Contact Enclosure
Via Enclosure
N-Well Rule
Implant Rule
```

정확한 Rule 값은 사용 중인 PDK의 Design Rule Manual을 따른다.

---

# 22. LVS

LVS에서는 Schematic과 Layout의 연결 관계가 동일한지 확인한다.

특히 다음 Mapping을 확인한다.

```text
S + E → NAND_S → S_B

R + E → NAND_R → R_B

S_B + QB → NAND_Q → Q

R_B + Q → NAND_QB → QB
```

주요 오류 후보:

```text
Q / QB 반대 연결
S_B / R_B Label 오류
Feedback 연결 오류
VDD / VSS 누락
Pin Name Mismatch
```

---

# 23. Post-Layout Simulation

PEX(Parasitic Extraction)를 수행한 후 Extracted View를 이용해 다시 Transient Simulation을 수행한다.

Schematic Simulation과 비교할 항목:

```text
Set Delay
Reset Delay
Rise Time
Fall Time
Dynamic Power
Feedback Stability
```

Layout 이후 Parasitic R/C에 의해 Delay가 증가하는 것이 일반적이다.

---

# 24. 장점과 단점

## 장점

- 구조가 단순함
- NAND Gate 4개로 구현 가능
- Enable 기능 제공
- 1-bit 상태 저장 가능
- Latch / Flip-Flop 설계의 기본 구조

## 단점

- `S = R = 1` 금지 상태 존재
- 데이터 입력이 S와 R 두 개로 나뉨
- 일반적인 Data Storage에는 D Latch보다 사용이 불편함

---

# 25. 향후 확장

Gated SR Latch는 다음 회로의 기본 구조로 활용할 수 있다.

```text
Gated SR Latch
      ↓
Gated D Latch
      ↓
D Flip-Flop
      ↓
Register
      ↓
Counter / MAC / Sequential Circuit
```

---

# 26. 결론

Gated SR Latch는 기본 SR Latch에 Enable 신호를 추가하여 특정 조건에서만 상태 변경을 허용하는 Level-Sensitive Sequential Circuit이다.

NAND Gate 4개만으로 비교적 간단하게 구현할 수 있으며 Full-Custom 설계에서는 NAND2 Cell의 설계, Cross-Coupled Feedback 연결, 초기 상태 설정, Propagation Delay 및 Feedback Routing이 중요하다.

다만 `E = 1`, `S = 1`, `R = 1`인 경우 금지 상태가 존재한다.

이러한 문제를 구조적으로 제거하고 하나의 Data 입력으로 상태를 제어하도록 개선한 회로가 **Gated D Latch**이다.
