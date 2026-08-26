# CMOS NAND / NAND3 Gate Layout 설계 및 고찰

## 1. 설계 개요

CMOS NAND Gate는 **Pull-Up Network(PUN)**와 **Pull-Down Network(PDN)**가 상보적으로 구성되는 기본적인 CMOS 논리회로이다.

이번 설계에서는 **2-input NAND(NAND2)**와 **3-input NAND(NAND3)**를 Full-Custom 방식으로 구현하며, 입력 수 증가에 따른 transistor 구성과 Layout 구조의 변화를 비교하였다.

### NAND2

$$
\boxed{Y=\overline{A\cdot B}}
$$

* PMOS: 2개 병렬
* NMOS: 2개 직렬
* 총 MOSFET: 4개

### NAND3

$$
\boxed{Y=\overline{A\cdot B\cdot C}}
$$

* PMOS: 3개 병렬
* NMOS: 3개 직렬
* 총 MOSFET: 6개

즉 NAND Gate는 입력 수 \(N\)에 대해 기본적으로

$$
\boxed{N\text{개의 PMOS}+N\text{개의 NMOS}}
$$

로 구성된다.

---

# 2. NAND2의 기본 구조

NAND2의 CMOS 구조는 다음과 같다.

```text
                 VDD
                  |
            ┌─────┴─────┐
          PMOS A      PMOS B
            │             │
            └─────┬───────┘
                  │
                VOUT
                  │
               NMOS A
                  │
               NMOS B
                  │
                 VSS
```

Pull-Up Network에서는 두 PMOS가 병렬이고 Pull-Down Network에서는 두 NMOS가 직렬이다.

따라서 두 입력이 모두 HIGH일 때만 VOUT에서 VSS까지 완전한 Pull-Down path가 형성된다.

| A | B | VOUT |
| - | - | ---- |
| 0 | 0 | 1    |
| 0 | 1 | 1    |
| 1 | 0 | 1    |
| 1 | 1 | 0    |

---

# 3. NAND3의 기본 구조

NAND3는 NAND2의 구조를 3개의 입력으로 확장한 것이다.

```text
                    VDD
                     |
          ┌──────────┼──────────┐
          │          │          │
       PMOS A     PMOS B     PMOS C
          │          │          │
          └──────────┼──────────┘
                     │
                   VOUT
                     │
                  NMOS A
                     │
                    X1
                     │
                  NMOS B
                     │
                    X2
                     │
                  NMOS C
                     │
                    VSS
```

Pull-Up Network는

$$
P_A\parallel P_B\parallel P_C
$$

이고 Pull-Down Network는

$$
N_A-N_B-N_C
$$

의 직렬 구조이다.

따라서

$$
A=B=C=1
$$

인 경우에만 모든 NMOS가 ON되어

$$
VOUT\rightarrow VSS
$$

가 된다.

하나의 입력이라도 LOW이면 해당 PMOS가 ON되므로 VOUT은 HIGH가 된다.

| A | B | C | VOUT |
| - | - | - | ---- |
| 0 | 0 | 0 | 1    |
| 0 | 0 | 1 | 1    |
| 0 | 1 | 0 | 1    |
| 0 | 1 | 1 | 1    |
| 1 | 0 | 0 | 1    |
| 1 | 0 | 1 | 1    |
| 1 | 1 | 0 | 1    |
| 1 | 1 | 1 | 0    |

---

# 4. NAND2와 NAND3 Layout 비교

NAND2와 NAND3의 가장 큰 차이는 입력 수 증가에 따라 MOSFET의 개수와 NMOS stack의 길이가 증가한다는 것이다.

| 항목           |      NAND2 |      NAND3 |
| ------------ | ---------: | ---------: |
| 입력           |          2 |          3 |
| PMOS         |          2 |          3 |
| NMOS         |          2 |          3 |
| 총 MOSFET     |          4 |          6 |
| PMOS 구조      | 2 Parallel | 3 Parallel |
| NMOS 구조      |   2 Series |   3 Series |
| NMOS 내부 Node |          1 |          2 |
| Layout 복잡도   |         낮음 |   상대적으로 높음 |
| Pull-down 저항 |         증가 |       더 증가 |

NAND3에서는 NMOS가 하나 더 추가되기 때문에 **transistor 배치와 diffusion sharing의 중요성이 더욱 커진다.**

---

# 5. NMOS의 Series 구조와 Diffusion Sharing

NAND Layout에서 중요한 최적화 방법 중 하나가 **Diffusion Sharing**이다.

NAND2의 NMOS는

```text
VOUT ──[N_A]── X ──[N_B]── VSS
```

와 같이 연결된다.

두 NMOS를 인접하게 배치하면 중간 node `X`에 해당하는 Source/Drain diffusion을 공유할 수 있다.

NAND3에서는

```text
VOUT ──[N_A]── X1 ──[N_B]── X2 ──[N_C]── VSS
```

가 된다.

따라서 3개의 NMOS를 연속적으로 배치하여

```text
Active ===================================

        A          B          C
        │          │          │
        │Poly      │Poly      │Poly
        │          │          │
────────│──────────│──────────│────────
 VOUT       X1         X2        VSS
```

와 같은 형태로 구현할 수 있다.

이를 통해 별도의 Metal routing 없이 transistor 사이의 직렬 연결을 diffusion 자체로 구현할 수 있다.

### Diffusion Sharing의 장점

* Cell 면적 감소
* Contact 수 감소
* Metal routing 감소
* Source/Drain junction 면적 감소 가능
* Parasitic capacitance 감소 가능
* Layout 구조 단순화

특히 NAND3처럼 직렬 transistor 수가 증가할수록 diffusion sharing을 고려한 배치의 효과가 커진다.

---

# 6. PMOS Parallel Network

NAND2에서는 두 PMOS가 병렬이고 NAND3에서는 세 PMOS가 병렬이다.

NAND3의 경우

$$
S_{P_A}=S_{P_B}=S_{P_C}=VDD
$$

$$
D_{P_A}=D_{P_B}=D_{P_C}=VOUT
$$

이 되도록 연결해야 한다.

```text
          VDD
     ┌─────┼─────┐
     │     │     │
    PA    PB    PC
     │     │     │
     └─────┼─────┘
          VOUT
```

직렬 NMOS와 달리 PMOS는 병렬이기 때문에 단순히 하나의 연속 diffusion으로 배치했다고 해서 모든 연결이 자동으로 병렬이 되는 것은 아니다.

따라서 각 Source/Drain이 **VDD 또는 VOUT 중 어느 net에 연결되는지**를 확인하면서 배치 및 Metal routing을 구성해야 한다.

---

# 7. 입력 A, B, C의 Gate Routing

NAND3에서는 각각의 입력이 PMOS와 NMOS 한 쌍의 Gate를 제어한다.

$$
A\rightarrow P_A,N_A
$$

$$
B\rightarrow P_B,N_B
$$

$$
C\rightarrow P_C,N_C
$$

PMOS와 NMOS를 적절하게 정렬하면 각 입력에 해당하는 Poly를 수직으로 배치할 수 있다.

```text
             PMOS 영역

        A       B       C
        │       │       │
        │       │       │
────────│───────│───────│────────
        │       │       │
        │       │       │
────────│───────│───────│────────
        │       │       │

             NMOS 영역
```

이러한 구조는 Layout의 규칙성을 높이고 입력 routing을 단순화하는 데 유리하다.

다만 Poly는 Metal보다 일반적으로 저항이 크기 때문에 장거리 routing에는 적합하지 않다. 필요한 지점에서 Poly Contact를 통해 Metal로 연결하는 것이 바람직하다.

---

# 8. VOUT Routing

NAND2와 NAND3 모두 PMOS network와 NMOS network가 만나는 지점이 VOUT이다.

$$
VOUT=PUN_{\text{output}}=PDN_{\text{output}}
$$

NAND3에서는 세 PMOS의 공통 Drain과 NMOS stack의 최상단 Drain을 연결해야 한다.

```text
PMOS A Drain ──┐
PMOS B Drain ──┼──── VOUT
PMOS C Drain ──┤
NMOS A Drain ──┘
```

출력 node에는 diffusion capacitance, wire capacitance 및 다음 logic gate의 gate capacitance 등이 존재한다.

따라서 VOUT routing은 가능한 한 짧고 단순하게 구성하는 것이 바람직하다.

---

# 9. NAND3의 Stack Effect

NAND3에서는 NMOS 3개가 직렬로 연결되므로 **transistor stacking**이 NAND2보다 강하게 나타난다.

OFF 상태에서 직렬 transistor 사이의 내부 node 전압이 변화하면서 각 MOSFET의 실제 \(V_{GS}\), \(V_{DS}\), body effect 조건이 달라질 수 있다.

이러한 stack effect는 일반적으로 leakage current 감소에 도움이 될 수 있다.

즉,

$$
\text{Series OFF MOS 증가}
$$

에 따라 단일 OFF MOS에 비해 leakage가 감소하는 효과를 기대할 수 있다.

그러나 반대로 ON 상태에서는 직렬 MOSFET의 수가 증가하므로 Pull-Down path의 저항이 증가하는 문제가 발생한다.

---

# 10. NAND3의 Pull-Down Resistance

NAND2에서 NMOS 하나의 ON resistance를 단순히 \(R_N\)이라고 하면

$$
R_{PD,NAND2}\approx2R_N
$$

으로 볼 수 있다.

NAND3에서는

$$
R_{PD,NAND3}\approx3R_N
$$

이 된다.

따라서 동일한 transistor size를 사용한다면 대체로

$$
R_{PD,NAND3}>R_{PD,NAND2}>R_{PD,INV}
$$

가 된다.

Pull-down resistance가 증가하면 출력 capacitance를 방전하는 데 더 많은 시간이 필요하다.

RC 관점에서 propagation delay를 단순화하면

$$
t_{pd}\propto R_{eq}C_L
$$

이므로 NAND3의 Falling Delay가 증가할 수 있다.

---

# 11. Transistor Sizing에 대한 고찰

NAND3에서 NMOS stack의 증가된 저항을 보상하기 위해 NMOS의 Width를 증가시키는 방법을 사용할 수 있다.

MOSFET의 ON resistance는 대략

$$
R_{ON}\propto\frac{L}{W}
$$

이므로

$$
W\uparrow
\Rightarrow
R_{ON}\downarrow
$$

이다.

매우 단순한 1차 근사로 Inverter NMOS의 width를 \(W_N\)이라고 하면 NAND2의 NMOS를 각각 약

$$
2W_N
$$

NAND3의 NMOS를 각각 약

$$
3W_N
$$

수준으로 만드는 접근을 생각할 수 있다.

하지만 이는 어디까지나 **등가 저항을 맞추기 위한 단순 근사**이며 실제 sizing은 PDK 모델을 사용한 simulation으로 결정해야 한다.

Width를 크게 하면 동시에

* Gate capacitance 증가
* Diffusion capacitance 증가
* Cell area 증가
* 이전 stage의 load 증가

가 발생하기 때문이다.

따라서

$$
\boxed{\text{Speed ↔ Power ↔ Area}}
$$

사이의 trade-off를 고려해야 한다.

---

# 12. NAND3에서 증가하는 Parasitic

NAND3에서는 NAND2보다 transistor가 많기 때문에 기생성분도 증가한다.

특히 NMOS stack 내부에

$$
X_1,\;X_2
$$

두 개의 internal node가 존재한다.

각 node에는 Source/Drain junction에 의한 parasitic capacitance가 존재한다.

```text
VOUT
 │
N_A
 │
X1 ── Cparasitic
 │
N_B
 │
X2 ── Cparasitic
 │
N_C
 │
VSS
```

따라서 입력 수가 증가할수록 단순히 transistor 개수만 증가하는 것이 아니라 **내부 node 및 parasitic까지 증가**한다는 점을 고려해야 한다.

---

# 13. NAND3 Layout의 면적 최적화

NAND3는 6개의 transistor가 필요하기 때문에 NAND2보다 Layout 면적이 증가한다.

그러나 단순히 NAND2에 transistor 하나씩을 추가하는 방식보다 transistor ordering과 diffusion sharing을 고려하면 면적 증가를 줄일 수 있다.

중요한 Layout 최적화 요소는 다음과 같다.

* NMOS 3개의 연속적인 diffusion sharing
* PMOS 배치 최적화
* A/B/C Poly 정렬
* VOUT routing 최소화
* VDD/VSS rail 규칙성 유지
* 불필요한 Contact/Via 감소
* 불필요한 Metal layer 사용 감소
* PDK minimum spacing 활용

즉 NAND3에서는 **transistor를 먼저 배치하고 나중에 억지로 연결하는 것보다 connectivity를 고려하여 transistor ordering을 먼저 결정하는 것**이 중요하다.

---

# 14. Metal Layer 사용에 대한 고찰

NAND2와 NAND3는 모두 비교적 작은 logic cell이므로 local connection을 위해 필요 이상으로 높은 Metal layer를 사용하는 것은 효율적이지 않을 수 있다.

예를 들어

```text
M1
 │ Via1
M2
 │ Via2
M3
 │ Via3
M4
```

와 같이 상위 Metal까지 올라가면 layer transition에 필요한 Via 수가 증가한다.

불필요한 상위 Metal 사용은 다음과 같은 문제를 발생시킬 수 있다.

* Via resistance 증가
* Layout complexity 증가
* DRC 조건 증가
* Routing resource 낭비
* LVS debugging 복잡도 증가
* 추가 parasitic 발생 가능

따라서 **필요한 수준까지만 Metal을 사용하여 routing하는 것**이 중요하다.

그러나 상위 Metal 자체가 나쁜 것은 아니다. Power, Clock, 장거리 Global signal 등에서는 상위 Metal이 오히려 유리할 수 있다.

결국

> **Local routing은 필요한 수준의 낮은 Metal을 이용하여 단순하게 구성하고, 상위 Metal은 장거리 및 상위 hierarchy routing을 위해 효율적으로 활용한다.**

라는 원칙으로 정리할 수 있다.

---

# 15. Body 및 Power Connection

NAND2와 NAND3 모두 PMOS Body는 VDD에, NMOS Body는 VSS에 연결하는 것이 기본적이다.

$$
Body_{PMOS}\rightarrow VDD
$$

$$
Body_{NMOS}\rightarrow VSS
$$

따라서 Layout에서는 적절한 Well/Substrate Contact를 배치한다.

또한 cell 상단과 하단에 각각 VDD/VSS rail을 구성하면 이후 여러 logic cell을 배치할 때 전원 routing을 규칙적으로 구성하기 쉽다.

---

# 16. DRC 및 LVS

Layout 완성 후에는 DRC와 LVS를 통해 설계를 검증한다.

### DRC

다음과 같은 물리적인 Design Rule을 검사한다.

* Minimum Width
* Minimum Spacing
* Minimum Area
* Enclosure
* Contact/Via Rule
* Well Rule
* Implant Rule

### LVS

NAND3에서는 특히 다음 사항을 확인해야 한다.

* PMOS 3개
* NMOS 3개
* PMOS 3개가 Parallel인지
* NMOS 3개가 Series인지
* A/B/C가 각각 올바른 Gate에 연결되어 있는지
* VOUT connectivity
* VDD/VSS connectivity
* Body connection
* Top-level Pin 이름

따라서

$$
\boxed{\text{DRC PASS + LVS PASS}}
$$

를 통해 Layout의 제조 규칙과 Schematic 대비 connectivity를 검증할 수 있다.

---

# 17. NAND2 → NAND3 확장을 통한 고찰

NAND2와 NAND3를 함께 설계함으로써 **입력 수가 증가하면 단순히 transistor 개수만 증가하는 것이 아니라 Layout과 회로 성능의 여러 요소가 동시에 변화한다는 점**을 확인할 수 있다.

입력 수가

$$
2\rightarrow3
$$

으로 증가하면

$$
\text{MOSFET 수}:4\rightarrow6
$$

$$
\text{NMOS stack}:2\rightarrow3
$$

$$
\text{Internal node}:1\rightarrow2
$$

가 된다.

그 결과

* Layout area 증가
* NMOS pull-down resistance 증가
* Internal parasitic 증가
* Gate input capacitance 증가 가능
* Routing 복잡도 증가
* Diffusion sharing의 중요성 증가

등의 변화가 발생한다.

특히 NAND3의 NMOS stack은 **Layout 관점에서는 diffusion sharing을 통해 compact하게 만들 수 있다는 장점**이 있지만, **회로 성능 관점에서는 직렬 저항 증가로 인해 Pull-Down 성능이 저하될 수 있다는 trade-off**를 가진다.

따라서 transistor sizing과 physical layout optimization을 함께 고려해야 한다.

---

# 18. 최종 고찰

NAND2 및 NAND3 Layout 설계를 통해 Inverter보다 복잡한 Series/Parallel transistor network를 Full-Custom 방식으로 구현할 수 있었다.

NAND2에서는 2개의 직렬 NMOS를 통해 diffusion sharing의 기본 개념을 적용할 수 있으며, NAND3에서는 이를 3개의 연속적인 NMOS로 확장하면서 **transistor ordering, shared diffusion 및 internal node의 중요성**을 더욱 명확하게 확인할 수 있다.

특히 NAND3 설계를 통해 입력 수 증가가 다음과 같이 연결된다는 것을 확인할 수 있다.

$$
\boxed{
\text{Input 증가}
\rightarrow
\text{Transistor 증가}
\rightarrow
\text{Stack 증가}
\rightarrow
R_{eq}\text{ 및 Parasitic 증가}
\rightarrow
\text{Delay/Area 변화}
}
$$

따라서 좋은 Full-Custom NAND Layout은 단순히 DRC와 LVS를 통과하는 것에서 끝나는 것이 아니라,

$$
\boxed{
\text{Correct Connectivity}
+
\text{Diffusion Sharing}
+
\text{Compact Area}
+
\text{Short Routing}
+
\text{Low Parasitic}
+
\text{Appropriate Sizing}
}
$$

을 함께 고려해야 한다.

NAND2에서 NAND3로의 확장 설계는 이후 더 복잡한 CMOS logic gate와 Adder, Latch 등을 설계하기 위한 중요한 기초가 되며, **논리회로의 topology와 실제 physical layout 사이의 관계를 이해하는 과정**이라는 점에서 의미가 있다.
