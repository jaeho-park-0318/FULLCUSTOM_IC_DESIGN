# CMOS AND Gate Full-Custom Layout 설계 보고서

## 1. 설계 개요

본 설계에서는 CMOS AND Gate를 Full-Custom 방식으로 구현하였다. AND Gate를 transistor level에서 독립적으로 구성하는 대신, 앞서 설계한 **2-input NAND Gate와 CMOS Inverter를 계층적으로 조합하여 AND Gate를 구현**하였다.

AND Gate의 논리식은 다음과 같다.

\[
Y=A\cdot B
\]

2-input NAND Gate의 출력은

\[
X=\overline{A\cdot B}
\]

이며, 이 출력을 Inverter에 입력하면

\[
Y=\overline{X}
\]

이므로

\[
Y=\overline{\overline{A\cdot B}}=A\cdot B
\]

가 된다.

따라서 AND Gate는 다음과 같이 구성할 수 있다.

\[
\boxed{AND=NAND+Inverter}
\]

전체 회로 구조는 다음과 같다.

    A ─────┐
           │
           ├──── NAND ──── X ──── Inverter ──── Y
           │
    B ─────┘

여기서 `X`는 NAND Gate의 출력이자 Inverter의 입력으로 사용되는 내부 Node이고, Inverter의 출력 `Y`가 최종 AND Gate의 출력이 된다.


## 2. AND Gate 동작 원리

2-input AND Gate의 Truth Table은 다음과 같다.

| A | B | X = NAND Output | Y = AND Output |
|---|---|---|---|
| 0 | 0 | 1 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

입력 A와 B 중 하나 이상이 LOW이면 NAND Gate의 출력은 HIGH가 된다.

\[
A\cdot B=0
\]

이면

\[
X=\overline{A\cdot B}=1
\]

이고, 이 신호가 Inverter를 통과하면

\[
Y=0
\]

이 된다.

반대로 A와 B가 모두 HIGH인 경우

\[
A=B=1
\]

이므로

\[
X=\overline{1\cdot1}=0
\]

이 되고, Inverter를 통과한 최종 출력은

\[
Y=1
\]

이 된다.

따라서 전체 회로는

\[
\boxed{Y=A\cdot B}
\]

의 AND 논리를 수행한다.


## 3. Transistor-Level 구성

AND Gate는 NAND Gate와 Inverter를 조합하여 구성하기 때문에 전체 transistor 수는 두 하위 회로에서 사용하는 MOSFET 수의 합으로 결정된다.

2-input CMOS NAND Gate는

- PMOS 2개
- NMOS 2개

로 구성되어 총 4개의 MOSFET을 사용한다.

NAND의 Pull-Up Network에서는 PMOS가 병렬로 연결되고 Pull-Down Network에서는 NMOS가 직렬로 연결된다.

    PMOS : Parallel
    NMOS : Series

CMOS Inverter는

- PMOS 1개
- NMOS 1개

로 구성되므로 2개의 MOSFET을 사용한다.

따라서 전체 AND Gate의 transistor 수는

\[
4+2=6
\]

으로,

\[
\boxed{\text{Total MOSFET}=6}
\]

이다.


## 4. Hierarchical Design

본 AND Gate Layout에서는 이미 설계하고 검증한 NAND와 Inverter Cell을 재사용하여 상위 AND Cell을 구성하였다.

전체 계층 구조는 다음과 같다.

    AND
    │
    ├── NAND
    │   ├── PMOS
    │   ├── PMOS
    │   ├── NMOS
    │   └── NMOS
    │
    └── Inverter
        ├── PMOS
        └── NMOS

이러한 Hierarchical Design 방식을 이용하면 NAND와 Inverter의 transistor를 상위 Layout에서 다시 처음부터 배치할 필요 없이 이미 검증된 하위 Cell을 하나의 Block으로 사용할 수 있다.

따라서 회로 규모가 커질수록 Cell 재사용을 통해 설계와 검증의 복잡도를 줄일 수 있다.


## 5. AND Gate Layout 구성

AND Gate Layout에서는 NAND Cell과 Inverter Cell을 배치한 후 두 Cell 사이의 내부 신호를 연결하였다.

가장 중요한 연결은

\[
\boxed{\text{NAND VOUT}\rightarrow\text{Inverter VIN}}
\]

이다.

Layout의 논리적 연결은 다음과 같다.

    VIN1 ─────┐
              │
              │    ┌──────────┐
              ├───▶│          │
              │    │   NAND   │──── X ────┐
              ├───▶│          │           │
              │    └──────────┘           ▼
    VIN2 ─────┘                      ┌──────────┐
                                    │ Inverter │──── VOUT
                                    └──────────┘

최종 AND Cell의 Top-Level Pin은 다음과 같이 구성할 수 있다.

- `VIN1`
- `VIN2`
- `VOUT`
- `VDD`
- `VSS`

NAND의 출력과 Inverter 입력 사이의 `X`는 상위 회로 외부에서 사용할 필요가 없는 내부 Net이므로 별도의 Top-Level Pin으로 구성하지 않는다.


## 6. Cell Placement에 대한 고려

NAND와 Inverter 사이에는 반드시 내부 신호 `X`가 연결되어야 하므로 두 Cell의 상대적인 배치가 중요하다.

두 Cell 사이의 거리가 증가하면 NAND 출력과 Inverter 입력을 연결하는 Metal의 길이가 증가한다.

배선 길이가 증가하면 일반적으로

\[
R_{wire}\uparrow
\]

및

\[
C_{wire}\uparrow
\]

가 발생할 수 있다.

따라서 NAND의 Output과 Inverter의 Input이 가능한 가까워지도록 배치하여 내부 배선 길이를 줄이는 것이 효율적이다.

Schematic에서는 `X`가 단순한 Wire이지만 Layout에서는 실제 Metal 구조이므로 저항과 capacitance를 가지는 물리적인 Interconnect라는 점을 고려해야 한다.


## 7. Internal Node X에 대한 고찰

NAND와 Inverter 사이에는

\[
X=\overline{A\cdot B}
\]

인 내부 Node가 존재한다.

이 Node에는 NAND Gate 자체의 출력 parasitic뿐만 아니라 Inverter의 입력 Gate capacitance와 두 Cell 사이의 배선 capacitance가 연결된다.

단순화하면

\[
C_X=C_{NAND,out}+C_{wire}+C_{INV,in}
\]

과 같이 생각할 수 있다.

따라서 NAND와 Inverter 사이의 배선을 불필요하게 길게 만들면 내부 Node의 parasitic이 증가하여 신호 전달 속도에 영향을 줄 수 있다.

두 Cell을 가까이 배치하고 짧은 Metal을 이용하여 연결하는 것이 Layout 측면에서 유리하다.


## 8. VDD 및 VSS Routing

NAND와 Inverter는 동일한 VDD와 VSS를 사용한다.

따라서 두 하위 Cell의 Power Rail을 상위 AND Layout에서 동일한 전원 Net으로 연결해야 한다.

    ═══════════════════════ VDD
          │          │
        NAND        INV
          │          │
          └────X─────┘
          │          │
    ═══════════════════════ VSS

가능하다면 NAND와 Inverter의 VDD/VSS Rail을 동일한 방향과 높이로 정렬하면 상위 Cell에서 전원 Routing을 보다 간단하게 구성할 수 있다.

또한 Layout의 VDD/VSS Pin 이름이 Schematic과 정확하게 일치하도록 설정해야 LVS에서 Pin mismatch가 발생하지 않는다.


## 9. Metal Routing에 대한 고찰

AND Gate 내부의 NAND 출력과 Inverter 입력 사이의 연결은 비교적 짧은 Local Routing이다.

따라서 낮은 Metal에서 충분히 연결할 수 있다면 필요 이상으로 높은 Metal Layer까지 올라갈 필요가 없다.

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

와 같이 불필요하게 높은 Metal까지 올라가면 Via의 수가 증가한다.

이러한 구조는

- Via resistance 증가
- Layout complexity 증가
- DRC 조건 증가
- Routing resource 사용
- LVS debugging 복잡도 증가
- 추가 parasitic 발생 가능성

등의 문제를 만들 수 있다.

따라서 AND와 같은 작은 Logic Cell 내부에서는 **필요한 수준의 Metal Layer만 사용하여 단순하고 짧은 Routing을 구성하는 것**이 효율적이다.

다만 상위 Metal 자체가 나쁜 것은 아니며, 장거리 Global Signal이나 Power/Clock Routing에서는 낮은 저항 등의 장점으로 인해 상위 Metal이 더 적합할 수 있다.


## 10. DRC 검증

Layout을 완성한 후 DRC(Design Rule Check)를 수행하여 실제 공정에서 요구되는 물리적인 설계 규칙을 만족하는지 확인한다.

대표적으로 다음과 같은 항목을 검사한다.

- Minimum Width
- Minimum Spacing
- Minimum Area
- Metal Spacing
- Via Spacing
- Via Enclosure
- Well Rule
- Implant Rule

NAND와 Inverter가 개별적으로 DRC PASS 상태라고 하더라도 상위 AND Layout에서 새롭게 추가한 Routing과 Via가 Design Rule을 위반할 수 있다.

따라서 Top-Level AND Cell에서도 DRC를 다시 수행하여

\[
\boxed{\text{DRC PASS}}
\]

를 확인해야 한다.


## 11. LVS 검증

DRC 이후에는 LVS(Layout Versus Schematic)를 수행하여 Layout과 Schematic의 전기적 연결이 동일한지 확인한다.

AND Gate에서는 특히 다음의 연결 관계가 중요하다.

\[
VIN1\rightarrow NAND
\]

\[
VIN2\rightarrow NAND
\]

\[
NAND_{OUT}\rightarrow INV_{IN}
\]

\[
INV_{OUT}\rightarrow VOUT
\]

또한 두 하위 Cell의 전원은

\[
VDD_{NAND}=VDD_{INV}=VDD
\]

\[
VSS_{NAND}=VSS_{INV}=VSS
\]

로 연결되어야 한다.

하위 NAND와 Inverter가 LVS PASS 상태라고 하더라도 상위 AND Cell에서 Cell 사이의 연결이 잘못되어 있으면 LVS Error가 발생한다.

대표적으로

- Open Net
- Short Net
- Unbound Pin
- Pin Mismatch
- 잘못된 Instance Terminal 연결

등이 발생할 수 있다.

따라서 최종적으로

\[
\boxed{\text{DRC PASS + LVS PASS}}
\]

를 만족해야 AND Layout이 Schematic과 동일하게 구현되었다고 판단할 수 있다.


## 12. NAND와 AND Layout 비교

AND Gate는 NAND Gate의 출력에 Inverter가 하나 추가된 구조이다.

NAND는

\[
Y_{NAND}=\overline{A\cdot B}
\]

이고 AND는

\[
Y_{AND}=\overline{Y_{NAND}}
\]

이므로

\[
Y_{AND}=A\cdot B
\]

가 된다.

| 항목 | NAND | AND |
|---|---|---|
| Logic | \(\overline{A\cdot B}\) | \(A\cdot B\) |
| PMOS | 2 | 3 |
| NMOS | 2 | 3 |
| 총 MOSFET | 4 | 6 |
| Logic Stage | 1 | 2 |
| 추가 회로 | 없음 | Inverter |
| 내부 Stage Node | 없음 | NAND → INV |
| Layout 면적 | 작음 | 상대적으로 증가 |

AND는 NAND보다 Inverter Stage가 하나 추가되기 때문에 transistor 수와 Layout 면적이 증가한다.


## 13. Propagation Delay에 대한 고찰

AND Gate는 NAND와 Inverter의 두 Logic Stage를 거쳐 최종 출력이 결정된다.

따라서 단순화하면 전체 propagation delay는

\[
t_{pd,AND}\approx t_{pd,NAND}+t_{pd,INV}
\]

로 생각할 수 있다.

실제로는 NAND 출력에 연결되는 Inverter의 Gate capacitance와 NAND-Inverter 사이의 배선 parasitic도 영향을 준다.

즉, NAND 단독 회로와 비교하면 AND Gate는 추가적인 Inverter Stage로 인해 논리 깊이와 부하가 증가한다.

따라서 Layout에서는 NAND와 Inverter 사이의 거리를 줄여 내부 Interconnect의 parasitic을 최소화하는 것이 중요하다.


## 14. Layout 면적에 대한 고찰

AND Gate는 NAND와 Inverter를 함께 사용하므로 NAND 단독 Layout보다 면적이 증가한다.

그러나 이미 최적화된 NAND와 Inverter Cell을 재사용하면 transistor를 모두 새롭게 배치하는 것보다 설계 과정이 단순해진다.

상위 AND Layout에서는

- NAND와 Inverter의 적절한 배치
- 두 Cell 사이의 간격 감소
- VDD/VSS Rail 정렬
- 내부 Net Routing 최소화
- 불필요한 Via 제거
- 불필요한 상위 Metal 사용 감소

등을 고려하여 전체 면적을 최적화할 수 있다.

단순히 Cell 사이의 간격을 최소화하는 것이 아니라 PDK에서 요구하는 Minimum Spacing 등의 Design Rule을 만족하는 범위에서 최적화해야 한다.


## 15. Hierarchical Layout 설계의 장점

AND Gate 설계를 통해 검증된 기본 Logic Cell을 재사용하는 Hierarchical Design의 장점을 확인할 수 있다.

하위 NAND와 Inverter Cell이 각각

\[
\text{Schematic}
\rightarrow
\text{Layout}
\rightarrow
\text{DRC}
\rightarrow
\text{LVS}
\]

과정을 통해 검증되어 있다면 상위 AND에서는 transistor 하나하나의 구현보다 **Cell 사이의 연결과 상위 Pin을 중심으로 설계 및 검증**할 수 있다.

이러한 설계 방법은 회로가 복잡해질수록 더욱 중요하다.

    MOSFET
       ↓
    INV / NAND / NOR
       ↓
      AND / OR
       ↓
       XOR
       ↓
    Half Adder
       ↓
    Full Adder
       ↓
    Larger Digital Block

즉, 작은 기본 Cell을 정확하게 설계하고 검증한 후 이를 조합하여 더 큰 회로를 구성함으로써 설계 복잡도를 체계적으로 관리할 수 있다.


## 16. 설계 고찰

이번 AND Gate 설계에서는 새로운 transistor network를 처음부터 구성하기보다 기존에 설계한 NAND와 Inverter를 재사용하였다.

이를 통해 Full-Custom 설계에서도 **Cell Reuse와 Hierarchical Design이 효과적으로 활용될 수 있음**을 확인하였다.

또한 Schematic에서는 NAND와 Inverter 사이가 단순한 Wire로 연결되지만 Layout에서는 실제 Metal을 사용하여 구현해야 하므로 배선에 의한 저항과 기생 capacitance가 추가된다는 점을 고려해야 한다.

따라서 상위 Layout에서는 단순히 논리적으로 연결하는 것에서 끝나는 것이 아니라

\[
\boxed{
\text{Connectivity}
+
\text{Placement}
+
\text{Routing}
+
\text{Parasitic}
+
\text{Area}
}
\]

를 함께 고려해야 한다.

특히 하위 Cell이 각각 DRC/LVS를 통과했더라도 상위 Cell에서 새롭게 생성된 Interconnect와 Pin은 별도의 오류를 발생시킬 수 있으므로 Top-Level에서 다시 DRC와 LVS를 수행하는 것이 중요하다.


## 17. 최종 고찰

CMOS AND Gate Layout은 기존에 설계한 NAND Gate와 Inverter를 계층적으로 조합하여 구현하였다.

논리적으로

\[
\boxed{
Y=\overline{\overline{A\cdot B}}=A\cdot B
}
\]

의 관계를 이용하며, 2-input AND Gate는 총 6개의 MOSFET으로 구성된다.

이번 설계에서는 NAND처럼 새로운 Series/Parallel transistor topology를 구현하는 것보다 **검증된 하위 Logic Cell을 조합하여 상위 회로를 만드는 Hierarchical Full-Custom Design 과정**에 중점을 두었다.

특히 NAND의 출력과 Inverter의 입력을 연결하는 내부 Net을 가능한 짧게 구성하고, 두 Cell의 VDD/VSS Rail을 효율적으로 연결하는 것이 중요하다.

또한 불필요한 상위 Metal과 Via 사용을 줄여 Layout 구조를 단순화하면 면적, parasitic, DRC/LVS debugging 측면에서 유리할 수 있다.

결과적으로 AND Gate 설계를 통해 다음과 같은 설계 흐름을 확인할 수 있었다.

\[
\boxed{
\text{NAND / Inverter 설계 및 검증}
\rightarrow
\text{Cell Reuse}
\rightarrow
\text{Hierarchical Integration}
\rightarrow
\text{Routing Optimization}
\rightarrow
\text{DRC/LVS Verification}
}
\]

이러한 설계 방식은 이후 XOR, Half Adder, Full Adder와 같이 여러 Logic Cell을 조합하여 구현하는 더 복잡한 Full-Custom Digital Circuit의 설계 기반으로 활용할 수 있다.
