# CMOS Half Adder Full-Custom Layout 설계 보고서

## 1. 설계 개요

본 설계에서는 2개의 1-bit 입력을 더하여 `SUM`과 `CARRY`를 출력하는 **Half Adder**를 Full-Custom 방식으로 구현하였다.

Half Adder는 두 입력 `A`, `B`에 대해 다음과 같은 출력을 생성한다.

```text
SUM   = A XOR B
CARRY = A · B
```

따라서 기존에 설계하고 DRC/LVS 검증을 완료한 **XOR Gate와 AND Gate를 계층적으로 조합하여 Half Adder를 구성**할 수 있다.

전체 구조는 다음과 같다.

```text
A ─────┬──── XOR ───── SUM
       │
B ─────┘

A ─────┬──── AND ───── CARRY
       │
B ─────┘
```

즉,

```text
Half Adder = XOR + AND
```

의 구조로 이해할 수 있다.

---

## 2. Half Adder의 동작

Half Adder의 Truth Table은 다음과 같다.

| A | B | SUM | CARRY |
| - | - | --- | ----- |
| 0 | 0 | 0   | 0     |
| 0 | 1 | 1   | 0     |
| 1 | 0 | 1   | 0     |
| 1 | 1 | 0   | 1     |

두 입력이 서로 다르면 `SUM`이 HIGH가 되고, 두 입력이 모두 HIGH인 경우에는 `CARRY`가 HIGH가 된다.

즉,

```text
A = B
→ SUM = 0

A != B
→ SUM = 1
```

이며,

```text
A = 1 and B = 1
→ CARRY = 1
```

이다.

---

## 3. 논리식

Half Adder의 두 출력은 다음과 같이 표현된다.

```text
SUM = A XOR B
```

Boolean 형태로 쓰면

```text
SUM = (NOT A · B) + (A · NOT B)
```

이다.

Carry는 AND 연산으로 표현된다.

```text
CARRY = A · B
```

따라서 Half Adder는 XOR와 AND 두 개의 Logic Block을 동시에 이용하는 구조이다.

---

## 4. Hierarchical Design

본 Half Adder에서는 이미 설계한 XOR와 AND Cell을 하위 Block으로 사용하였다.

계층 구조는 다음과 같다.

```text
HALF_ADDER
├── XOR
│   └── XOR transistor network
│
└── AND
    ├── NAND
    └── Inverter
```

AND Cell 자체도 NAND와 Inverter의 조합이므로 전체 설계를 더 세부적으로 보면 다음과 같은 계층 구조가 된다.

```text
HALF_ADDER
├── XOR
│
└── AND
    ├── NAND
    └── Inverter
```

이처럼 검증된 하위 Logic Cell을 반복적으로 재사용하면 회로 규모가 커져도 Layout 설계를 체계적으로 관리할 수 있다.

---

## 5. Layout 구성

Half Adder Layout에서는 XOR Cell과 AND Cell을 배치하고 두 입력 `A`, `B`를 각각 두 Cell에 동시에 전달한다.

구조적으로는 다음과 같다.

```text
                ┌─────────┐
A ────────┬────▶│   XOR   │──── SUM
          │     └─────────┘
          │
          │     ┌─────────┐
          └────▶│   AND   │──── CARRY
                └─────────┘

                ┌─────────┐
B ────────┬────▶│   XOR   │
          │     └─────────┘
          │
          │     ┌─────────┐
          └────▶│   AND   │
                └─────────┘
```

Top-Level Pin은 일반적으로 다음과 같이 구성된다.

```text
A
B
SUM
CARRY
VDD
VSS
```

---

## 6. 입력 A/B Routing

Half Adder에서는 하나의 입력이 두 개의 하위 Cell에 동시에 전달된다.

즉,

```text
A → XOR input
A → AND input
```

그리고

```text
B → XOR input
B → AND input
```

으로 연결되어야 한다.

따라서 Layout에서는 입력 A와 B가 각각 두 Cell에 정확하게 연결되어 있는지 확인하는 것이 매우 중요하다.

잘못 연결되면 LVS에서 다음과 같은 문제가 발생할 수 있다.

```text
Bad Initial Net Binding
Open Instance Connection
Bad Matched Net
```

등의 오류가 나타날 수 있다.

실제로 계층적 Layout에서는 Pin 이름은 같더라도 특정 Instance의 입력까지 Metal이 실제로 이어져 있지 않으면 LVS에서는 Open으로 판단한다.

---

## 7. XOR와 AND Cell Placement

XOR와 AND는 동일한 두 입력을 공유하므로 두 Cell의 입력 Pin 위치를 고려하여 배치하면 Routing을 단순화할 수 있다.

좋은 배치의 목표는 다음과 같다.

```text
Shared Input Routing 최소화
Interconnect Length 최소화
Via 수 최소화
Cell 간 간격 최소화
```

예를 들어 XOR와 AND의 입력 Pin이 비슷한 방향에 위치하도록 배치하면 A와 B를 분기하여 두 Cell에 연결하기 쉬워진다.

반대로 Cell 배치가 좋지 않으면 입력 Routing이 서로 교차하고 상위 Metal Layer를 추가로 사용해야 할 수 있다.

따라서 Half Adder부터는 하위 Cell의 단순한 배치뿐만 아니라 **Pin 위치와 Routing 가능성을 고려한 Placement**가 중요해진다.

---

## 8. SUM Output

Half Adder의 `SUM`은 XOR의 출력이다.

```text
SUM = A XOR B
```

따라서 Layout에서는

```text
XOR VOUT → SUM
```

으로 연결하면 된다.

XOR Cell이 이미 검증되어 있다면 SUM path에서는 XOR 출력이 Top-Level SUM Pin까지 정상적으로 연결되어 있는지 확인하는 것이 핵심이다.

SUM은 이후 Full Adder에서는 다른 Half Adder의 입력으로 사용될 수 있으므로 상위 계층에서 Routing하기 편리한 위치에 Pin을 배치하는 것도 중요하다.

---

## 9. CARRY Output

Half Adder의 `CARRY`는 AND의 출력이다.

```text
CARRY = A · B
```

Layout에서는

```text
AND VOUT → CARRY
```

로 연결한다.

마찬가지로 CARRY는 이후 Full Adder에서 OR Gate의 입력으로 사용되므로 상위 Cell의 Routing을 고려한 Pin 위치를 선택하면 전체 Layout을 더 효율적으로 구성할 수 있다.

---

## 10. VDD / VSS Routing

XOR와 AND는 동일한 VDD와 VSS를 사용한다.

따라서 Half Adder 상위 Layout에서 두 하위 Cell의 Power Rail을 하나의 전원 Net으로 연결한다.

```text
════════════════════════ VDD
      │            │
     XOR          AND
      │            │
      │            │
════════════════════════ VSS
```

가능하다면 두 Cell의 VDD/VSS Rail이 같은 위치와 방향으로 정렬되도록 배치하면 전원 Routing을 보다 단순하게 만들 수 있다.

Schematic과 Layout에서 Pin 이름도 정확히 일치해야 한다.

```text
VDD ↔ VDD
VSS ↔ VSS
```

Pin이 존재하지 않거나 이름이 다르면 LVS에서 `Unbound Pin` 문제가 발생할 수 있다.

---

## 11. Metal Layer 사용에 대한 고찰

Half Adder에서는 단일 Gate보다 Routing이 복잡해지기 때문에 입력 A/B와 SUM/CARRY를 연결하는 과정에서 여러 Metal Layer가 필요할 수 있다.

하지만 Local Routing을 위해 필요 이상으로 높은 Metal Layer까지 사용하는 것은 효율적이지 않을 수 있다.

예를 들어

```text
M1
 │
Via1
 │
M2
 │
Via2
 │
M3
 │
Via3
 │
M4
```

와 같이 연결하면 여러 번의 Layer Transition이 발생한다.

이 경우

* Via 수 증가
* Via Resistance 증가
* Routing Complexity 증가
* DRC Rule 증가
* LVS Debugging 복잡도 증가
* 상위 Metal Routing Resource 사용
* 추가 Parasitic 발생 가능성

등의 문제가 생길 수 있다.

따라서 Half Adder 내부의 Local Signal은 가능한 범위에서 필요한 수준의 Metal만 이용하여 연결하는 것이 좋다.

핵심은

```text
Upper Metal 사용 금지 X
불필요한 Upper Metal 사용 최소화 O
```

이다.

---

## 12. Via 사용에 대한 고찰

Metal Layer를 바꿀 때마다 Via가 필요하다.

따라서 같은 Routing을 구현할 수 있다면 Via 개수가 적은 구조가 일반적으로 더 단순하다.

```text
Metal Layer Transition 증가
→ Via 증가
→ Resistance 증가 가능
→ Layout Complexity 증가
```

특히 작은 Logic Cell 내부에서는 상위 Metal까지 올라갔다 다시 내려오는 구조보다 가능한 짧고 단순한 연결이 유리하다.

다만 신호 교차를 해결하거나 Routing congestion을 줄이기 위해 필요한 경우에는 적절한 Metal Layer와 Via를 사용해야 한다.

---

## 13. DRC 검증

Half Adder Layout을 완성한 후에는 DRC를 수행하여 PDK에서 요구하는 제조 규칙을 만족하는지 확인한다.

주요 검사 항목은 다음과 같다.

* Minimum Width
* Minimum Spacing
* Minimum Area
* Metal Spacing
* Via Spacing
* Via Enclosure
* Well Rule
* Implant Rule

XOR와 AND Cell이 각각 DRC PASS 상태라도 상위 Half Adder Layout에서 새롭게 생성된 Metal과 Via는 별도의 DRC Error를 발생시킬 수 있다.

따라서 Top-Level Half Adder에서도 반드시 DRC를 다시 수행해야 한다.

```text
Final Layout → DRC PASS
```

상태를 확인해야 한다.

---

## 14. LVS 검증

LVS에서는 Schematic과 Layout의 전기적인 Connectivity를 비교한다.

Half Adder에서 핵심적으로 확인해야 하는 연결은 다음과 같다.

```text
A → XOR input
A → AND input

B → XOR input
B → AND input

XOR VOUT → SUM
AND VOUT → CARRY
```

그리고 전원은

```text
XOR VDD = AND VDD = VDD
XOR VSS = AND VSS = VSS
```

로 연결되어야 한다.

하위 XOR와 AND가 각각 LVS PASS 상태더라도 상위 Layout에서 입력 A 또는 B가 한 Cell에만 연결되어 있으면 LVS Error가 발생한다.

예를 들어 Schematic이

```text
A → XOR
A → AND
```

로 되어 있는데 Layout이

```text
A → XOR
```

까지만 연결되어 있다면 Open Connection으로 판단된다.

따라서 계층적 Layout에서는 **Pin 이름뿐만 아니라 실제 Instance Terminal까지 연결되어 있는지** 확인하는 것이 중요하다.

---

## 15. Half Adder에서 발생하기 쉬운 LVS 오류

Half Adder처럼 여러 하위 Cell이 동일한 입력을 공유하는 회로에서는 다음 오류가 발생하기 쉽다.

### Open Connection

```text
A → XOR       정상
A → AND       연결 안 됨
```

이런 경우 하나의 Schematic Net이 Layout에서 두 개의 Net으로 분리된다.

### Short Connection

```text
A ─┐
   ├── 잘못 연결
B ─┘
```

서로 다른 입력이 같은 Metal로 연결되면 Short가 발생한다.

### Pin Mismatch

```text
Schematic : A, B
Layout    : VIN1, VIN2
```

처럼 이름이 다르면 Pin Error가 발생할 수 있다.

### VDD/VSS Pin Error

Power Rail은 존재하지만 실제 Top-Level Pin이 생성되어 있지 않으면 LVS에서 Unbound Pin으로 나타날 수 있다.

---

## 16. Layout 면적 최적화

Half Adder는 XOR와 AND 두 Cell을 사용하는 만큼 단일 Logic Gate보다 면적이 커진다.

면적 최적화를 위해 다음 요소를 고려할 수 있다.

* XOR와 AND Cell 간 간격 감소
* A/B Routing 길이 최소화
* SUM/CARRY Pin의 적절한 위치 선정
* VDD/VSS Rail 정렬
* 불필요한 Via 감소
* 불필요한 상위 Metal 감소
* Cell orientation 조절
* DRC Minimum Spacing 활용

특히 Cell을 단순히 가까이 배치하는 것보다 **Cell Pin의 위치를 고려해 배치하는 것이 Routing 면적 감소에 더 중요할 수 있다.**

---

## 17. Propagation Delay

Half Adder의 SUM과 CARRY는 서로 다른 Logic Path를 가진다.

SUM Path는

```text
A/B → XOR → SUM
```

이고 CARRY Path는

```text
A/B → AND → CARRY
```

이다.

따라서 두 출력의 propagation delay는 서로 다를 수 있다.

```text
tSUM ≈ tXOR

tCARRY ≈ tAND
```

AND가 NAND와 Inverter의 2-stage 구조라면 CARRY path는 내부적으로

```text
Input → NAND → Inverter → CARRY
```

를 거친다.

반면 XOR의 topology에 따라 SUM path의 Logic Depth와 등가 저항이 달라진다.

따라서 Half Adder에서는 단순히 논리 기능만 맞는 것이 아니라 `SUM`과 `CARRY`의 Timing 차이도 고려할 수 있다.

---

## 18. Parasitic에 대한 고찰

Half Adder 상위 Layout에서는 하위 Cell 내부의 parasitic뿐만 아니라 Cell 사이 Routing에 의한 parasitic도 추가된다.

전체 delay는 단순화하면

```text
Delay ∝ Equivalent Resistance × Load Capacitance
```

로 볼 수 있다.

Layout에서 다음 요소들이 parasitic을 증가시킬 수 있다.

* 긴 Metal Routing
* 불필요한 Via
* 큰 Metal 면적
* Cell 사이의 긴 거리
* 높은 Fan-out
* 인접 배선과의 Coupling

따라서 Layout 이후 PEX를 수행하면 Schematic Simulation에서는 보이지 않았던 interconnect parasitic까지 고려할 수 있다.

---

## 19. Hierarchical Design의 장점

Half Adder 설계에서 Hierarchical Design의 장점이 더욱 명확하게 나타난다.

기존에

```text
XOR → DRC/LVS PASS
AND → DRC/LVS PASS
```

상태라면 Half Adder에서는 두 Cell 내부 transistor를 모두 다시 검증하기보다 상위에서 새롭게 생성된 연결을 중심으로 설계할 수 있다.

전체 설계 흐름은 다음과 같다.

```text
MOSFET
  ↓
INV / NAND / NOR
  ↓
AND / OR / XOR
  ↓
Half Adder
```

즉 기본 Cell을 검증한 후 상위 Cell에서 반복 재사용함으로써 설계 복잡도를 단계적으로 관리할 수 있다.

---

## 20. Full Adder와의 연관성

Half Adder는 이후 Full Adder를 구성하는 핵심 Block이다.

일반적인 Full Adder는 두 개의 Half Adder와 하나의 OR Gate를 이용하여 구성할 수 있다.

```text
A ─────┐
       │
       ▼
   ┌─────────┐
B ─▶│ HA1    │
   └─────────┘
      │ SUM1
      │
      ▼
   ┌─────────┐
Cin▶│ HA2    │──── SUM
   └─────────┘
      │ C2
      │
C1 ───┴──── OR ─── COUT
```

논리적으로는

```text
SUM1 = A XOR B
C1   = A · B
```

두 번째 Half Adder에서

```text
SUM  = SUM1 XOR Cin
C2   = SUM1 · Cin
```

그리고 최종 Carry는

```text
COUT = C1 + C2
```

가 된다.

따라서 Half Adder를 정확하게 설계하고 검증해 두는 것은 Full Adder Layout 설계의 기반이 된다.

---

## 21. 설계 고찰

Half Adder는 XOR와 AND를 단순히 조합한 회로이지만 Layout 관점에서는 단일 Logic Gate와 다른 중요한 특징을 갖는다.

가장 큰 차이는 **동일한 입력 A와 B를 여러 하위 Cell에 분배해야 한다는 점**이다.

이 때문에 상위 Cell에서는 하위 Cell의 내부 구조보다

```text
Placement
Routing
Pin 위치
Power Rail
Net Connectivity
```

가 더욱 중요해진다.

특히 입력 Routing이 한 Instance까지만 연결되어 있거나 서로 다른 Net이 잘못 합쳐지면 하위 Cell 자체는 모두 LVS PASS여도 Top-Level Half Adder에서는 LVS Error가 발생할 수 있다.

이를 통해 Hierarchical Design에서는

```text
하위 Cell이 정상
≠
상위 Cell도 자동으로 정상
```

이라는 점을 확인할 수 있다.

상위 Cell에서 새롭게 형성되는 모든 연결은 다시 DRC/LVS 검증이 필요하다.

---

## 22. 최종 고찰

Half Adder는

```text
SUM   = A XOR B
CARRY = A · B
```

의 두 출력을 생성하는 기본적인 조합논리회로이다.

본 설계에서는 기존에 설계하고 검증한 XOR와 AND Cell을 재사용하여

```text
Half Adder = XOR + AND
```

의 Hierarchical 구조로 구현하였다.

Half Adder Layout에서는 transistor-level 설계보다 **검증된 Cell 사이의 배치와 Routing을 효율적으로 구성하는 과정**이 주요 설계 요소가 된다.

특히 입력 A/B가 XOR와 AND에 동시에 정확하게 연결되어야 하며, SUM과 CARRY 출력 및 VDD/VSS Power Net 역시 Schematic과 동일하게 구성해야 한다.

또한 Routing 과정에서는 필요 이상의 상위 Metal과 Via 사용을 줄이고, Cell 사이의 거리를 줄여 불필요한 parasitic과 Layout 복잡도를 감소시키는 것이 중요하다.

최종적인 설계 흐름은 다음과 같이 정리할 수 있다.

```text
XOR / AND 설계 및 검증
        ↓
Cell Placement
        ↓
Shared Input Routing
        ↓
SUM / CARRY Routing
        ↓
Power Routing
        ↓
DRC
        ↓
LVS
        ↓
PEX / Post-Layout Simulation
```

Half Adder 설계를 통해 기본 Logic Gate를 단순히 개별적으로 설계하는 단계를 넘어, **검증된 Logic Cell을 조합하여 더 큰 Digital Block을 구성하는 Hierarchical Full-Custom Design 방법**을 적용할 수 있었다.

또한 이러한 Half Adder Cell은 이후 두 개의 Half Adder와 OR Gate를 이용한 Full Adder 설계의 핵심 하위 Block으로 재사용할 수 있다.
