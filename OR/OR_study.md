# CMOS OR Gate Full-Custom Layout 설계 보고서

## 1. 설계 개요

본 설계에서는 CMOS OR Gate를 Full-Custom 방식으로 구현하였다.

OR Gate를 transistor level에서 새롭게 설계하는 대신 기존에 설계한 **2-input NOR Gate와 CMOS Inverter를 조합하여 구현**하였다.

OR Gate의 논리식은

```text
Y = A + B
```

이다.

NOR Gate의 출력은

```text
X = NOT(A + B)
```

이고, 여기에 Inverter를 연결하면

```text
Y = NOT(X)
  = NOT(NOT(A + B))
  = A + B
```

가 된다.

따라서

```text
OR = NOR + Inverter
```

로 구현할 수 있다.

전체 구조는 다음과 같다.

```text
A ─────┐
       ├──── NOR ──── X ──── Inverter ──── Y
B ─────┘
```

---

## 2. OR Gate 동작

Truth Table은 다음과 같다.

| A | B | NOR Output X | OR Output Y |
| - | - | ------------ | ----------- |
| 0 | 0 | 1            | 0           |
| 0 | 1 | 0            | 1           |
| 1 | 0 | 0            | 1           |
| 1 | 1 | 0            | 1           |

즉 입력 중 하나 이상이 HIGH이면 출력이 HIGH가 된다.

```text
A = 0, B = 0 → Y = 0

A = 1 or B = 1 → Y = 1
```

---

## 3. Transistor 구성

2-input NOR Gate는

* PMOS 2개: Series
* NMOS 2개: Parallel
* 총 4 MOSFET

으로 구성된다.

Inverter는

* PMOS 1개
* NMOS 1개
* 총 2 MOSFET

을 사용한다.

따라서 OR 전체에서는

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

OR Layout은 NOR와 Inverter를 하위 Cell로 사용하는 구조이다.

```text
OR
├── NOR
│   ├── PMOS
│   ├── PMOS
│   ├── NMOS
│   └── NMOS
└── Inverter
    ├── PMOS
    └── NMOS
```

이미 검증된 하위 Cell을 재사용하면 상위 회로의 설계 복잡도를 줄일 수 있다.

---

## 5. Layout 구성

OR Layout에서 가장 중요한 연결은

```text
NOR VOUT → Inverter VIN
```

이다.

전체 구성은 다음과 같다.

```text
VIN1 ─────┐
          ├──── NOR ──── X ──── Inverter ──── VOUT
VIN2 ─────┘
```

Top-Level Pin은 일반적으로 다음과 같이 구성할 수 있다.

```text
VIN1
VIN2
VOUT
VDD
VSS
```

---

## 6. Cell Placement

NOR와 Inverter의 거리가 길어질수록 내부 Node `X`의 Metal Routing도 길어진다.

```text
Wire Length ↑
→ Rwire ↑
→ Cwire ↑
→ Delay 증가 가능
```

따라서 NOR의 출력과 Inverter 입력을 가능한 가까이 배치하는 것이 효율적이다.

---

## 7. Internal Node X

NOR의 출력은

```text
X = NOT(A + B)
```

이다.

이 Node는 Inverter 입력으로 연결된다.

실제 Layout에서는 내부 Node `X`에

```text
NOR Output Parasitic
+ Wire Capacitance
+ Inverter Gate Capacitance
```

가 존재한다.

따라서 `X`의 배선 길이를 줄이는 것이 중요하다.

---

## 8. VDD / VSS Routing

NOR와 Inverter는 동일한 VDD와 VSS를 공유한다.

```text
════════════════════ VDD
      │        │
     NOR      INV
      │        │
      └── X ───┘
      │        │
════════════════════ VSS
```

두 Cell의 Power Rail을 정렬하면 Routing을 보다 단순하게 구성할 수 있다.

---

## 9. Metal Routing

OR 내부의 NOR-to-Inverter 연결은 Local Routing이므로 필요 이상의 상위 Metal 사용을 줄이는 것이 좋다.

불필요하게 Metal Layer를 높이면

* Via 증가
* Via Resistance 증가
* Layout 복잡도 증가
* DRC 조건 증가
* Routing Resource 낭비
* LVS Debugging 복잡도 증가

등의 문제가 발생할 수 있다.

따라서

```text
Local Signal → 필요한 수준의 Lower Metal
Global / Power / Clock → 필요시 Upper Metal
```

과 같이 사용하는 것이 효율적이다.

---

## 10. DRC

OR Top-Level에서도 새롭게 생성된 Routing에 대해 DRC를 수행해야 한다.

주요 항목은

* Width
* Spacing
* Area
* Via Enclosure
* Via Spacing
* Metal Rule

등이다.

```text
Final Layout → DRC PASS
```

를 확인해야 한다.

---

## 11. LVS

LVS에서는 다음 연결을 확인한다.

```text
VIN1 → NOR
VIN2 → NOR
NOR VOUT → Inverter VIN
Inverter VOUT → OR VOUT
```

전원은

```text
NOR VDD = Inverter VDD = VDD
NOR VSS = Inverter VSS = VSS
```

가 되어야 한다.

하위 Cell이 LVS PASS 상태라도 상위 OR의 Routing이 틀리면 Open, Short 또는 Pin Error가 발생할 수 있다.

---

## 12. NOR와 OR 비교

| 항목           | NOR          | OR        |
| ------------ | ------------ | --------- |
| Logic        | `NOT(A + B)` | `A + B`   |
| PMOS         | 2            | 3         |
| NMOS         | 2            | 3         |
| Total MOSFET | 4            | 6         |
| Logic Stage  | 1            | 2         |
| 추가 회로        | 없음           | Inverter  |
| 내부 Node      | 없음           | NOR → INV |

---

## 13. Propagation Delay

OR는

```text
Input
→ NOR
→ Inverter
→ Output
```

의 2-stage 구조이다.

따라서 단순화하면

```text
OR Delay ≈ NOR Delay + Inverter Delay
```

로 볼 수 있다.

NOR 출력과 Inverter 입력 사이의 배선 parasitic을 줄이면 추가 Delay를 감소시키는 데 도움이 된다.

---

## 14. 설계 고찰

OR Gate 설계를 통해 기존에 설계한 NOR와 Inverter를 재사용하여 상위 Logic Cell을 구성하는 Hierarchical Design 방식을 적용하였다.

하위 Cell 자체의 transistor Layout을 다시 설계할 필요는 없지만, 상위 Cell에서는

* Placement
* Routing
* Power 연결
* Pin 설정
* DRC
* LVS

를 다시 확인해야 한다.

---

## 15. 최종 고찰

OR Gate는

```text
OR = NOR + Inverter
```

구조로 구현하였다.

논리식은

```text
Y = NOT(NOT(A + B))
  = A + B
```

이다.

이번 설계를 통해 검증된 기본 Cell을 재사용하여 상위 회로를 구성할 수 있음을 확인하였다.

전체 흐름은 다음과 같다.

```text
NOR / Inverter 설계
        ↓
DRC / LVS 검증
        ↓
Cell Reuse
        ↓
OR Hierarchical Integration
        ↓
Routing Optimization
        ↓
Top-Level DRC / LVS
```

이는 이후 더 복잡한 Logic Cell과 Adder를 설계하는 데 활용할 수 있다.
