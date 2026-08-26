# CMOS Full Adder Full-Custom Layout 설계 보고서

## 1. 설계 개요

본 설계에서는 **1-bit Full Adder**를 Full-Custom 방식으로 구현하였다.

Full Adder는 세 입력 `A`, `B`, `Cin`을 받아 `SUM`과 `COUT`을 출력한다.

```text
SUM  = A XOR B XOR Cin
COUT = (A · B) + (Cin · (A XOR B))
```

본 설계에서는 기존에 설계하고 검증한 **Half Adder 2개와 OR Gate 1개**를 계층적으로 조합하였다.

```text
Full Adder = 2 × Half Adder + OR
```

전체 구조는 다음과 같다.

```text
A ─────┐
       ▼
    ┌─────┐
B ─▶ │ HA1 │
    └─────┘
     │   │
   SUM1  C1
     │   │
     ▼   │
    ┌─────┐
Cin▶│ HA2 │──── SUM
    └─────┘
        │
        C2
        │
C1 ─────┴──── OR ──── COUT
```

---

## 2. Full Adder 동작

Truth Table은 다음과 같다.

| A | B | Cin | SUM | COUT |
| - | - | --- | --- | ---- |
| 0 | 0 | 0   | 0   | 0    |
| 0 | 0 | 1   | 1   | 0    |
| 0 | 1 | 0   | 1   | 0    |
| 0 | 1 | 1   | 0   | 1    |
| 1 | 0 | 0   | 1   | 0    |
| 1 | 0 | 1   | 0   | 1    |
| 1 | 1 | 0   | 0   | 1    |
| 1 | 1 | 1   | 1   | 1    |

첫 번째 Half Adder에서는

```text
SUM1 = A XOR B
C1   = A · B
```

를 생성한다.

두 번째 Half Adder에서는

```text
SUM = SUM1 XOR Cin
C2  = SUM1 · Cin
```

을 생성하고, 마지막으로

```text
COUT = C1 + C2
```

를 통해 최종 Carry를 생성한다.

---

## 3. Hierarchical Design

전체 계층 구조는 다음과 같다.

```text
FULL_ADDER
├── HALF_ADDER 1
│   ├── XOR
│   └── AND
│
├── HALF_ADDER 2
│   ├── XOR
│   └── AND
│
└── OR
```

보다 전체적인 설계 흐름으로 보면

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

형태로 설계가 확장된다.

검증된 하위 Cell을 재사용하기 때문에 Full Adder에서는 transistor 하나하나보다 **Cell Placement와 Cell 간 Routing**이 중요한 설계 요소가 된다.

---

## 4. Layout 구성

Full Adder Layout에서 중요한 내부 연결은 다음과 같다.

```text
HA1 SUM   → HA2 Input
Cin       → HA2 Input

HA1 CARRY → OR Input
HA2 CARRY → OR Input

HA2 SUM   → SUM
OR VOUT   → COUT
```

Top-Level Pin은 다음과 같이 구성한다.

```text
A
B
Cin
SUM
COUT
VDD
VSS
```

특히 `HA1 SUM → HA2`와 두 Carry 신호의 `OR` 연결은 Full Adder의 핵심 내부 Routing이다.

---

## 5. Cell Placement

Full Adder에서는 하위 Cell을 단순히 가까이 배치하는 것보다 **Cell 간 Connectivity를 고려하여 배치하는 것**이 중요하다.

주요 고려사항은 다음과 같다.

* HA1과 HA2의 SUM 연결 최소화
* HA1/HA2 Carry와 OR 사이의 거리 최소화
* Routing 교차 최소화
* VDD/VSS Rail 정렬
* Via 사용 최소화
* Top-Level Pin 접근성 확보

좋은 Placement를 통해 Routing 길이와 Layout 면적을 동시에 감소시킬 수 있다.

---

## 6. VDD / VSS Routing

두 Half Adder와 OR Gate는 동일한 VDD와 VSS를 공유한다.

```text
════════════════════════ VDD
    │         │        │
   HA1       HA2       OR
    │         │        │
════════════════════════ VSS
```

따라서

```text
HA1 VDD = HA2 VDD = OR VDD = VDD
HA1 VSS = HA2 VSS = OR VSS = VSS
```

가 되어야 한다.

Power Rail을 동일한 방향으로 정렬하면 상위 Layout의 전원 Routing을 단순화할 수 있다.

---

## 7. Metal4 → Metal3 Routing 개선

초기 Full Adder Layout에서는 Routing 편의성을 위해 **Metal4까지 사용**하였다.

이후 Layout을 다시 검토하여 불필요한 상위 Metal을 제거하고 **Metal3 이하에서 Routing이 완료되도록 재설계**하였다.

기존 구조:

```text
M1
 ↓
M2
 ↓
M3
 ↓
M4
```

개선 구조:

```text
M1
 ↓
M2
 ↓
M3
```

Metal4를 제거함으로써 단순히 사용 Layer의 수를 줄이는 것뿐만 아니라 전체 Routing 구조를 보다 단순하게 구성할 수 있었다.

---

## 8. 불필요한 상위 Metal 사용에 대한 고찰

상위 Metal 자체가 나쁜 것은 아니다.

상위 Metal은 일반적으로

* Long Global Signal
* Clock
* Power Routing
* Large Bus

등에서 유용하다.

그러나 Full Adder 내부의 짧은 Local Signal에서 필요 이상으로 높은 Metal을 사용하면 다음과 같은 단점이 발생할 수 있다.

* Via 수 증가
* Via Resistance 증가
* Routing 복잡도 증가
* DRC 조건 증가
* LVS Debugging 복잡도 증가
* 상위 Metal Routing Resource 사용
* 추가 Parasitic 발생 가능성

따라서 중요한 것은

```text
Upper Metal 사용 금지 X
불필요한 Upper Metal 사용 최소화 O
```

이다.

본 설계에서는 Metal4까지 사용했던 Routing을 Metal3 이하로 재구성함으로써 **동일한 Connectivity를 더 단순한 Physical Structure로 구현**하였다.

---

## 9. LVS Debugging

Full Adder 설계 과정에서는 다음과 같은 LVS Error를 확인할 수 있었다.

```text
Bad Initial Net Bindings
Bad Matched Nets
Open Internal Nets
Unmatched Internal Nets
Suggested Terminal Rewire
Matched Instances with Bad Net Connections
```

특히 발생했던 문제는 크게 두 가지로 정리할 수 있다.

### Half Adder 입력 연결 오류

Schematic과 Layout에서 Half Adder의 `VIN1`, `VIN2` 연결이 서로 다르면 LVS에서 Terminal Rewire 관련 Error가 발생할 수 있다.

```text
Schematic:
VIN1 = Cin
VIN2 = SUM1

Layout:
VIN1 = SUM1
VIN2 = Cin
```

따라서 Cell을 Rotate/Mirror한 경우에도 실제 Pin 위치와 Net을 정확하게 확인해야 한다.

### Carry Net Open

Half Adder의 Carry와 OR 입력 사이가 끊어지면 Schematic에서는 하나의 Net이지만 Layout에서는 두 개의 Net으로 인식된다.

```text
HA Carry ───── X ───── OR Input
               ↑
              Open
```

LVS Report에서

```text
These layout nets should connect together
```

와 같은 메시지가 나타난다면 두 Layout Net 사이의 Open을 우선 확인할 수 있다.

---

## 10. DRC / LVS 검증

하위 Half Adder와 OR가 모두 DRC/LVS PASS 상태라도 Full Adder에서 새롭게 생성된 Routing은 다시 검증해야 한다.

DRC에서는

* Metal Width
* Metal Spacing
* Via Spacing
* Via Enclosure
* Minimum Area

등을 확인한다.

LVS에서는 다음 Connectivity를 확인한다.

```text
A, B → HA1

HA1 SUM → HA2
Cin     → HA2

HA1 CARRY → OR
HA2 CARRY → OR

HA2 SUM → SUM
OR VOUT → COUT
```

최종적으로

```text
DRC PASS
+
LVS PASS
```

를 모두 만족해야 한다.

---

## 11. Propagation Delay 및 Parasitic

Full Adder의 SUM Path는 대략

```text
A/B
 ↓
HA1 XOR
 ↓
SUM1
 ↓
HA2 XOR
 ↓
SUM
```

으로 구성된다.

Carry Path는 입력에 따라

```text
A/B → HA1 AND → OR → COUT
```

또는

```text
A/B → HA1 XOR → HA2 AND → OR → COUT
```

경로를 가질 수 있다.

따라서 여러 Logic Stage와 Cell 간 Interconnect로 인해 propagation delay가 발생한다.

Layout에서는

```text
Metal Resistance
Metal Capacitance
Via Resistance
Coupling Capacitance
```

등의 parasitic이 추가되므로 내부 Routing을 가능한 짧고 단순하게 구성하는 것이 중요하다.

---

## 12. 설계 고찰

Full Adder 설계를 통해 **Hierarchical Full-Custom Layout에서 Cell 간 Routing의 중요성**을 확인할 수 있었다.

하위 Half Adder와 OR가 모두 정상적으로 검증되었더라도 상위 Full Adder에서는

```text
Open
Short
Pin Swap
Internal Net Mismatch
```

등의 새로운 문제가 발생할 수 있다.

즉,

```text
Sub-cell PASS
≠
Top-level PASS
```

이며, 상위 계층에서 생성된 모든 Connectivity를 다시 검증해야 한다.

또한 초기 Metal4 Routing을 Metal3 이하로 재설계하면서 단순히 회로를 연결하는 것에서 끝나는 것이 아니라 **동일한 기능을 더 효율적인 Physical Structure로 구현하는 것이 Layout Optimization의 핵심**임을 확인하였다.

---

## 13. 최종 고찰

Full Adder의 논리식은 다음과 같다.

```text
SUM  = A XOR B XOR Cin
COUT = (A · B) + (Cin · (A XOR B))
```

본 설계에서는

```text
Full Adder
= 2 × Half Adder + OR
```

구조를 이용하여 Hierarchical Full-Custom Layout을 구현하였다.

전체 설계 과정은 다음과 같이 정리할 수 있다.

```text
Half Adder / OR 검증
        ↓
Cell Placement
        ↓
Internal Net Routing
        ↓
Metal / Via Optimization
        ↓
Metal4 → Metal3 개선
        ↓
DRC
        ↓
LVS
        ↓
PEX / Post-Layout Simulation
```

Full Adder 설계를 통해 좋은 Full-Custom Layout은 단순히 Schematic과 동일하게 연결하는 것뿐만 아니라

```text
Correct Connectivity
+ Efficient Placement
+ Short Routing
+ Appropriate Metal Usage
+ Low Parasitic
+ DRC / LVS Verification
```

을 함께 고려해야 한다는 점을 확인할 수 있었다.

Full Adder는 이후 Ripple Carry Adder, Multi-bit Adder, MAC 및 Dot Product Accelerator와 같은 더 큰 Arithmetic Circuit을 구성하기 위한 기본 연산 Cell로 활용할 수 있다.
