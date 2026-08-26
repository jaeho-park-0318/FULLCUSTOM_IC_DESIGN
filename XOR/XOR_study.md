# CMOS XOR Gate Full-Custom Layout 설계 보고서

## 1. 설계 개요

본 설계에서는 2-input CMOS XOR(Exclusive OR) Gate를 Full-Custom 방식으로 구현하였다.

XOR는 두 입력이 서로 다른 논리값을 가질 때 HIGH를 출력한다.

논리식은 다음과 같다.

```text
Y = A XOR B
```

Boolean 식으로 표현하면

```text
Y = (NOT A · B) + (A · NOT B)
```

이다.

즉,

```text
A = 0, B = 1
```

또는

```text
A = 1, B = 0
```

일 때 출력이 HIGH가 된다.

XOR는 이후 Half Adder에서

```text
SUM = A XOR B
```

를 생성하는 핵심 Logic Gate이다.

---

## 2. XOR 동작

Truth Table은 다음과 같다.

| A | B | Y |
| - | - | - |
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

즉,

```text
A = B   → Y = 0
A != B  → Y = 1
```

이다.

Boolean 식은

```text
Y = (NOT A · B) + (A · NOT B)
```

로 표현할 수 있다.

---

## 3. XOR 회로 구성

XOR는 NAND나 NOR보다 복잡한 Logic Network를 필요로 한다.

구현 방식에 따라 transistor 수와 topology가 달라질 수 있으므로 Layout에서는 반드시 실제 Schematic의 transistor 연결 관계를 기준으로 설계해야 한다.

개념적으로는 다음 신호가 사용될 수 있다.

```text
A
B
NOT A
NOT B
```

그리고

```text
NOT A · B
```

와

```text
A · NOT B
```

두 조건을 OR하여 최종 출력을 만든다.

```text
Y = (NOT A · B) + (A · NOT B)
```

---

## 4. Layout 설계

XOR는 기본적인 Inverter, NAND, NOR보다 transistor와 내부 Node가 많기 때문에 Layout 난이도가 높아진다.

Layout 시작 전에 다음을 분석해야 한다.

* Series transistor
* Parallel transistor
* Shared Diffusion 가능 여부
* A/B Gate 연결
* 반전 신호 연결
* 내부 Net
* 최종 VOUT Node

따라서 설계 순서를

```text
Schematic Topology 분석
        ↓
Transistor Ordering
        ↓
Placement
        ↓
Diffusion Sharing
        ↓
Routing
```

순으로 진행하는 것이 효율적이다.

---

## 5. Transistor Placement

XOR는 내부 연결이 복잡하기 때문에 transistor ordering이 중요하다.

서로 직접 연결되는 transistor를 멀리 배치하면

```text
Routing Length ↑
Via Count ↑
Layout Area ↑
Parasitic ↑
```

가 발생할 수 있다.

반대로 connectivity를 고려하여 인접 배치하면

* 짧은 Routing
* 적은 Via
* 작은 Layout 면적
* 낮은 parasitic 가능성
* 높은 Layout 가독성

등의 장점을 얻을 수 있다.

---

## 6. Diffusion Sharing

Series로 연결되는 MOSFET의 Source/Drain이 동일한 Net을 공유한다면 diffusion sharing을 이용할 수 있다.

예를 들어

```text
M1 ── X ── M2
```

구조라면 Layout에서

```text
        Gate1       Gate2
          │           │
──────────│───────────│──────────
       Node1     X        Node2
```

와 같이 가운데 diffusion을 내부 Node `X`로 공유할 수 있다.

Diffusion Sharing은 다음과 같은 장점이 있다.

* Active Area 감소
* Contact 감소
* Metal Routing 감소
* Junction Capacitance 감소 가능
* Cell Area 감소

다만 XOR에서는 connectivity가 복잡하므로 **실제로 동일한 Net인지 확인한 후 sharing해야 한다.**

---

## 7. Input Routing

XOR에는

```text
A
B
```

두 입력이 존재한다.

Topology에 따라 각 입력은 여러 MOSFET의 Gate로 전달될 수 있다.

또한

```text
NOT A
NOT B
```

와 같은 반전 신호가 내부에서 사용될 수 있다.

따라서 A와 B가 정확한 Gate에 연결되는지 확인하는 것이 중요하다.

신호 교차가 발생하는 경우 서로 다른 Metal Layer를 이용해 Short를 방지할 수 있다.

Poly는 MOSFET Gate를 형성하는 데 사용하지만 긴 Routing에서는 Metal보다 저항이 크므로 필요한 지점에서 Contact를 통해 Metal로 전환하는 것이 일반적이다.

---

## 8. Output Routing

XOR 출력은

```text
VOUT = A XOR B
```

이다.

VOUT에는 다음과 같은 capacitance가 존재할 수 있다.

```text
Output Diffusion Capacitance
+ Wire Capacitance
+ Next Stage Gate Capacitance
```

따라서 VOUT Routing을 가능한 짧게 구성하고 출력 Node의 불필요한 diffusion 면적을 줄이는 것이 좋다.

---

## 9. Internal Node

XOR는 NAND나 NOR보다 내부 Node가 많을 수 있다.

Schematic에서는 단순한 Wire로 표시되지만 Layout에서는 실제 물리적인 구조이다.

따라서 내부 Node에는

```text
Resistance != 0
Capacitance != 0
```

이다.

즉 내부 배선이 길어지면 parasitic 증가로 인해 delay에 영향을 줄 수 있다.

따라서 외부 출력뿐만 아니라 **내부 Net Routing도 가능한 짧게 구성하는 것이 중요하다.**

---

## 10. VDD / VSS Routing

일반적인 Layout에서는 상단에 VDD, 하단에 VSS Rail을 배치할 수 있다.

```text
════════════════════ VDD

      PMOS Network

        XOR Logic

      NMOS Network

════════════════════ VSS
```

Body Connection은 일반적으로 다음과 같다.

```text
PMOS Body → VDD
NMOS Body → VSS
```

Well/Substrate Contact를 이용하여 Body Potential을 안정적으로 유지해야 한다.

---

## 11. Metal Layer 사용

XOR는 내부 Routing이 복잡하여 신호가 교차하는 경우가 많다.

따라서 여러 Metal Layer를 사용할 수 있지만 필요 이상으로 높은 Layer를 사용하면 Via가 증가한다.

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

불필요한 상위 Metal 사용의 단점은 다음과 같다.

* Via 증가
* Via Resistance 증가
* Layout 복잡도 증가
* DRC Rule 증가
* Routing Resource 낭비
* LVS Debugging 난이도 증가
* 추가 Parasitic 발생 가능

따라서

```text
필요한 Metal까지만 사용
```

하는 것이 중요하다.

단, 신호 교차나 congestion을 해결하기 위해 상위 Metal이 필요한 경우에는 사용하는 것이 적절하다.

핵심은

```text
상위 Metal 사용 금지 X
불필요한 상위 Metal 사용 최소화 O
```

이다.

---

## 12. Layout 면적 최적화

XOR Layout 최적화에서는 다음 요소를 고려할 수 있다.

* Transistor Ordering
* Diffusion Sharing
* Internal Net Routing 최소화
* Output Routing 최소화
* Input Routing 최적화
* Contact 감소
* Via 감소
* 불필요한 상위 Metal 감소
* VDD/VSS Rail 정렬

단순히 모든 구조를 가깝게 만드는 것이 아니라 DRC Rule을 만족하면서 면적과 Routing을 동시에 최적화해야 한다.

---

## 13. Propagation Delay

XOR의 Delay는 단순화하면 다음과 같이 생각할 수 있다.

```text
Delay ∝ Equivalent Resistance × Load Capacitance
```

즉,

```text
t_pd ∝ R_eq · C_L
```

이다.

Layout에서

* 긴 배선
* 큰 Diffusion
* 많은 Via
* 큰 출력 Node

가 존재하면 parasitic 증가로 인해 Post-Layout Delay가 증가할 수 있다.

따라서 Layout 이후에는 PEX를 통해 parasitic을 추출한 후 Post-Layout Simulation을 수행하는 것이 바람직하다.

---

## 14. DRC

XOR Layout 작성 후 DRC를 수행하여 PDK의 제조 규칙을 확인한다.

주요 항목은 다음과 같다.

* Minimum Width
* Minimum Spacing
* Minimum Area
* Poly Spacing
* Metal Spacing
* Contact Rule
* Via Rule
* Well Rule
* Implant Rule

XOR는 Routing이 복잡하기 때문에 Metal Spacing과 Via 관련 DRC를 특히 주의해야 한다.

---

## 15. LVS

LVS에서는 Schematic과 Layout의 전기적 동일성을 확인한다.

주요 확인 사항은 다음과 같다.

* PMOS/NMOS 개수
* 각 MOS Gate 연결
* A/B 입력 연결
* Internal Net 연결
* VOUT 연결
* VDD/VSS 연결
* Top-Level Pin

대표적인 LVS Error에는

* Open Net
* Short Net
* Bad Net Match
* Device Mismatch
* Pin Mismatch
* Unbound Pin

등이 있다.

최종적으로

```text
DRC PASS
+
LVS PASS
```

를 만족해야 한다.

---

## 16. 기본 Logic Gate와 XOR 비교

| 항목                | Inverter | NAND / NOR        | XOR      |
| ----------------- | -------- | ----------------- | -------- |
| 입력                | 1        | 2                 | 2        |
| 회로 구조             | 단순       | Series / Parallel | 복합 Logic |
| Internal Node     | 적음       | 증가                | 많음       |
| Routing 난이도       | 낮음       | 중간                | 높음       |
| Diffusion Sharing | 단순       | 중요                | 더욱 중요    |
| Metal 교차          | 적음       | 증가                | 많음       |
| Layout 난이도        | 낮음       | 중간                | 높음       |

XOR에서는 이전에 학습한

```text
Transistor Placement
Diffusion Sharing
Metal Routing
Via 사용
DRC
LVS
```

등의 내용을 종합적으로 적용하게 된다.

---

## 17. Half Adder와의 관계

Half Adder의 두 출력은

```text
SUM   = A XOR B
CARRY = A · B
```

이다.

따라서 XOR와 AND Cell을 이용하여 Half Adder를 구성할 수 있다.

```text
A ─────┬──── XOR ──── SUM
       │
B ─────┘

A ─────┬──── AND ──── CARRY
       │
B ─────┘
```

즉 XOR는 Half Adder의 SUM 생성에 직접 사용된다.

---

## 18. Hierarchical Design

XOR가 DRC/LVS를 통과하면 이후 상위 회로에서 하나의 검증된 Cell로 사용할 수 있다.

```text
MOSFET
  ↓
INV / NAND / NOR
  ↓
AND / OR / XOR
  ↓
Half Adder
  ↓
Full Adder
```

이러한 계층적 설계를 통해 회로 규모가 커져도 설계 복잡도를 관리할 수 있다.

---

## 19. 최종 고찰

XOR는

```text
Y = A XOR B
```

이며 Boolean 식으로는

```text
Y = (NOT A · B) + (A · NOT B)
```

로 표현할 수 있다.

XOR는 Inverter, NAND, NOR에 비해 내부 connectivity가 복잡하기 때문에 Layout에서는 transistor ordering과 Routing 계획이 특히 중요하다.

좋은 XOR Layout은 다음 요소를 함께 고려해야 한다.

```text
Correct Connectivity
+ Efficient Placement
+ Diffusion Sharing
+ Short Routing
+ Low Parasitic
+ Compact Area
```

특히 XOR는 이후 Half Adder의 SUM을 생성하는 핵심 Cell이므로 정확하게 설계하고 DRC/LVS를 통해 검증해 두는 것이 중요하다.

전체적인 설계 흐름은 다음과 같이 정리할 수 있다.

```text
Schematic 분석
      ↓
Transistor Ordering
      ↓
Placement
      ↓
Diffusion Sharing
      ↓
Routing
      ↓
DRC
      ↓
LVS
      ↓
PEX / Post-Layout Simulation
```

이를 통해 XOR는 단순한 Logic Gate 설계를 넘어 이후 Half Adder와 Full Adder를 구성하기 위한 중요한 Full-Custom Design 단계라고 할 수 있다.
