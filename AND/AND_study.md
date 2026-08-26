# CMOS AND Gate Full-Custom Layout 설계 보고서

## 1. 설계 개요

본 설계에서는 CMOS AND Gate를 Full-Custom 방식으로 구현하였다.

AND Gate를 transistor level에서 처음부터 새롭게 구성하는 대신, 기존에 설계하고 검증한 **2-input NAND Gate와 CMOS Inverter를 계층적으로 조합하여 AND Gate를 구현**하였다.

AND Gate의 논리식은 다음과 같다.

```text
Y = A · B
```

NAND Gate의 출력은

```text
X = NOT(A · B)
```

이고, NAND의 출력을 Inverter에 입력하면

```text
Y = NOT(X)
  = NOT(NOT(A · B))
  = A · B
```

가 된다.

따라서 전체 구조는 다음과 같다.

```text
AND = NAND + Inverter
```

회로 구조는 다음과 같다.

```text
A ─────┐
       ├──── NAND ──── X ──── Inverter ──── Y
B ─────┘
```

여기서 `X`는 NAND Gate의 출력이자 Inverter의 입력인 내부 Node이고, `Y`가 최종 AND 출력이다.

---

## 2. AND Gate 동작

2-input AND Gate의 Truth Table은 다음과 같다.

| A | B | NAND Output X | AND Output Y |
| - | - | ------------- | ------------ |
| 0 | 0 | 1             | 0            |
| 0 | 1 | 1             | 0            |
| 1 | 0 | 1             | 0            |
| 1 | 1 | 0             | 1            |

즉 두 입력이 모두 HIGH일 때만 출력이 HIGH가 된다.

```text
A = 1 and B = 1
→ Y = 1
```

그 외의 경우에는

```text
Y = 0
```

이다.

---

## 3. Transistor 구성

2-input NAND Gate는 다음과 같이 구성된다.

* PMOS 2개: Parallel
* NMOS 2개: Series
* 총 4 MOSFET

CMOS Inverter는

* PMOS 1개
* NMOS 1개
* 총 2 MOSFET

을 사용한다.

따라서 AND Gate 전체에서는

```text
4 MOSFET + 2 MOSFET = 6 MOSFET
```

즉,

```text
Total MOSFET = 6
```

이다.

---

## 4. Hierarchical Design

AND Gate Layout에서는 기존에 검증한 NAND와 Inverter Cell을 재사용하였다.

```text
AND
├── NAND
│   ├── PMOS
│   ├── PMOS
│   ├── NMOS
│   └── NMOS
└── Inverter
    ├── PMOS
    └── NMOS
```

이러한 설계 방법을 Hierarchical Design이라고 할 수 있다.

하위 Cell이 이미 DRC와 LVS를 통과한 상태라면 상위 AND에서는 transistor 하나하나를 다시 설계할 필요 없이 **Cell 사이의 연결과 배치에 집중할 수 있다.**

---

## 5. Layout 구성

AND Layout에서 가장 중요한 내부 연결은 다음과 같다.

```text
NAND VOUT → Inverter VIN
```

전체 연결은 다음과 같이 구성된다.

```text
VIN1 ─────┐
          │
          ├──── NAND ──── X ──── Inverter ──── VOUT
          │
VIN2 ─────┘
```

Top-Level Pin은 다음과 같이 구성할 수 있다.

```text
VIN1
VIN2
VOUT
VDD
VSS
```

NAND 출력 `X`는 상위 Cell 외부로 노출할 필요가 없는 내부 Net이다.

---

## 6. Cell Placement

NAND와 Inverter 사이에는 반드시 내부 신호 `X`가 연결되어야 한다.

따라서 두 Cell을 지나치게 멀리 배치하면 NAND 출력과 Inverter 입력 사이의 Metal 길이가 증가한다.

배선 길이가 증가하면 일반적으로

```text
Wire Length ↑
→ Wire Resistance ↑
→ Wire Capacitance ↑
→ Delay 증가 가능
```

와 같은 영향을 받을 수 있다.

따라서 NAND 출력과 Inverter 입력이 가능한 가까워지도록 Cell을 배치하는 것이 유리하다.

---

## 7. Internal Node X

내부 Node `X`는

```text
X = NOT(A · B)
```

이며 NAND의 출력이자 Inverter의 입력이다.

이 Node에는 다음과 같은 parasitic 성분이 존재할 수 있다.

```text
NAND Output Diffusion Capacitance
+ Metal Wire Capacitance
+ Inverter Input Gate Capacitance
```

따라서 NAND와 Inverter 사이의 배선을 가능한 짧게 구성하면 불필요한 parasitic을 감소시키는 데 도움이 된다.

---

## 8. VDD / VSS Routing

NAND와 Inverter는 동일한 VDD와 VSS를 사용한다.

```text
════════════════════ VDD
      │        │
    NAND      INV
      │        │
      └── X ───┘
      │        │
════════════════════ VSS
```

가능하다면 두 Cell의 VDD/VSS Rail을 동일한 위치에 정렬하여 전원 Routing을 단순화하는 것이 좋다.

또한 Schematic과 Layout에서

```text
VDD
VSS
```

Pin 이름이 정확히 일치해야 LVS Pin Error를 방지할 수 있다.

---

## 9. Metal Layer 사용

AND 내부의 NAND 출력과 Inverter 입력 연결은 짧은 Local Routing이다.

따라서 낮은 Metal에서 충분히 Routing할 수 있다면 불필요하게 상위 Metal까지 사용할 필요가 없다.

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

와 같이 높은 Metal까지 올라가면 Via 수가 증가한다.

불필요한 상위 Metal 사용은 다음과 같은 단점이 있다.

* Via 수 증가
* Via Resistance 증가
* Layout 복잡도 증가
* DRC 조건 증가
* Routing Resource 낭비
* LVS Debugging 난이도 증가
* 추가 Parasitic 발생 가능

따라서 Local Routing은 가능한 한 필요한 수준의 Metal만 사용하여 구성하는 것이 효율적이다.

---

## 10. DRC

Layout 작성 후 DRC를 수행하여 제조 규칙을 만족하는지 확인한다.

주요 검사 항목은 다음과 같다.

* Minimum Width
* Minimum Spacing
* Minimum Area
* Metal Spacing
* Via Spacing
* Via Enclosure
* Well Rule
* Implant Rule

하위 NAND와 Inverter가 이미 DRC PASS 상태라도 상위 AND에서 새롭게 추가한 Routing에 대해서는 다시 DRC가 필요하다.

```text
Final Check → DRC PASS
```

---

## 11. LVS

LVS에서는 Layout과 Schematic의 전기적 연결을 비교한다.

AND에서는 다음 연결이 중요하다.

```text
VIN1 → NAND
VIN2 → NAND
NAND VOUT → Inverter VIN
Inverter VOUT → AND VOUT
```

전원은

```text
NAND VDD = Inverter VDD = VDD
NAND VSS = Inverter VSS = VSS
```

가 되어야 한다.

대표적인 LVS Error에는 다음이 있다.

* Open Net
* Short Net
* Pin Mismatch
* Unbound Pin
* Bad Net Connection

최종적으로

```text
DRC PASS
+
LVS PASS
```

를 모두 만족해야 한다.

---

## 12. NAND와 AND 비교

| 항목            | NAND         | AND        |
| ------------- | ------------ | ---------- |
| Logic         | `NOT(A · B)` | `A · B`    |
| PMOS          | 2            | 3          |
| NMOS          | 2            | 3          |
| Total MOSFET  | 4            | 6          |
| Logic Stage   | 1            | 2          |
| 추가 회로         | 없음           | Inverter   |
| 내부 Stage Node | 없음           | NAND → INV |

AND는 NAND 출력에 Inverter가 추가된 구조이므로 NAND보다 transistor 수와 Layout 면적이 증가한다.

---

## 13. Propagation Delay

AND는 두 Logic Stage를 사용한다.

```text
Input
→ NAND
→ Inverter
→ Output
```

따라서 전체 Delay는 단순하게

```text
AND Delay ≈ NAND Delay + Inverter Delay
```

로 생각할 수 있다.

실제로는 NAND 출력의 parasitic capacitance, Inverter Gate capacitance, 두 Cell 사이의 배선 parasitic 등이 추가로 영향을 준다.

---

## 14. 설계 고찰

AND Gate 설계를 통해 검증된 기본 Cell을 재사용하는 Hierarchical Design의 장점을 확인할 수 있었다.

특히 하위 Cell이 이미 DRC/LVS를 통과했다 하더라도 상위 Cell에서는

* Cell Placement
* Cell 간 Routing
* Top-Level Pin
* VDD/VSS 연결

등을 다시 검증해야 한다.

또한 Schematic에서 단순한 Wire로 표현되는 내부 연결도 Layout에서는 실제 Metal이므로 Resistance와 Capacitance를 가진다는 점을 고려해야 한다.

---

## 15. 최종 고찰

AND Gate는

```text
AND = NAND + Inverter
```

구조를 이용하여 구현하였다.

논리적으로는

```text
Y = NOT(NOT(A · B))
  = A · B
```

이다.

본 설계를 통해 단순히 transistor를 직접 구성하는 것뿐만 아니라, 이미 검증된 Logic Cell을 계층적으로 조합하여 보다 복잡한 회로를 구성할 수 있음을 확인하였다.

전체 설계 흐름은 다음과 같이 정리할 수 있다.

```text
NAND / Inverter 설계
        ↓
DRC / LVS 검증
        ↓
Cell Reuse
        ↓
AND Hierarchical Integration
        ↓
Routing Optimization
        ↓
Top-Level DRC / LVS
```

이러한 방식은 이후 Half Adder, Full Adder 등 보다 큰 Digital Logic Block을 Full-Custom 방식으로 설계할 때 기본적인 설계 방법으로 활용할 수 있다.
