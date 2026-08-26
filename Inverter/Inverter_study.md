# CMOS Inverter Layout 설계 및 고찰

## 1. 설계 개요

CMOS Inverter는 **PMOS와 NMOS 각각 1개로 구성되는 가장 기본적인 CMOS 논리회로**이다. 단순한 논리 반전 기능뿐만 아니라 NAND, NOR, XOR, Adder, Latch 등 더 복잡한 디지털 회로를 구성하는 기본 단위가 된다.

Full-Custom IC Design에서는 Schematic 수준에서 회로의 논리적 동작을 확인하는 것뿐만 아니라, MOSFET을 실제 반도체 공정에서 제작할 수 있도록 **Physical Layout으로 구현하는 과정**이 필요하다.

CMOS Inverter의 기본 구조는 다음과 같다.

```text
          VDD
           |
         PMOS
           |
           +------ VOUT
           |
         NMOS
           |
          VSS

VIN ──────┬── PMOS Gate
          └── NMOS Gate
```

입력 `VIN`은 NMOS와 PMOS의 Gate에 공통으로 연결되고, 두 MOSFET의 Drain을 연결하여 `VOUT`을 구성한다.

---

## 2. CMOS Inverter의 동작

### VIN = LOW

입력 전압이 LOW일 경우,

* PMOS : ON
* NMOS : OFF

가 되어 VDD와 VOUT 사이에 conduction path가 형성된다.

따라서

$$
V_{OUT}\approx V_{DD}
$$

가 되어 HIGH가 출력된다.

### VIN = HIGH

입력 전압이 HIGH일 경우,

* PMOS : OFF
* NMOS : ON

이 되어 VOUT과 VSS 사이에 conduction path가 형성된다.

따라서

$$
V_{OUT}\approx V_{SS}
$$

가 되어 LOW가 출력된다.

결과적으로

$$
\boxed{V_{OUT}=\overline{V_{IN}}}
$$

의 논리적 반전 동작을 수행한다.

---

# 3. Inverter Layout의 기본 구조

Full-Custom Layout에서는 Schematic에 존재하는 PMOS와 NMOS를 실제 공정 Layer를 이용하여 구현한다.

일반적인 CMOS Inverter Layout은 위쪽에 PMOS, 아래쪽에 NMOS를 배치하는 구조를 사용한다.

```text
           VDD
══════════════════════
             │
         ┌─ PMOS ─┐
VIN ─────┤  Gate  │
         └────┬───┘
              │
            VOUT
              │
         ┌────┴───┐
VIN ─────┤  Gate  │
         └─ NMOS ─┘
             │
══════════════════════
           VSS
```

이와 같은 배치는 회로의 구조를 직관적으로 표현하면서 VDD/VSS 및 입출력 routing을 단순화할 수 있다는 장점이 있다.

---

# 4. PMOS와 NMOS 배치

PMOS는 일반적으로 **N-Well 내부**에 배치하며 NMOS는 P-type substrate 또는 P-Well 영역에 배치한다.

Layout에서는 PMOS와 NMOS를 수직 방향으로 배치하여 다음과 같은 구조를 만드는 것이 일반적이다.

```text
VDD
────────────────────

     PMOS
      │
      │
    VOUT
      │
      │
     NMOS

────────────────────
VSS
```

이렇게 배치하면 PMOS와 NMOS의 Drain 사이의 거리를 줄일 수 있어 VOUT 연결을 짧게 만들 수 있다.

배선 길이가 감소하면 일반적으로 interconnect resistance와 parasitic capacitance를 감소시키는 데 유리하므로 Layout의 면적뿐만 아니라 회로 성능 측면에서도 효과적이다.

---

# 5. Gate 연결

Inverter에서 가장 중요한 연결 중 하나는 PMOS와 NMOS의 Gate를 동일한 `VIN`에 연결하는 것이다.

$$
V_{G,P}=V_{G,N}=V_{IN}
$$

Layout에서는 Poly 또는 해당 공정에서 허용되는 Metal routing을 이용하여 두 Gate를 연결할 수 있다.

```text
        PMOS
          │
VIN ───── Gate
          │
          │
VIN ───── Gate
          │
        NMOS
```

Gate 연결에서는 불필요하게 배선을 길게 만들지 않는 것이 중요하다.

Gate에 존재하는 capacitance는 입력 부하에 직접 영향을 주므로 routing으로 인한 추가 parasitic을 최소화하는 것이 좋다.

---

# 6. Output 연결

PMOS와 NMOS의 Drain은 서로 연결하여 `VOUT`을 형성한다.

$$
D_P=D_N=V_{OUT}
$$

따라서 Layout에서도 두 Drain 사이를 가능한 짧고 단순하게 연결하는 것이 중요하다.

```text
PMOS Drain
     │
     ├────── VOUT
     │
NMOS Drain
```

이 노드에는 MOSFET의 junction capacitance와 다음 stage의 gate capacitance, interconnect capacitance 등이 존재한다.

이를 전체적으로 단순화하면

$$
C_L=C_{junction}+C_{wire}+C_{gate,next}+\cdots
$$

와 같은 load capacitance가 형성된다.

Inverter가 switching할 때 이 capacitance를 충·방전해야 하므로 VOUT 주변의 불필요한 배선과 면적을 줄이는 것이 propagation delay 감소에 중요하다.

---

# 7. VDD / VSS Routing

PMOS의 Source는 VDD에, NMOS의 Source는 VSS에 연결한다.

$$
S_P\rightarrow V_{DD}
$$

$$
S_N\rightarrow V_{SS}
$$

일반적으로 cell의 상단과 하단에 각각 VDD와 VSS rail을 구성하면 이후 여러 logic cell을 연결할 때도 일관된 구조를 유지할 수 있다.

```text
══════════════════════ VDD
       │
      PMOS
       │
      VOUT
       │
      NMOS
       │
══════════════════════ VSS
```

Power line은 신호 배선과 달리 여러 transistor의 전류를 공급해야 하므로 충분한 Metal width와 contact/via 구조를 확보해야 한다.

---

# 8. Well/Substrate Contact의 중요성

Layout에서는 단순히 Source, Drain, Gate만 연결하는 것으로 끝나지 않는다.

PMOS가 위치하는 N-Well은 적절한 전위에 고정해야 하며 일반적으로

$$
N\text{-Well}\rightarrow V_{DD}
$$

로 연결한다.

NMOS의 Body 역시

$$
P\text{-Substrate 또는 P-Well}\rightarrow V_{SS}
$$

로 연결한다.

즉,

```text
PMOS Body → VDD
NMOS Body → VSS
```

가 기본적인 구성이다.

Well/Substrate contact가 적절하지 않으면 body potential이 불안정해질 수 있고 body effect 및 noise 문제가 발생할 수 있다.

또한 적절한 substrate/well contact 배치는 **Latch-up 방지** 측면에서도 중요하다.

---

# 9. Contact와 Via

Active 영역이나 Poly를 Metal과 연결하기 위해서는 Contact가 필요하며 서로 다른 Metal layer 사이의 연결에는 Via를 사용한다.

예를 들어,

```text
Active
  │
Contact
  │
Metal1
  │
Via1
  │
Metal2
```

와 같은 구조가 사용될 수 있다.

Contact와 Via는 단순히 두 layer를 겹쳐놓는 것이 아니라 PDK에서 정의한 enclosure, spacing, width 등의 Design Rule을 만족해야 한다.

또한 불필요하게 높은 Metal까지 routing하면 여러 개의 Via가 필요해진다.

예를 들어,

```text
M1
 │ Via1
M2
 │ Via2
M3
 │ Via3
M4
```

와 같은 구조는 짧은 local signal에서는 오히려 Layout 복잡성과 via resistance를 증가시킬 수 있다.

따라서 Inverter와 같은 작은 cell 내부에서는 **필요한 범위 내에서 가능한 단순한 Metal routing을 구성하는 것이 중요하다.**

---

# 10. Layout 면적 최적화

Full-Custom Layout에서는 단순히 연결만 정상적으로 구현하는 것이 아니라 **동일한 기능을 얼마나 작은 면적으로 효율적으로 구현할 수 있는지**도 중요하다.

Layout 면적을 감소시키기 위해서는 다음을 고려할 수 있다.

* PMOS와 NMOS의 적절한 배치
* Source/Drain 주변의 불필요한 공간 감소
* 배선 길이 최소화
* 불필요한 Metal layer 사용 감소
* 불필요한 Via 감소
* PDK의 Minimum Spacing 활용
* Well 및 Implant Design Rule 고려

다만 무조건 모든 구조를 최소 간격으로 배치하는 것이 최선은 아니다.

지나친 면적 최적화는 DRC violation을 발생시키거나 routing을 어렵게 만들 수 있으므로 **Design Rule을 만족하면서 배선과 소자 배치를 효율적으로 구성하는 것**이 중요하다.

---

# 11. DRC

**DRC(Design Rule Check)**는 작성한 Layout이 실제 공정에서 제작 가능한 구조인지 검사하는 과정이다.

대표적으로 다음 항목을 검사한다.

* Minimum Width
* Minimum Spacing
* Minimum Enclosure
* Minimum Area
* Contact/Via Rule
* Well Rule
* Implant Rule

따라서

$$
\boxed{\text{DRC PASS}}
$$

는 Layout이 해당 PDK의 물리적 설계 규칙을 만족한다는 것을 의미한다.

하지만 DRC PASS만으로 회로가 Schematic과 동일하다는 것을 보장하지는 않는다.

---

# 12. LVS

**LVS(Layout Versus Schematic)**는 Layout에서 추출한 회로와 원래 작성한 Schematic을 비교하는 과정이다.

대표적으로 다음을 비교한다.

* NMOS/PMOS 개수
* MOSFET 종류
* Source/Drain/Gate 연결
* Net connectivity
* Input/Output Pin
* VDD/VSS
* 경우에 따라 Device Parameter

따라서

$$
\boxed{\text{LVS PASS}}
$$

가 되어야 Layout이 Schematic과 전기적으로 동일하게 구현되었다고 판단할 수 있다.

Full-Custom 설계에서는 일반적으로

$$
\boxed{\text{Schematic}
\rightarrow
\text{Layout}
\rightarrow
\text{DRC}
\rightarrow
\text{LVS}}
$$

순서로 검증한다.

---

# 13. Layout 설계 과정에서의 고찰

CMOS Inverter는 MOSFET 두 개만 사용하는 매우 단순한 회로이지만, Layout을 직접 구현하면 Schematic에서 보이지 않았던 다양한 Physical Design 요소를 확인할 수 있다.

Schematic에서는 단순히

```text
PMOS
  │
NMOS
```

와 같이 표현되는 연결도 실제 Layout에서는 Active, Poly, Contact, Metal, Via, Well, Implant 등의 여러 공정 Layer를 이용하여 구현해야 한다.

특히 **회로도에서 선 하나로 표현되는 net이 Layout에서는 실제 면적과 길이를 가지는 물리적 배선이라는 점**이 중요하다.

이 때문에 Layout에서는 다음을 동시에 고려해야 한다.

$$
\boxed{
\text{Connectivity}
+
\text{Area}
+
\text{Parasitic}
+
\text{Design Rule}
+
\text{Reliability}
}
$$

---

# 14. Metal Layer 사용에 대한 고찰

Layout을 처음 설계할 때는 배선을 쉽게 연결하기 위해 상위 Metal layer를 사용하는 것이 편리할 수 있다.

하지만 작은 logic cell의 local connection에서 필요 이상으로 높은 Metal을 사용하면 layer transition을 위해 추가적인 Via가 필요하게 된다.

이는

* Via resistance 증가
* Layout 복잡도 증가
* DRC 조건 증가
* Routing resource 낭비
* LVS debugging 복잡도 증가

등으로 이어질 수 있다.

반대로 상위 Metal은 일반적으로 긴 Global signal, Clock, Power routing 등에서는 낮은 저항과 routing flexibility 때문에 매우 유용하다.

따라서 중요한 것은 **무조건 낮은 Metal을 사용하는 것이 아니라 신호의 특성과 배선 거리에 적절한 Metal layer를 선택하는 것**이다.

이를 다음과 같이 정리할 수 있다.

> **Local routing에는 필요한 수준의 낮은 Metal을 사용하고, 상위 Metal은 장거리 및 Global routing을 위해 효율적으로 활용한다.**

---

# 15. Schematic과 Layout의 차이에 대한 고찰

Schematic 설계에서는 주로 회로의 논리적/전기적 동작에 집중한다.

반면 Layout에서는 동일한 회로를 실제 반도체 공정 구조로 변환해야 하기 때문에 훨씬 많은 물리적 요소를 고려해야 한다.

| Schematic  | Layout                      |
| ---------- | --------------------------- |
| 논리적 연결     | 물리적 연결                      |
| 이상적인 Wire  | 저항/기생성분을 가진 Metal           |
| MOS Symbol | 실제 Active/Poly/Implant/Well |
| Node 연결 중심 | Contact/Via까지 고려            |
| 기능 중심      | 기능 + 면적 + 성능 + 공정성          |
| Simulation | DRC/LVS/PEX 필요              |

따라서 Layout은 단순히 **Schematic을 그림으로 옮기는 과정이 아니라 실제 제조 가능한 형태로 회로를 구현하고 최적화하는 과정**이라고 볼 수 있다.

---

# 16. 최종 고찰

CMOS Inverter Layout 설계를 통해 Full-Custom IC Design에서 가장 기본적인 transistor-level physical design 과정을 이해할 수 있었다.

특히 설계 과정에서 중요하다고 판단한 점은 다음과 같다.

1. PMOS와 NMOS를 효율적으로 배치하여 배선 길이를 최소화해야 한다.
2. VIN과 VOUT 등의 주요 net은 가능한 단순하게 routing하는 것이 유리하다.
3. PMOS Body와 NMOS Body의 적절한 bias를 위해 Well/Substrate Contact가 필요하다.
4. 불필요한 Contact, Via 및 상위 Metal 사용을 줄이는 것이 Layout 단순화에 도움이 된다.
5. DRC를 통해 Layout의 공정 규칙 만족 여부를 검증해야 한다.
6. LVS를 통해 Schematic과 Layout의 전기적 connectivity가 동일한지 검증해야 한다.
7. 최종적으로는 PEX 및 Post-Layout Simulation을 통해 실제 배선의 parasitic이 회로 성능에 미치는 영향까지 확인할 필요가 있다.

CMOS Inverter는 구조 자체는 단순하지만 **이후 NAND, NOR, XOR, Half Adder, Full Adder, Latch와 같은 더 복잡한 회로의 Layout 설계를 위한 기본 단위**가 된다.

따라서 Inverter Layout을 통해 습득한 **MOS 배치, Well/Body 연결, Contact/Via 구성, Metal routing, DRC/LVS 검증 및 Layout 최적화 방법**은 이후 Full-Custom Digital IC 설계 전반에 공통적으로 적용할 수 있는 핵심적인 기초라고 할 수 있다.
