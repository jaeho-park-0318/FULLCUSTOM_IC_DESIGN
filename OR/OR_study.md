# CMOS OR Gate Full-Custom Layout 설계 보고서

## 1. 설계 개요

본 설계에서는 CMOS OR Gate를 Full-Custom 방식으로 구현하였다. OR Gate를 transistor level에서 독립적으로 새롭게 구성하는 대신, 앞서 설계하고 검증한 **2-input NOR Gate와 CMOS Inverter를 계층적으로 조합하여 OR Gate를 구현**하였다.

OR Gate의 논리식은 다음과 같다.

\[
Y=A+B
\]

2-input NOR Gate의 출력은

\[
X=\overline{A+B}
\]

이며, 이 출력을 Inverter의 입력으로 연결하면

\[
Y=\overline{X}
\]

이므로

\[
Y=\overline{\overline{A+B}}=A+B
\]

가 된다.

따라서 OR Gate는 다음과 같이 구성할 수 있다.

\[
\boxed{OR = NOR + Inverter}
\]

전체적인 회로 구조는 다음과 같다.

    A ─────┐
           │
           ├──── NOR ──── X ──── Inverter ──── Y
           │
    B ─────┘

여기서 `X`는 NOR Gate의 출력이자 Inverter의 입력이 되는 내부 Node이며, Inverter의 출력 `Y`가 최종적인 OR Gate의 출력이 된다.


## 2. OR Gate 동작 원리

2-input OR Gate의 Truth Table은 다음과 같다.

| A | B | X = NOR Output | Y = OR Output |
|---|---|---|---|
| 0 | 0 | 1 | 0 |
| 0 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 |
| 1 | 1 | 0 | 1 |

입력 A와 B가 모두 LOW인 경우 NOR Gate의 출력 `X`는 HIGH가 된다.

\[
A=0,\quad B=0
\]

\[
X=\overline{0+0}=1
\]

이 신호가 Inverter를 통과하면서 다시 반전되므로

\[
Y=\overline{1}=0
\]

이 된다.

반대로 A 또는 B 중 하나 이상의 입력이 HIGH인 경우 NOR Gate의 출력은 LOW가 된다.

\[
A+B=1
\]

이면

\[
X=0
\]

이고, Inverter를 통과한 최종 출력은

\[
Y=1
\]

이 된다.

따라서 전체 회로는 최종적으로

\[
\boxed{Y=A+B}
\]

의 OR 논리를 정상적으로 수행한다.


## 3. Transistor-Level 구성

본 OR Gate는 NOR Gate와 Inverter의 조합으로 구성되므로 전체 transistor 수는 두 하위 회로의 transistor 수를 합하여 구할 수 있다.

2-input CMOS NOR Gate는 다음과 같이 구성된다.

- PMOS 2개
- NMOS 2개

따라서 총 4개의 MOSFET이 필요하다.

NOR Gate의 Pull-Up Network에서는 PMOS 2개가 직렬로 연결되고 Pull-Down Network에서는 NMOS 2개가 병렬로 연결된다.

    PMOS : Series
    NMOS : Parallel

CMOS Inverter는

- PMOS 1개
- NMOS 1개

로 구성되므로 총 2개의 MOSFET이 필요하다.

따라서 전체 OR Gate에서 사용되는 MOSFET 수는

\[
4+2=6
\]

으로,

\[
\boxed{\text{Total MOSFET}=6}
\]

이다.

전체 transistor-level 구조를 단순화하면 다음과 같이 나타낼 수 있다.

                 VDD
                  │
               PMOS A
                  │
               PMOS B
                  │
                  X
             ┌────┴────┐
             │         │
          NMOS A     NMOS B
             │         │
             └────┬────┘
                  │
                 VSS

                  X
                  │
              Inverter
                  │
                  Y

첫 번째 NOR Stage에서

\[
X=\overline{A+B}
\]

를 생성하고 두 번째 Inverter Stage에서

\[
Y=\overline{X}
\]

를 생성하는 2-stage 구조이다.


## 4. Hierarchical Design 방식

본 OR Gate Layout의 중요한 특징은 이미 설계된 NOR와 Inverter Cell을 재사용하여 상위 회로를 구성했다는 점이다.

Full-Custom 설계라고 해서 모든 상위 회로를 항상 transistor부터 다시 그릴 필요는 없다. 기본 Logic Cell을 transistor level에서 설계하고 DRC/LVS를 통해 정상적으로 검증하였다면 해당 Cell을 하나의 하위 Block으로 사용하여 더 복잡한 회로를 구성할 수 있다.

따라서 본 설계에서는 다음과 같은 계층 구조를 사용하였다.

    OR
    │
    ├── NOR
    │   ├── PMOS
    │   ├── PMOS
    │   ├── NMOS
    │   └── NMOS
    │
    └── Inverter
        ├── PMOS
        └── NMOS

이러한 방식을 **Hierarchical Design**이라고 할 수 있다.

Hierarchical Design을 사용하면 이미 검증된 하위 Cell을 반복적으로 사용할 수 있기 때문에 회로가 복잡해질수록 설계 및 검증 효율을 높일 수 있다.


## 5. OR Gate Layout 구성

OR Gate Layout에서는 NOR Cell과 Inverter Cell을 배치한 후 두 Cell 사이의 신호와 전원을 연결하였다.

가장 중요한 내부 연결은

\[
\boxed{\text{NOR VOUT}\rightarrow\text{Inverter VIN}}
\]

이다.

Layout의 논리적 연결을 나타내면 다음과 같다.

    VIN1 ─────┐
              │
              │    ┌─────────┐
              ├───▶│         │
              │    │   NOR   │──── X ────┐
              ├───▶│         │           │
              │    └─────────┘           ▼
    VIN2 ─────┘                     ┌──────────┐
                                   │ Inverter │──── VOUT
                                   └──────────┘

NOR Gate의 출력과 Inverter의 입력 사이에 형성되는 내부 Net은 최종 OR Gate의 외부 Pin으로 노출될 필요가 없으며, 두 Cell을 연결하기 위한 내부 신호로 사용된다.

최종적으로 OR Gate의 Top-Level Pin은 다음과 같이 구성할 수 있다.

- `VIN1`
- `VIN2`
- `VOUT`
- `VDD`
- `VSS`


## 6. Cell Placement에 대한 고려

NOR와 Inverter를 연결할 때 두 Cell의 상대적인 배치가 중요하다.

NOR 출력은 반드시 Inverter 입력으로 전달되어야 하므로 두 Cell을 지나치게 멀리 배치하면 내부 Net의 배선 길이가 증가한다.

배선 길이가 증가하면

\[
R_{wire}\uparrow
\]

및

\[
C_{wire}\uparrow
\]

가 발생할 수 있다.

따라서 전체 interconnect delay 역시 증가할 수 있다.

단순한 RC 모델을 적용하면 배선에 의한 지연은 대략

\[
t_{delay}\propto RC
\]

관계로 이해할 수 있다.

따라서 NOR의 Output Pin과 Inverter의 Input Pin 사이의 거리를 고려하여 두 Cell을 배치하고, 가능한 한 짧은 경로로 연결하는 것이 효율적이다.


## 7. Internal Node X에 대한 고찰

OR Gate에는 NOR와 Inverter 사이에 내부 Node `X`가 존재한다.

\[
X=\overline{A+B}
\]

이 Node는 NOR Gate의 출력인 동시에 Inverter의 입력이다.

Schematic에서는 단순한 Wire로 표현되지만 실제 Layout에서는 Metal을 이용하여 구현되는 물리적인 Interconnect이므로 저항과 기생 capacitance를 가진다.

실제 내부 Node의 capacitance에는 대략적으로

\[
C_X=C_{NOR,out}+C_{wire}+C_{INV,in}
\]

과 같은 성분이 포함될 수 있다.

특히 Inverter의 Gate capacitance가 NOR Gate의 출력 부하로 작용한다.

따라서 NOR와 Inverter를 가까이 배치하여 내부 배선 길이를 줄이면 불필요한 wire parasitic을 감소시키는 데 유리하다.


## 8. VDD 및 VSS 연결

NOR와 Inverter는 동일한 VDD/VSS 전원을 사용한다.

따라서 Layout에서는 두 하위 Cell의 Power Rail을 적절하게 연결하여 동일한 전원 Net을 공유하도록 구성해야 한다.

    ═══════════════════════ VDD
          │          │
         NOR        INV
          │          │
          └────X─────┘
          │          │
    ═══════════════════════ VSS

Hierarchical Layout에서는 하위 Cell 각각의 VDD/VSS 연결이 존재하더라도 상위 OR Cell에서 동일한 Power Net으로 정상적으로 인식되는지 확인하는 것이 중요하다.

특히 LVS 과정에서는 Schematic의 `VDD`, `VSS`와 Layout의 Power Pin 이름이 정확하게 일치해야 한다.


## 9. Metal Routing에 대한 고찰

OR Gate는 비교적 작은 Logic Block이므로 NOR와 Inverter 사이의 local signal을 연결하기 위해 필요 이상의 상위 Metal Layer를 사용할 필요는 없다.

예를 들어 낮은 Metal에서 충분히 연결할 수 있는 신호를 높은 Metal까지 올리면

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
     ...

와 같이 여러 번의 Layer Transition이 필요할 수 있다.

불필요한 상위 Metal 사용은

- Via 수 증가
- Via Resistance 증가
- Layout 구조 복잡화
- 추가적인 DRC 조건
- Routing Resource 사용
- LVS Debugging 난이도 증가
- 추가 Parasitic 발생 가능성

등의 단점이 있다.

따라서 NOR와 Inverter 사이의 내부 연결과 같은 Local Routing은 가능한 한 필요한 수준의 Metal Layer에서 단순하게 구성하는 것이 효율적이다.

그러나 높은 Metal 자체가 나쁜 것은 아니다. 긴 Global Signal, Clock, VDD/VSS Power Routing 등에서는 상위 Metal의 낮은 저항과 넓은 Routing Resource가 유리할 수 있다.

따라서 Metal Layer 선택에서는

> **무조건 낮은 Metal을 사용하는 것이 아니라 신호의 길이와 목적에 맞는 적절한 Metal Layer를 선택하는 것이 중요하다.**


## 10. DRC 검증

Layout을 완성한 후에는 DRC(Design Rule Check)를 수행한다.

DRC는 Layout이 사용하고 있는 PDK에서 정의한 실제 반도체 제조 규칙을 만족하는지 확인하는 과정이다.

대표적으로 다음과 같은 항목을 검사한다.

- Minimum Width
- Minimum Spacing
- Minimum Area
- Metal Spacing
- Via Enclosure
- Via Spacing
- Well Rule
- Implant Rule

하위 NOR와 Inverter Cell이 각각 DRC를 통과한 상태라 하더라도 상위 OR Layout에서 새롭게 생성한 Cell 간 Routing이 Design Rule을 위반할 수 있으므로 Top-Level OR에서도 다시 DRC를 수행해야 한다.

최종적으로

\[
\boxed{\text{DRC PASS}}
\]

를 확인하여 물리적인 설계 규칙을 만족하는지 검증한다.


## 11. LVS 검증

DRC 이후에는 LVS(Layout Versus Schematic)를 수행하여 Layout에서 추출된 회로와 Schematic의 회로가 전기적으로 동일한지 확인한다.

OR Gate에서는 특히 다음의 연결 관계가 중요하다.

\[
VIN1\rightarrow NOR
\]

\[
VIN2\rightarrow NOR
\]

\[
NOR_{OUT}\rightarrow INV_{IN}
\]

\[
INV_{OUT}\rightarrow VOUT
\]

또한

\[
VDD_{NOR}=VDD_{INV}=VDD
\]

\[
VSS_{NOR}=VSS_{INV}=VSS
\]

가 되어야 한다.

LVS에서는 단순히 하위 Cell의 개수만 확인하는 것이 아니라 각 Cell의 Terminal이 어떤 Net에 연결되어 있는지 비교한다.

따라서 NOR와 Inverter 자체가 LVS PASS 상태이더라도 상위 OR에서 두 Cell 사이의 연결이 잘못되어 있으면 LVS Error가 발생할 수 있다.

대표적인 오류로는

- Open Net
- Short Net
- Pin Mismatch
- Unbound Pin
- 잘못된 Cell Terminal 연결

등이 있다.

따라서

\[
\boxed{\text{DRC PASS + LVS PASS}}
\]

를 모두 만족해야 최종 OR Layout이 Schematic과 동일하게 구현되었다고 판단할 수 있다.


## 12. NOR 단독 구현과 OR 구현의 차이

NOR Gate에서 OR Gate로 확장하면서 가장 큰 변화는 출력에 Inverter Stage가 하나 추가된다는 것이다.

NOR에서는

\[
Y_{NOR}=\overline{A+B}
\]

이고 OR에서는

\[
Y_{OR}=\overline{Y_{NOR}}
\]

이므로

\[
Y_{OR}=A+B
\]

가 된다.

구조적으로 비교하면 다음과 같다.

| 항목 | NOR | OR |
|---|---|---|
| Logic | \(\overline{A+B}\) | \(A+B\) |
| PMOS | 2 | 3 |
| NMOS | 2 | 3 |
| 총 MOSFET | 4 | 6 |
| Stage | 1 | 2 |
| 추가 회로 | 없음 | Inverter |
| 내부 Stage Node | 없음 | NOR → INV 연결 |
| Layout 복잡도 | 상대적으로 낮음 | 증가 |

OR Gate는 NOR Gate보다 transistor가 2개 증가하고 하나의 Logic Stage가 추가되므로 면적과 propagation delay 역시 증가할 수 있다.


## 13. Propagation Delay에 대한 고찰

OR Gate는 NOR와 Inverter의 두 Logic Stage로 구성되어 있기 때문에 입력 변화가 최종 출력에 전달되기 위해 두 Stage를 거쳐야 한다.

단순화하면 전체 delay는

\[
t_{pd,OR}\approx t_{pd,NOR}+t_{pd,INV}
\]

로 생각할 수 있다.

실제로는 NOR 출력에 연결된 Inverter의 Gate capacitance, NOR와 Inverter 사이의 Interconnect parasitic 및 최종 Load capacitance 등의 영향을 함께 받는다.

따라서 OR Gate의 Layout에서는 단순히 논리적으로 올바르게 연결하는 것뿐만 아니라 두 Stage 사이의 배선을 짧게 구성하여 불필요한 parasitic을 감소시키는 것도 중요하다.


## 14. Layout 면적에 대한 고찰

OR Gate는 NOR와 Inverter 두 개의 Cell을 사용하기 때문에 NOR 단독 Layout보다 면적이 증가한다.

그러나 이미 최적화된 NOR와 Inverter Layout을 재사용하면 각 transistor를 다시 배치하는 것보다 설계 과정이 단순해진다.

상위 OR Layout에서는

- NOR와 Inverter 사이의 간격 최소화
- Power Rail 정렬
- 내부 Net 배선 최소화
- 불필요한 Via 제거
- 불필요한 상위 Metal 사용 감소

등을 고려하여 전체 Cell의 면적을 최적화할 수 있다.

단, 면적을 지나치게 줄이는 과정에서 DRC Minimum Spacing 등의 Rule을 위반하지 않도록 주의해야 한다.


## 15. Hierarchical Layout 설계의 장점

OR Gate 설계를 통해 Hierarchical Layout의 장점을 확인할 수 있었다.

하위 NOR와 Inverter가 이미 각각

\[
\text{Schematic}\rightarrow\text{Layout}\rightarrow\text{DRC}\rightarrow\text{LVS}
\]

과정을 통해 검증되었다면 상위 OR에서는 각 transistor의 연결을 처음부터 다시 검증하는 것보다 **Cell 사이의 Connectivity를 중심으로 설계 및 검증**할 수 있다.

이 방식은 회로 규모가 증가할수록 더욱 중요해진다.

예를 들어 이후의 회로는 다음과 같이 검증된 Cell을 조합하여 구성할 수 있다.

    MOSFET
       ↓
    INV / NAND / NOR
       ↓
      XOR / OR
       ↓
    Half Adder
       ↓
    Full Adder
       ↓
    Larger Digital Block

따라서 기본 Logic Cell을 정확하게 설계하고 검증하는 것은 이후 복잡한 Full-Custom Digital Circuit 설계의 기반이 된다.


## 16. 설계 고찰

이번 OR Gate 설계에서는 새로운 transistor topology를 처음부터 구성하는 대신 기존에 설계한 NOR와 Inverter를 재사용하였다.

이를 통해 Full-Custom 설계에서도 **기본 Cell의 재사용과 Hierarchical Design이 중요하다**는 점을 확인할 수 있었다.

또한 하위 Cell이 정상적으로 검증되어 있더라도 상위 Cell에서 발생하는 Routing Error, Pin Error, Open/Short 등의 문제는 별도로 검증해야 한다는 점을 알 수 있었다.

특히 NOR와 Inverter 사이의 내부 Node는 Schematic에서는 단순한 Wire이지만 Layout에서는 실제 Metal 구조로 구현되기 때문에 배선 길이와 Via 사용에 따라 parasitic이 발생한다.

따라서 Layout에서는 단순히

\[
\text{Correct Connectivity}
\]

만 고려하는 것이 아니라

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


## 17. 최종 고찰

CMOS OR Gate Layout은 기존에 설계한 NOR Gate와 Inverter를 조합하여 구현하였다.

논리적으로

\[
\boxed{
Y=\overline{\overline{A+B}}=A+B
}
\]

관계를 이용하였으며, 총 6개의 MOSFET으로 구성된다.

본 설계는 NOR나 NAND와 같이 새로운 Series/Parallel transistor network 자체를 학습하는 것보다는 **검증된 기본 Logic Cell을 재사용하여 상위 Logic Cell을 구성하는 Hierarchical Full-Custom Design 과정**에 의미가 있다.

특히 NOR 출력과 Inverter 입력 사이의 내부 Net을 정확하게 연결하고, 두 Cell을 효율적으로 배치하여 배선 길이와 불필요한 Via를 줄이는 것이 중요하다.

또한 하위 Cell이 DRC/LVS를 통과했다고 하더라도 상위 OR Layout에서 새롭게 형성되는 Interconnect와 Top-Level Pin에 대해서는 다시 DRC와 LVS를 수행해야 한다.

결과적으로 OR Gate 설계를 통해

\[
\boxed{
\text{Verified Basic Cell}
\rightarrow
\text{Hierarchical Integration}
\rightarrow
\text{Routing Optimization}
\rightarrow
\text{DRC/LVS Verification}
}
\]

이라는 Full-Custom 설계 흐름을 확인할 수 있었다.

이는 이후 XOR, Half Adder, Full Adder 및 Latch와 같이 여러 Logic Cell을 조합하여 구현하는 더 복잡한 회로의 Layout 설계에 활용할 수 있는 기본적인 설계 방법이다.
