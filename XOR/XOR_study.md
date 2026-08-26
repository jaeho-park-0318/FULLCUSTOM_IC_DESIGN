# CMOS XOR Gate Full-Custom Layout 설계 보고서

## 1. 설계 개요

본 설계에서는 2-input CMOS XOR(Exclusive OR) Gate를 Full-Custom 방식으로 구현하였다.

XOR Gate는 두 입력이 **서로 다른 논리값을 가질 때 HIGH를 출력**하는 논리회로이며, 논리식은 다음과 같다.

\[
\boxed{Y=A\oplus B}
\]

Boolean 식으로 표현하면

\[
\boxed{Y=\overline{A}B+A\overline{B}}
\]

이다.

즉,

- \(A=0,\ B=1\)
- \(A=1,\ B=0\)

인 경우에만 출력이 HIGH가 된다.

XOR는 Inverter, NAND, NOR와 같은 기본 Logic Gate보다 회로 구조가 복잡하지만, **Adder를 구성하는 핵심 Logic Gate**라는 점에서 중요하다.

특히 이후 Half Adder에서는

\[
SUM=A\oplus B
\]

로 사용되므로 XOR Layout은 Half Adder 및 Full Adder 설계의 중요한 기본 Cell이 된다.


## 2. XOR Gate 동작 원리

2-input XOR Gate의 Truth Table은 다음과 같다.

| A | B | Y |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

두 입력이 같으면 출력은 LOW이고, 두 입력이 다르면 출력은 HIGH이다.

따라서 XOR는 다음과 같이 이해할 수 있다.

\[
A=B\Rightarrow Y=0
\]

\[
A\neq B\Rightarrow Y=1
\]

Boolean 식을 이용하면

\[
Y=\overline{A}B+A\overline{B}
\]

이며, 첫 번째 항 \(\overline{A}B\)는 \(A=0,\ B=1\)인 경우를 나타내고 두 번째 항 \(A\overline{B}\)는 \(A=1,\ B=0\)인 경우를 나타낸다.

두 조건을 OR 연산하면 XOR의 최종 출력이 된다.


## 3. XOR Gate의 회로 구성

XOR는 NAND나 NOR처럼 단순한 하나의 Series/Parallel Network만으로 표현하기 어렵기 때문에 구현 방식에 따라 transistor 구조와 개수가 달라질 수 있다.

본 설계에서는 Schematic에서 정의한 XOR 회로의 transistor connectivity를 기준으로 Layout을 구현하였다.

XOR의 논리적 구조는 다음과 같이 표현할 수 있다.

\[
Y=\overline{A}B+A\overline{B}
\]

따라서 회로 내부에서는 입력 A와 B뿐만 아니라 경우에 따라

\[
\overline{A},\quad\overline{B}
\]

와 같은 반전 신호도 필요하다.

개념적으로는 다음과 같은 구조로 이해할 수 있다.

    A ────────────────┐
    │                 │
    └── INV ── A_bar  │
                      ├── XOR Logic ── Y
    ┌── INV ── B_bar  │
    │                 │
    B ────────────────┘

실제 Full-Custom 설계에서는 선택한 XOR topology에 따라 transistor 수와 내부 node가 달라지므로 **Layout은 반드시 실제 Schematic의 연결 관계를 기준으로 구현해야 한다.**


## 4. XOR Layout 설계

XOR는 Inverter, NAND, NOR보다 transistor와 내부 node가 많기 때문에 Layout의 복잡도가 증가한다.

Layout 설계에서는 단순히 MOSFET을 배치한 후 Metal로 연결하는 것보다 먼저 Schematic을 분석하여

- 어떤 transistor가 Series인지
- 어떤 transistor가 Parallel인지
- 어떤 Source/Drain을 공유할 수 있는지
- 입력 A와 B가 어느 Gate에 연결되는지
- 내부 반전 신호가 어디로 전달되는지
- 최종 VOUT이 어느 node에서 형성되는지

를 파악하는 것이 중요하다.

즉,

\[
\boxed{\text{Schematic Topology 분석}\rightarrow\text{Transistor Placement}\rightarrow\text{Routing}}
\]

순서로 접근하는 것이 효율적이다.


## 5. Transistor Placement에 대한 고찰

XOR Layout에서는 transistor 수가 증가하기 때문에 transistor를 어떤 순서로 배치하는지가 전체 Layout 면적과 Routing 복잡도에 큰 영향을 준다.

잘못된 순서로 transistor를 배치하면 Schematic상으로는 가까운 node가 Layout에서는 멀리 떨어질 수 있다.

이 경우 해당 node를 연결하기 위해 긴 Metal Routing이나 추가 Via가 필요하게 된다.

반대로 서로 직접 연결되는 transistor를 인접하게 배치하면

- Routing 길이 감소
- Via 감소
- Layout 면적 감소
- Parasitic 감소 가능
- Layout 가독성 향상

등의 장점을 얻을 수 있다.

따라서 XOR처럼 복잡한 Logic Gate에서는 **Routing을 시작하기 전에 transistor ordering을 충분히 고려하는 것이 중요하다.**


## 6. Diffusion Sharing

Full-Custom Layout에서는 Series로 연결되는 MOSFET을 인접하게 배치하여 Source/Drain diffusion을 공유할 수 있다.

예를 들어 Schematic에서

    M1 ── X ── M2

와 같이 연결된 두 MOSFET이 존재한다면 Layout에서는

          Gate1       Gate2
            │           │
    ────────│───────────│────────
        Node1      X        Node2

와 같이 두 transistor 사이의 diffusion을 내부 node `X`로 공유할 수 있다.

Diffusion Sharing을 적절히 사용하면

- Active 영역 감소
- Contact 수 감소
- Metal Routing 감소
- Junction capacitance 감소 가능
- 전체 Cell 면적 감소

등의 장점이 있다.

하지만 XOR는 NAND/NOR보다 connectivity가 복잡하기 때문에 **단순히 물리적으로 가까운 transistor의 diffusion을 연결해서는 안 된다.**

반드시 Schematic에서 실제로 동일한 Net에 연결되어 있는 Source/Drain인지 확인해야 한다.


## 7. 입력 A/B Routing

XOR에는 두 개의 입력

\[
A,\quad B
\]

가 존재하며 회로 구조에 따라 동일한 입력이 여러 MOSFET의 Gate를 제어할 수 있다.

또한 XOR 구현 방식에 따라

\[
\overline{A},\quad\overline{B}
\]

와 같은 내부 반전 신호가 사용될 수 있다.

따라서 입력 Routing에서는 동일한 신호가 연결되어야 하는 모든 Gate가 정확하게 연결되었는지 확인해야 한다.

특히 Layout이 복잡해지면서 A와 B가 서로 교차하는 경우가 발생할 수 있으므로 Metal Layer를 적절히 분리하여 Short가 발생하지 않도록 구성해야 한다.

Poly는 MOSFET Gate 형성에 반드시 필요하지만 일반적으로 Metal보다 저항이 크기 때문에 장거리 Routing은 Metal을 이용하는 것이 효율적이다.

따라서 필요한 위치에서 Poly Contact를 이용하여

    Poly
      │
    Contact
      │
    Metal

형태로 신호를 전달할 수 있다.


## 8. VOUT Routing

XOR의 최종 출력은

\[
VOUT=A\oplus B
\]

이다.

VOUT은 다음 Logic Stage를 구동하는 신호이므로 Layout에서 중요한 node 중 하나이다.

출력 node에는 MOSFET의 Source/Drain junction capacitance뿐만 아니라 Metal Routing capacitance와 다음 Stage의 Gate capacitance가 연결된다.

이를 단순화하면

\[
C_L=C_{diffusion}+C_{wire}+C_{gate,next}+\cdots
\]

로 표현할 수 있다.

따라서 VOUT의 diffusion 영역과 배선 길이를 불필요하게 크게 만들면 출력 capacitance가 증가할 수 있다.

가능한 한 VOUT을 형성하는 transistor를 적절히 배치하고 짧은 Metal Routing을 이용하는 것이 유리하다.


## 9. Internal Node에 대한 고찰

XOR는 NAND나 NOR보다 많은 내부 node를 가질 수 있다.

Schematic에서는 이러한 내부 node가 단순한 Wire로 표현되지만 Layout에서는 실제로

- Diffusion
- Contact
- Metal
- Via

등으로 구성되는 물리적인 구조이다.

따라서 각 내부 node에는 저항과 capacitance가 존재한다.

\[
R_{node}\neq0
\]

\[
C_{node}\neq0
\]

내부 node의 배선이 길어지거나 불필요한 Contact/Via가 증가하면 parasitic이 증가하여 XOR의 propagation delay에 영향을 줄 수 있다.

따라서 XOR Layout에서는 외부 입력과 출력뿐만 아니라 **내부 Net의 길이를 최소화하는 것도 중요한 최적화 요소**이다.


## 10. VDD 및 VSS Routing

PMOS Network는 VDD와 연결되고 NMOS Network는 VSS와 연결된다.

일반적으로 Cell 상단에 VDD, 하단에 VSS Rail을 구성하면 Layout을 규칙적으로 만들 수 있다.

    ═══════════════════════ VDD

          PMOS Network

          XOR Logic

          NMOS Network

    ═══════════════════════ VSS

이러한 구조는 이후 XOR를 Half Adder와 같은 상위 Cell에서 사용할 때 다른 Logic Cell과 Power Rail을 정렬하는 데도 유리하다.

또한 PMOS가 위치하는 N-Well의 Body는 VDD에, NMOS의 P-Substrate 또는 P-Well Body는 VSS에 적절하게 연결해야 한다.

\[
Body_{PMOS}\rightarrow VDD
\]

\[
Body_{NMOS}\rightarrow VSS
\]


## 11. Metal Layer 사용에 대한 고찰

XOR는 NAND/NOR보다 내부 연결이 복잡하기 때문에 Routing 과정에서 신호가 서로 교차하는 경우가 많다.

따라서 여러 Metal Layer를 이용하면 Routing을 쉽게 구성할 수 있지만, 단순히 배선이 어렵다는 이유로 필요 이상으로 높은 Metal까지 사용하는 것은 효율적이지 않을 수 있다.

예를 들어

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

처럼 상위 Metal까지 올라가면 추가적인 Via가 필요하다.

불필요한 상위 Metal 사용은

- Via 수 증가
- Via Resistance 증가
- Layout 복잡도 증가
- DRC 조건 증가
- Routing Resource 낭비
- LVS Debugging 난이도 증가
- 추가 Parasitic 발생 가능성

등의 단점이 있다.

따라서 XOR Layout에서는 신호 교차를 해결하면서도 **가능한 한 필요한 수준의 Metal Layer에서 Routing을 완료하는 것**이 중요하다.

즉,

> **상위 Metal을 사용하지 않는 것이 목적이 아니라, Routing에 필요한 만큼만 Metal Layer를 사용하여 불필요한 Via와 배선 복잡도를 줄이는 것이 중요하다.**


## 12. Layout 면적 최적화

XOR는 기본 Logic Gate보다 구조가 복잡하기 때문에 Layout 면적을 줄이기 위해서는 transistor placement와 Routing을 함께 고려해야 한다.

주요 Layout 최적화 요소는 다음과 같다.

- 연결 관계를 고려한 Transistor Ordering
- 가능한 Source/Drain의 Diffusion Sharing
- 내부 Net 배선 길이 최소화
- VOUT Routing 최소화
- 입력 A/B Routing 최적화
- 불필요한 Contact 및 Via 감소
- 불필요한 상위 Metal 사용 감소
- VDD/VSS Rail의 규칙적인 배치
- PDK Minimum Spacing 활용

중요한 것은 단순히 모든 구조를 최대한 가깝게 배치하는 것이 아니다.

\[
\boxed{\text{DRC Rule을 만족하면서 Connectivity와 Area를 함께 최적화}}
\]

하는 것이 Full-Custom Layout의 핵심이다.


## 13. XOR의 Propagation Delay

XOR는 Inverter나 NAND/NOR보다 복잡한 transistor network를 사용하기 때문에 신호가 출력까지 전달되는 경로 역시 복잡할 수 있다.

Propagation delay는 단순화하면

\[
t_{pd}\propto R_{eq}C_L
\]

로 생각할 수 있다.

여기서 \(R_{eq}\)는 입력 상태에 따라 형성되는 Pull-Up 또는 Pull-Down path의 등가 저항이며 \(C_L\)은 출력 node의 전체 Load Capacitance이다.

따라서 Layout에서

- 긴 Metal 배선
- 큰 Diffusion 영역
- 불필요한 Contact/Via
- 큰 출력 node

등이 존재하면 parasitic 증가로 인해 Schematic Simulation보다 실제 Post-Layout 성능이 저하될 수 있다.

이 때문에 Layout 이후에는 PEX(Parasitic Extraction)를 수행하여 Post-Layout Simulation으로 성능을 다시 확인하는 것이 중요하다.


## 14. DRC 검증

XOR Layout을 완성한 후에는 DRC(Design Rule Check)를 수행한다.

DRC에서는 사용한 PDK에서 정의된 물리적인 제조 규칙을 검사한다.

대표적으로

- Minimum Width
- Minimum Spacing
- Minimum Area
- Metal Spacing
- Poly Spacing
- Contact Rule
- Via Enclosure
- Via Spacing
- Well Rule
- Implant Rule

등이 있다.

XOR는 Routing이 복잡하기 때문에 특히 Metal 간 spacing이나 Via enclosure 등의 오류가 발생할 가능성이 높아질 수 있다.

따라서 Layout 수정 후에는 반복적으로 DRC를 수행하여

\[
\boxed{\text{DRC PASS}}
\]

상태를 확인해야 한다.


## 15. LVS 검증

DRC 이후에는 LVS(Layout Versus Schematic)를 수행한다.

LVS는 Layout에서 추출된 transistor와 Net을 Schematic과 비교하여 두 회로가 전기적으로 동일한지 검증한다.

XOR에서는 특히

- PMOS/NMOS 개수
- 각 MOSFET의 Gate 연결
- 입력 A/B 연결
- Internal Net Connectivity
- VOUT 연결
- VDD/VSS 연결
- Top-Level Pin 이름

등을 확인해야 한다.

XOR는 내부 Net이 많기 때문에 Layout에서 하나의 Metal이나 Contact가 빠지면 Open Net이 발생할 수 있으며, 서로 다른 두 배선이 잘못 연결되면 Short가 발생할 수 있다.

대표적인 LVS Error에는

- Open Net
- Short Net
- Bad Net Match
- Device Mismatch
- Pin Mismatch
- Unbound Pin

등이 있다.

따라서 최종적으로

\[
\boxed{\text{DRC PASS + LVS PASS}}
\]

를 만족해야 Layout이 Schematic과 동일한 회로로 구현되었다고 판단할 수 있다.


## 16. 기본 Logic Gate와 XOR Layout 비교

XOR는 앞서 설계한 Inverter, NAND, NOR보다 Layout 복잡도가 높다.

| 항목 | Inverter | NAND/NOR | XOR |
|---|---|---|---|
| 입력 수 | 1 | 2 | 2 |
| 회로 구조 | 매우 단순 | Series/Parallel | 복합 Logic |
| Internal Node | 적음 | 증가 | 상대적으로 많음 |
| Routing 복잡도 | 낮음 | 보통 | 높음 |
| Diffusion Sharing | 단순 | 중요 | 더욱 신중하게 고려 |
| Metal 교차 | 적음 | 증가 | 많아질 수 있음 |
| Layout 난이도 | 낮음 | 보통 | 상대적으로 높음 |

Inverter에서는 기본적인 PMOS/NMOS 배치와 VDD/VSS 연결을 학습할 수 있었고, NAND/NOR에서는 Series/Parallel 구조와 Diffusion Sharing의 중요성을 확인할 수 있었다.

XOR에서는 이러한 요소들을 종합하여 **보다 복잡한 transistor network의 배치와 Routing을 최적화하는 과정**을 학습할 수 있다.


## 17. Half Adder와의 연관성

XOR Gate는 이후 Half Adder 설계에서 직접 사용된다.

Half Adder의 출력은

\[
SUM=A\oplus B
\]

\[
CARRY=A\cdot B
\]

이다.

따라서 앞서 설계한 XOR와 AND Cell을 이용하면

    A ─────┬──── XOR ──── SUM
           │
    B ─────┘

    A ─────┬──── AND ──── CARRY
           │
    B ─────┘

형태로 Half Adder를 구성할 수 있다.

즉, XOR Layout을 정확하게 설계하고 DRC/LVS를 통해 검증해 두면 이후 Half Adder에서는 XOR를 하나의 검증된 하위 Cell로 재사용할 수 있다.

이는

\[
\text{Basic Gate}
\rightarrow
\text{XOR/AND}
\rightarrow
\text{Half Adder}
\rightarrow
\text{Full Adder}
\]

로 이어지는 Hierarchical Full-Custom Design의 기반이 된다.


## 18. 설계 고찰

XOR Gate를 설계하면서 NAND나 NOR보다 transistor 간 연결 관계가 복잡해짐에 따라 **Schematic을 충분히 분석한 후 Layout을 시작하는 것이 중요함**을 확인할 수 있었다.

단순한 Logic Gate에서는 transistor를 배치한 후 Routing을 진행해도 비교적 쉽게 연결할 수 있지만, XOR처럼 내부 Net이 많아지면 transistor의 배치 순서가 좋지 않을 경우 Metal Routing이 복잡해지고 많은 Via 또는 상위 Metal을 사용해야 할 수 있다.

따라서 복잡한 Full-Custom Layout에서는

\[
\boxed{
\text{Schematic 분석}
\rightarrow
\text{Transistor Ordering}
\rightarrow
\text{Diffusion Sharing}
\rightarrow
\text{Placement}
\rightarrow
\text{Routing}
}
\]

의 순서로 접근하는 것이 중요하다.

또한 Schematic에서 단순한 Wire로 표현되는 내부 Node도 실제 Layout에서는 저항과 capacitance를 가지는 물리적인 구조라는 점을 고려해야 한다.

따라서 단순히 DRC/LVS를 통과하는 것에서 끝나는 것이 아니라 배선 길이, diffusion 면적, Via 수, Metal Layer 사용 등을 함께 고려하여 Layout을 최적화해야 한다.


## 19. 최종 고찰

CMOS XOR Gate Layout 설계를 통해 Inverter, NAND, NOR에서 학습한 기본적인 Full-Custom Layout 설계 방법을 보다 복잡한 Logic Gate에 적용할 수 있었다.

XOR의 논리식은

\[
\boxed{Y=A\oplus B=\overline{A}B+A\overline{B}}
\]

이며 두 입력이 서로 다른 경우에만 HIGH를 출력한다.

XOR Layout에서는 단순히 transistor의 개수보다 **각 transistor 사이의 복잡한 Connectivity를 효율적으로 Physical Layout으로 변환하는 과정**이 중요하다.

특히 transistor ordering과 diffusion sharing을 적절히 활용하면 Layout 면적과 Routing 복잡도를 줄일 수 있으며, 내부 Net과 VOUT의 배선을 짧게 구성하면 parasitic을 감소시키는 데 도움이 될 수 있다.

또한 신호 교차를 해결하기 위해 여러 Metal Layer를 사용할 수 있지만, 필요 이상의 상위 Metal과 Via 사용은 Layout 복잡도와 parasitic, DRC/LVS debugging 측면에서 불리할 수 있으므로 적절한 Layer 선택이 필요하다.

결과적으로 XOR Layout 설계를 통해 좋은 Full-Custom Layout은 단순히 Schematic과 동일한 회로를 만드는 것뿐만 아니라

\[
\boxed{
\text{Correct Connectivity}
+
\text{Efficient Placement}
+
\text{Diffusion Sharing}
+
\text{Short Routing}
+
\text{Low Parasitic}
+
\text{Compact Area}
}
\]

를 함께 고려해야 한다는 점을 확인할 수 있었다.

또한 XOR는 Half Adder의 `SUM`을 생성하는 핵심 Logic Cell이므로, 본 설계에서 검증한 XOR Cell을 이후 Half Adder 및 Full Adder의 Hierarchical Layout 설계에 재사용할 수 있다.
