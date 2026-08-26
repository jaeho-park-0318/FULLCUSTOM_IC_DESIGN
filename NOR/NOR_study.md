# CMOS NOR Gate Layout 설계 및 고찰

## 1. 설계 개요

CMOS NOR Gate는 NAND Gate와 함께 가장 기본적인 CMOS 논리회로 중 하나이며, **PMOS 2개와 NMOS 2개**, 총 4개의 MOSFET으로 구성된다.

2-input NOR의 논리식은

$$
\boxed{Y=\overline{A+B}}
$$

이다.

CMOS 구조에서는 다음과 같이 Pull-Up Network(PUN)와 Pull-Down Network(PDN)를 구성한다.

* **Pull-Up Network(PUN): PMOS 2개 직렬(Series)**
* **Pull-Down Network(PDN): NMOS 2개 병렬(Parallel)**

```text
             VDD
              │
           PMOS A
              │
              X
              │
           PMOS B
              │
             VOUT
          ┌────┴────┐
          │         │
       NMOS A    NMOS B
          │         │
          └────┬────┘
               │
              VSS
```

NAND와 비교하면 Series/Parallel 구조가 서로 반대이다.

| Gate | PMOS       | NMOS         |
| ---- | ---------- | ------------ |
| NAND | Parallel   | Series       |
| NOR  | **Series** | **Parallel** |

---

## 2. NOR Gate 동작

NOR Gate의 Truth Table은 다음과 같다.

| A | B | VOUT |
| - | - | ---- |
| 0 | 0 | 1    |
| 0 | 1 | 0    |
| 1 | 0 | 0    |
| 1 | 1 | 0    |

### A = 0, B = 0

두 PMOS가 모두 ON되고 두 NMOS는 모두 OFF된다.

따라서

$$
VDD\rightarrow P_A\rightarrow P_B\rightarrow VOUT
$$

의 Pull-Up path가 형성되어

$$
VOUT\approx VDD
$$

가 된다.

### 하나 이상의 입력이 HIGH인 경우

해당 입력에 연결된 PMOS는 OFF되고 NMOS는 ON된다.

PMOS는 직렬이므로 하나만 OFF되어도 Pull-Up path가 차단되고, 병렬 NMOS 중 하나가 ON되면 VOUT에서 VSS까지 Pull-Down path가 형성된다.

따라서

$$
VOUT\approx VSS
$$

가 된다.

---

# 3. NOR Layout의 기본 구조

Full-Custom Layout에서는 일반적으로 PMOS를 상단, NMOS를 하단에 배치한다.

```text
══════════════════════ VDD
          │
        PMOS A
          │
          X
          │
        PMOS B
          │
        VOUT
       ┌──┴──┐
     NMOS A NMOS B
       │       │
       └───┬───┘
           │
══════════════════════ VSS
```

Schematic의 전기적 연결 관계를 실제 Layout에서는

* Active
* Poly
* Contact
* Metal
* Via
* Well
* Implant

등의 공정 Layer를 이용하여 구현한다.

---

# 4. PMOS Series 구조와 Diffusion Sharing

NOR Layout에서 중요한 부분은 **PMOS의 직렬 연결**이다.

Schematic에서는

```text
VDD ──[PMOS A]── X ──[PMOS B]── VOUT
```

와 같이 두 PMOS 사이에 내부 node `X`가 존재한다.

Layout에서는 두 PMOS를 서로 인접하게 배치하여 Source/Drain diffusion을 공유할 수 있다.

```text
          A             B
          │             │
          │ Poly        │ Poly
          │             │
──────────│─────────────│──────────
   VDD          X             VOUT
```

여기서 두 PMOS 사이의 diffusion이 내부 node `X`가 된다.

### Diffusion Sharing의 장점

* Layout 면적 감소
* Contact 수 감소
* Metal routing 감소
* Source/Drain junction area 감소 가능
* Parasitic capacitance 감소 가능
* Layout 구조 단순화

NAND에서는 **직렬 NMOS의 diffusion sharing**이 중요했다면 NOR에서는 **직렬 PMOS의 diffusion sharing**이 중요하다.

---

# 5. NMOS Parallel 구조

NOR의 Pull-Down Network에서는 두 NMOS가 병렬로 연결된다.

$$
N_A\parallel N_B
$$

두 NMOS의 Drain은 VOUT에 연결되고 Source는 VSS에 연결된다.

```text
             VOUT
          ┌────┴────┐
          │         │
       NMOS A    NMOS B
          │         │
          └────┬────┘
               │
              VSS
```

따라서 A 또는 B 중 하나만 HIGH가 되어도 해당 NMOS가 ON되어 VOUT을 VSS로 방전할 수 있다.

Layout에서도 두 NMOS가 각각

$$
VOUT\leftrightarrow VSS
$$

의 독립적인 conduction path를 형성하도록 Source/Drain connectivity를 정확하게 구성해야 한다.

---

# 6. 입력 A/B Gate Routing

각 입력은 하나의 PMOS와 하나의 NMOS Gate에 동시에 연결된다.

$$
A\rightarrow P_A,N_A
$$

$$
B\rightarrow P_B,N_B
$$

PMOS와 NMOS를 적절하게 정렬하면 입력 Poly를 수직 방향으로 구성할 수 있다.

```text
          PMOS 영역

       A           B
       │           │
───────│───────────│──────
       │           │
       │           │
───────│───────────│──────
       │           │

          NMOS 영역
```

이러한 배치는 입력 routing을 단순화하고 Layout의 규칙성을 높이는 데 도움이 된다.

다만 Poly는 일반적으로 Metal보다 저항이 크기 때문에 장거리 routing에는 적합하지 않으며, 필요한 위치에서 Contact를 통해 Metal로 전환하는 것이 좋다.

---

# 7. VOUT Routing

PMOS Series Network의 출력과 병렬 NMOS의 Drain이 동일한 `VOUT`에 연결되어야 한다.

```text
PMOS Stack ──────┐
                 │
                 ├──── VOUT
                 │
NMOS A Drain ────┤
NMOS B Drain ────┘
```

출력 node에는

* MOSFET diffusion capacitance
* Metal capacitance
* 다음 stage의 gate capacitance
* Coupling capacitance

등의 parasitic capacitance가 존재한다.

이를 단순화하면

$$
C_L=C_{diffusion}+C_{wire}+C_{gate,next}+\cdots
$$

로 생각할 수 있다.

따라서 VOUT routing은 가능한 한 짧고 단순하게 구성하여 불필요한 parasitic을 줄이는 것이 중요하다.

---

# 8. NAND와 NOR의 Layout 차이

NAND와 NOR를 비교하면 Full-Custom CMOS Layout에서 Series/Parallel 관계의 차이를 명확하게 확인할 수 있다.

### NAND

```text
PMOS → Parallel
NMOS → Series
```

### NOR

```text
PMOS → Series
NMOS → Parallel
```

따라서 diffusion sharing을 중점적으로 고려해야 하는 위치도 달라진다.

| 항목                  | NAND       | NOR            |
| ------------------- | ---------- | -------------- |
| PMOS                | Parallel   | **Series**     |
| NMOS                | Series     | **Parallel**   |
| PUN                 | Parallel   | Series         |
| PDN                 | Series     | Parallel       |
| 주요 Shared Diffusion | NMOS       | **PMOS**       |
| 직렬 저항 문제            | NMOS       | **PMOS**       |
| 영향을 크게 받는 전환        | HIGH → LOW | **LOW → HIGH** |

이러한 차이는 단순한 Layout 형태뿐만 아니라 transistor sizing과 propagation delay에도 영향을 준다.

---

# 9. PMOS Series Resistance

NOR에서는 PMOS 두 개가 직렬로 연결되므로 Pull-Up Network의 유효 저항이 증가한다.

PMOS 하나의 ON resistance를 \(R_P\)라고 단순화하면

$$
R_{PU,NOR}\approx R_P+R_P
$$

따라서

$$
\boxed{R_{PU,NOR}\approx2R_P}
$$

정도로 생각할 수 있다.

출력의 LOW → HIGH transition에서는 PMOS가 load capacitance를 충전해야 한다.

단순한 RC 모델에서는

$$
t_{pLH}\propto R_{PU}C_L
$$

이므로 Pull-Up resistance가 증가하면 LOW → HIGH propagation delay가 증가할 수 있다.

---

# 10. NOR에서 PMOS Sizing이 중요한 이유

일반적으로 MOSFET에서는

$$
\mu_n>\mu_p
$$

이므로 동일한 \(W/L\)에서 NMOS의 drive strength가 PMOS보다 강한 경우가 많다.

그런데 NOR에서는 상대적으로 약한 PMOS가 다시 **직렬로 연결**된다.

따라서 NOR Gate에서는 Pull-Up Network의 drive strength가 약해질 수 있다.

이를 보상하기 위해 PMOS의 Width를 증가시키는 방법을 고려할 수 있다.

MOSFET의 ON resistance는 대략

$$
R_{ON}\propto\frac{L}{W}
$$

이므로

$$
W_P\uparrow
\Rightarrow
R_{ON,P}\downarrow
$$

이다.

하지만 PMOS Width를 증가시키면

* Layout 면적 증가
* Gate capacitance 증가
* Diffusion capacitance 증가
* 이전 stage의 load 증가

라는 trade-off가 발생한다.

따라서 실제 Full-Custom 설계에서는 단순히 PMOS를 크게 만드는 것이 아니라 simulation을 통해 적절한 transistor sizing을 결정해야 한다.

---

# 11. Internal Node와 Parasitic

NOR의 두 직렬 PMOS 사이에는 내부 node가 존재한다.

```text
VDD
 │
PMOS A
 │
 X ───── Cparasitic
 │
PMOS B
 │
VOUT
```

이 node에는 Source/Drain junction에 의한 parasitic capacitance가 존재한다.

Schematic에서는 단순한 내부 연결선으로 보이지만 실제 Layout에서는 물리적인 diffusion 영역이기 때문에 capacitance를 가진다.

따라서 shared diffusion을 compact하게 구성하면 내부 node의 junction area를 줄이는 데 도움이 될 수 있다.

---

# 12. Body 및 Well Connection

PMOS는 일반적으로 N-Well 내부에 위치하므로 N-Well을 적절한 전위에 고정해야 한다.

$$
Body_{PMOS}\rightarrow VDD
$$

NMOS의 P-Substrate 또는 P-Well은

$$
Body_{NMOS}\rightarrow VSS
$$

로 연결한다.

따라서 Layout에서는 적절한 Well/Substrate Contact를 배치해야 한다.

이는

* Body potential 안정화
* Body effect 관리
* Noise 감소
* Latch-up 방지

등의 측면에서 중요하다.

---

# 13. VDD / VSS Routing

일반적인 standard-cell 형태의 Full-Custom Layout에서는 상단에 VDD, 하단에 VSS rail을 구성할 수 있다.

```text
══════════════════════ VDD

        PMOS Network

            VOUT

        NMOS Network

══════════════════════ VSS
```

이러한 구조를 사용하면 여러 logic cell을 수평으로 배치할 때 VDD/VSS를 규칙적으로 연결하기 쉽다.

Power routing에서는 신호 배선보다 큰 전류가 흐를 수 있으므로 Metal width, Contact 및 Via 구성도 함께 고려해야 한다.

---

# 14. Metal Layer 사용에 대한 고찰

NOR와 같은 작은 logic cell의 local routing에서는 필요 이상의 상위 Metal layer 사용을 피하는 것이 효율적이다.

예를 들어 M1/M2에서 충분히 연결할 수 있는 신호를 굳이 M4까지 올리면

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

와 같이 추가적인 layer transition이 필요하다.

불필요한 상위 Metal 사용은

* Via 수 증가
* Via resistance 증가
* Layout complexity 증가
* DRC 조건 증가
* Routing resource 낭비
* LVS debugging 난이도 증가
* 추가 parasitic 발생 가능성

등의 단점이 있다.

따라서 **Local routing에서는 필요한 수준의 Metal을 이용하여 최대한 단순하게 연결하는 것**이 좋다.

그러나 상위 Metal 자체가 나쁜 것은 아니다. Power, Clock, Global Signal, 장거리 interconnect와 같이 낮은 배선 저항이 중요한 경우에는 상위 Metal을 사용하는 것이 오히려 유리할 수 있다.

즉,

> **Metal을 무조건 낮게 사용하는 것이 아니라 배선의 목적과 길이에 맞는 적절한 Metal layer를 선택하는 것이 중요하다.**

---

# 15. Layout 면적 최적화

NOR Layout의 면적을 최적화하기 위해 다음 요소를 고려할 수 있다.

* 직렬 PMOS의 Diffusion Sharing
* PMOS/NMOS의 적절한 배치
* 입력 Poly 정렬
* VOUT routing 최소화
* 불필요한 Contact/Via 제거
* 불필요한 Metal layer 사용 감소
* Well/Substrate Contact의 효율적인 배치
* PDK의 Minimum Width/Spacing 활용

특히 transistor를 먼저 임의로 배치하고 이후 Metal로 연결하는 방식보다는 **Schematic connectivity를 고려하여 transistor ordering을 먼저 결정하는 것이 효율적**이다.

---

# 16. DRC 및 LVS

Layout 작성 후에는 DRC와 LVS를 통해 설계를 검증한다.

## DRC

DRC에서는 Layout이 PDK에서 정의한 제조 규칙을 만족하는지 확인한다.

대표적으로

* Minimum Width
* Minimum Spacing
* Minimum Area
* Enclosure
* Contact/Via Rule
* Well Rule
* Implant Rule

등을 검사한다.

## LVS

LVS에서는 Schematic과 Layout에서 추출된 회로를 비교한다.

NOR에서는 특히

* PMOS 2개
* NMOS 2개
* PMOS Series 연결
* NMOS Parallel 연결
* A/B Gate 연결
* VOUT connectivity
* VDD/VSS connectivity
* Top-level Pin

등을 확인해야 한다.

따라서

$$
\boxed{\text{DRC PASS + LVS PASS}}
$$

를 통해 Layout의 물리적 설계 규칙과 Schematic 대비 전기적 connectivity를 검증할 수 있다.

---

# 17. NAND Layout과의 비교를 통한 고찰

NAND와 NOR Layout을 직접 설계함으로써 CMOS의 Pull-Up/Pull-Down Network가 서로 **Dual 관계**에 있다는 것을 확인할 수 있다.

NAND에서는 NMOS가 직렬이므로 Pull-Down Network의 배치와 diffusion sharing이 중요하다.

반대로 NOR에서는 PMOS가 직렬이므로 Pull-Up Network의 배치와 diffusion sharing이 중요하다.

또한 NOR의 경우 상대적으로 drive strength가 약한 PMOS가 직렬로 연결되기 때문에 transistor sizing과 Pull-Up delay를 더욱 주의해서 고려해야 한다.

이를 통해 논리식의 차이가 단순히 transistor 연결 관계만 바꾸는 것이 아니라,

$$
\text{Topology}
\rightarrow
\text{Layout}
\rightarrow
\text{Parasitic}
\rightarrow
\text{Delay}
$$

까지 영향을 미친다는 것을 확인할 수 있다.

---

# 18. 최종 고찰

CMOS NOR Layout 설계를 통해 Inverter에서 학습한 기본적인 MOS 배치, Body connection, Power routing, Contact/Via 구성에서 더 나아가 **직렬 PMOS와 병렬 NMOS를 이용한 CMOS logic network의 Physical Design**을 구현할 수 있었다.

특히 NOR Layout에서는

$$
\boxed{\text{PMOS = Series,\quad NMOS = Parallel}}
$$

이라는 구조를 정확하게 이해하는 것이 중요하다.

직렬 PMOS에서는 shared diffusion을 활용하여 Layout 면적과 parasitic을 줄일 수 있으며, 동시에 직렬 연결에 따른 Pull-Up resistance 증가와 transistor sizing의 trade-off도 고려해야 한다.

따라서 좋은 NOR Layout은 단순히 DRC/LVS를 통과하는 것에 그치지 않고

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

을 함께 고려하여 설계해야 한다.

NAND와 NOR의 Layout을 비교하여 설계하는 과정은 이후 XOR, Half Adder, Full Adder, Latch와 같은 더 복잡한 회로를 Full-Custom 방식으로 구현하기 위한 중요한 기초가 된다.
