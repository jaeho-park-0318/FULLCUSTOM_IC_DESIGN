# 4×4 Unsigned Array Multiplier (MUL4) Full-Custom 설계

## 1. 설계 목적

본 설계에서는 Cadence Virtuoso를 이용하여 **4-bit unsigned multiplication**을 수행하는 `MUL4` 회로를 Full-Custom 방식으로 구현한다.

입력은 두 개의 4-bit unsigned binary number이며,

```text
A<3:0>
B<3:0>
```

출력은 8-bit product이다.

```text
P<7:0>
```

수식으로 표현하면

$$
P=A\times B
$$

이며,

$$
A=A_3A_2A_1A_0
$$

$$
B=B_3B_2B_1B_0
$$

이다.

4-bit unsigned 입력의 최대값은

$$
15_{10}=1111_2
$$

이므로 최대 multiplication 결과는

$$
15\times15=225
$$

이다.

$$
225_{10}=11100001_2
$$

따라서 결과를 표현하기 위해 **8-bit output**이 필요하다.

```text
4-bit × 4-bit → 8-bit
```

최종 `MUL4` symbol의 형태는 다음과 같다.

```text
              ┌───────────────┐
A<3:0> ──────→│               │
              │     MUL4      │──────→ P<7:0>
B<3:0> ──────→│               │
              └───────────────┘
```

---

# 2. 전체 설계 구조

4×4 multiplier는 크게 두 부분으로 구성한다.

1. **Partial Product Generation**
2. **Partial Product Addition**

전체 구조는 다음과 같다.

```text
A<3:0>, B<3:0>
       │
       ▼
┌──────────────────────┐
│ Partial Product      │
│ Generation           │
│                      │
│ 16 × MUL1 / AND2     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ HA / FA Addition     │
│ Network              │
└──────────┬───────────┘
           │
           ▼
        P<7:0>
```

본 설계에서 사용하는 주요 하위 cell은 다음과 같다.

```text
INV
NAND2
AND2
XOR2

MUL1
HA
FA

MUL4
```

권장 hierarchy는 다음과 같다.

```text
INV
 │
NAND2
 │
AND2
 │
MUL1
 │
 ├───────────┐
 │           │
HA          FA
 │           │
 └─────┬─────┘
       │
      MUL4
```

---

# 3. MUL1 설계

## 3.1 1-bit multiplication

1-bit multiplication은 AND 연산과 동일하다.

$$
P=A\times B=A\cdot B
$$

따라서 진리표는 다음과 같다.

|  A |  B |  P |
| -: | -: | -: |
|  0 |  0 |  0 |
|  0 |  1 |  0 |
|  1 |  0 |  0 |
|  1 |  1 |  1 |

회로는 다음과 같다.

```text
A ─────┐
       AND2 ─────→ P
B ─────┘
```

Full-Custom CMOS에서는 AND2를 일반적으로

```text
NAND2 → INV
```

구조로 구현할 수 있다.

```text
A ──┐
    NAND2 ── X ── INV ── P
B ──┘
```

## 3.2 MUL1 Pin Label

`MUL1` schematic의 pin은 다음과 같이 지정한다.

### Input

```text
A
B
```

### Output

```text
P
```

### Power

```text
VDD
VSS
```

Symbol은 다음과 같이 만들 수 있다.

```text
        ┌────────┐
A ─────→│        │
        │  MUL1  │────→ P
B ─────→│        │
        └────────┘
```

---

# 4. Partial Product 생성

4-bit 입력을

```text
A<3:0>
B<3:0>
```

로 정의한다.

각 bit는

```text
A<0>
A<1>
A<2>
A<3>

B<0>
B<1>
B<2>
B<3>
```

이다.

각 partial product는 다음과 같이 정의한다.

$$
PP_{ij}=A_iB_j
$$

총 partial product의 개수는

$$
4\times4=16
$$

개이다.

---

# 5. Partial Product Label 정의

Virtuoso schematic에서 내부 net은 다음과 같이 labeling한다.

| A Bit | B Bit | Partial Product Label |
| ----- | ----- | --------------------- |
| A0    | B0    | `PP00`                |
| A1    | B0    | `PP10`                |
| A2    | B0    | `PP20`                |
| A3    | B0    | `PP30`                |
| A0    | B1    | `PP01`                |
| A1    | B1    | `PP11`                |
| A2    | B1    | `PP21`                |
| A3    | B1    | `PP31`                |
| A0    | B2    | `PP02`                |
| A1    | B2    | `PP12`                |
| A2    | B2    | `PP22`                |
| A3    | B2    | `PP32`                |
| A0    | B3    | `PP03`                |
| A1    | B3    | `PP13`                |
| A2    | B3    | `PP23`                |
| A3    | B3    | `PP33`                |

이를 행렬 형태로 보면 다음과 같다.

```text
             B0       B1       B2       B3
          ───────────────────────────────────
A0          PP00     PP01     PP02     PP03

A1          PP10     PP11     PP12     PP13

A2          PP20     PP21     PP22     PP23

A3          PP30     PP31     PP32     PP33
```

각 신호는 실제로 다음 연산을 의미한다.

```text
PP00 = A<0> AND B<0>
PP10 = A<1> AND B<0>
PP20 = A<2> AND B<0>
PP30 = A<3> AND B<0>

PP01 = A<0> AND B<1>
PP11 = A<1> AND B<1>
PP21 = A<2> AND B<1>
PP31 = A<3> AND B<1>

PP02 = A<0> AND B<2>
PP12 = A<1> AND B<2>
PP22 = A<2> AND B<2>
PP32 = A<3> AND B<2>

PP03 = A<0> AND B<3>
PP13 = A<1> AND B<3>
PP23 = A<2> AND B<3>
PP33 = A<3> AND B<3>
```

---

# 6. Virtuoso에서 Partial Product 배치

Schematic 왼쪽 영역에 `MUL1` 16개를 4×4 형태로 배치하면 관리하기 쉽다.

```text
B0 Column

A0 ──┐
     MUL1 ── PP00
B0 ──┘

A1 ──┐
     MUL1 ── PP10
B0 ──┘

A2 ──┐
     MUL1 ── PP20
B0 ──┘

A3 ──┐
     MUL1 ── PP30
B0 ──┘
```

동일한 방식으로 B1, B2, B3에 대한 partial product도 생성한다.

복잡한 배선을 직접 길게 연결하기보다는 각 output에

```text
PP00
PP01
...
PP33
```

net label을 붙여 HA/FA 부분에서 같은 이름을 이용하는 것이 schematic 가독성 측면에서 좋다.

---

# 7. Binary Multiplication의 자리 정렬

Partial product는 모두 같은 자리의 값이 아니다.

실제 multiplication은 다음과 같다.

```text
                  PP30 PP20 PP10 PP00
             PP31 PP21 PP11 PP01   0
        PP32 PP22 PP12 PP02   0     0
   PP33 PP23 PP13 PP03   0     0     0
------------------------------------------------
    P7   P6   P5   P4   P3   P2   P1   P0
```

각 weight별 partial product를 정리하면 다음과 같다.

```text
2^0 : PP00

2^1 : PP10, PP01

2^2 : PP20, PP11, PP02

2^3 : PP30, PP21, PP12, PP03

2^4 : PP31, PP22, PP13

2^5 : PP32, PP23

2^6 : PP33
```

이 값과 이전 column에서 발생한 carry를 HA와 FA로 합산하여 `P<7:0>`을 생성한다.

---

# 8. Half Adder 설계

Half Adder의 입력은 두 개이다.

```text
A
B
```

출력은

```text
SUM
COUT
```

이다.

수식은

$$
SUM=A\oplus B
$$

$$
COUT=A\cdot B
$$

이다.

Schematic은 다음과 같이 구성한다.

```text
         ┌── XOR2 ─────→ SUM
A ───────┤
         │
B ───────┤
         │
         └── AND2 ─────→ COUT
```

권장 pin label:

```text
Input:
A
B

Output:
SUM
COUT

Power:
VDD
VSS
```

---

# 9. Full Adder 설계

Full Adder의 입력은 다음과 같다.

```text
A
B
CIN
```

출력은

```text
SUM
COUT
```

이다.

Sum은

$$
SUM=A\oplus B\oplus C_{IN}
$$

이고 carry는

$$
C_{OUT}=AB+C_{IN}(A\oplus B)
$$

이다.

논리 게이트 기반 구조는 다음과 같다.

```text
A ───┐
     XOR ─── X1 ───┐
B ───┘              XOR ───→ SUM
                    │
CIN ────────────────┘


A ───┐
     AND ─── C1 ───┐
B ───┘             │
                   OR ─────→ COUT
X1 ──┐             │
     AND ─── C2 ───┘
CIN ─┘
```

권장 pin label:

```text
Input:
A
B
CIN

Output:
SUM
COUT

Power:
VDD
VSS
```

---

# 10. MUL4 Addition Network

본 구조에서는 partial product를 세 단계에 걸쳐 합산한다.

```text
Partial Products
       │
       ▼
    Stage 1
       │
       ▼
    Stage 2
       │
       ▼
    Stage 3
       │
       ▼
     P<7:0>
```

---

# 11. P0 생성

가장 낮은 bit인 `P0`는 별도의 adder가 필요 없다.

$$
P_0=PP_{00}
$$

따라서 schematic에서는

```text
PP00 ─────────────→ P<0>
```

로 직접 연결한다.

---

# 12. Stage 1

## 12.1 HA0

### Inputs

```text
PP10
PP01
```

### Outputs

```text
SUM  → P<1>
COUT → C2_A
```

구조:

```text
PP10 ──┐
       HA0 ─────→ P<1>
PP01 ──┘
         │
         └──────→ C2_A
```

---

## 12.2 FA0

### Inputs

```text
PP20
PP11
C2_A
```

### Outputs

```text
SUM  → S2_A
COUT → C3_A
```

```text
PP20 ──┐
PP11 ──┼────→ FA0 ─────→ S2_A
C2_A ──┘
                    │
                    └────→ C3_A
```

---

## 12.3 FA1

### Inputs

```text
PP30
PP21
C3_A
```

### Outputs

```text
SUM  → S3_A
COUT → C4_A
```

```text
PP30 ──┐
PP21 ──┼────→ FA1 ─────→ S3_A
C3_A ──┘
                    │
                    └────→ C4_A
```

---

## 12.4 HA1

### Inputs

```text
PP31
C4_A
```

### Outputs

```text
SUM  → S4_A
COUT → C5_A
```

```text
PP31 ──┐
       HA1 ─────→ S4_A
C4_A ──┘
         │
         └──────→ C5_A
```

---

# 13. Stage 1 전체 연결

```text
PP10 ─┐
      HA0 ──────────────→ P<1>
PP01 ─┘
        │
       C2_A
        ▼
PP20 ─┐
PP11 ─┼→ FA0 ───────────→ S2_A
C2_A ─┘
        │
       C3_A
        ▼
PP30 ─┐
PP21 ─┼→ FA1 ───────────→ S3_A
C3_A ─┘
        │
       C4_A
        ▼
PP31 ─┐
      HA1 ──────────────→ S4_A
C4_A ─┘
        │
       C5_A
```

---

# 14. Stage 2

Stage 2에서는 `B<2>`에 해당하는 partial product들을 Stage 1의 결과에 더한다.

---

## 14.1 HA2

### Inputs

```text
S2_A
PP02
```

### Outputs

```text
SUM  → P<2>
COUT → C3_B
```

```text
S2_A ──┐
       HA2 ─────────→ P<2>
PP02 ──┘
         │
         └──────────→ C3_B
```

---

## 14.2 FA2

### Inputs

```text
S3_A
PP12
C3_B
```

### Outputs

```text
SUM  → S3_B
COUT → C4_B
```

---

## 14.3 FA3

### Inputs

```text
S4_A
PP22
C4_B
```

### Outputs

```text
SUM  → S4_B
COUT → C5_B
```

---

## 14.4 FA4

### Inputs

```text
C5_A
PP32
C5_B
```

### Outputs

```text
SUM  → S5_B
COUT → C6_B
```

---

# 15. Stage 2 전체 연결

```text
S2_A ──┐
       HA2 ─────────────→ P<2>
PP02 ──┘
         │
        C3_B
         ▼
S3_A ──┐
PP12 ──┼→ FA2 ──────────→ S3_B
C3_B ──┘
         │
        C4_B
         ▼
S4_A ──┐
PP22 ──┼→ FA3 ──────────→ S4_B
C4_B ──┘
         │
        C5_B
         ▼
C5_A ──┐
PP32 ──┼→ FA4 ──────────→ S5_B
C5_B ──┘
         │
        C6_B
```

---

# 16. Stage 3

Stage 3에서는 마지막 `B<3>` partial product를 합산하여 최종 출력 `P<3>`~`P<7>`을 만든다.

---

## 16.1 HA3

### Inputs

```text
S3_B
PP03
```

### Outputs

```text
SUM  → P<3>
COUT → C4_C
```

---

## 16.2 FA5

### Inputs

```text
S4_B
PP13
C4_C
```

### Outputs

```text
SUM  → P<4>
COUT → C5_C
```

---

## 16.3 FA6

### Inputs

```text
S5_B
PP23
C5_C
```

### Outputs

```text
SUM  → P<5>
COUT → C6_C
```

---

## 16.4 FA7

### Inputs

```text
C6_B
PP33
C6_C
```

### Outputs

```text
SUM  → P<6>
COUT → P<7>
```

마지막 Full Adder의 carry output이 최종 MSB가 된다.

$$
P_7=FA7.COUT
$$

---

# 17. Stage 3 전체 연결

```text
S3_B ──┐
       HA3 ─────────────→ P<3>
PP03 ──┘
         │
        C4_C
         ▼
S4_B ──┐
PP13 ──┼→ FA5 ──────────→ P<4>
C4_C ──┘
         │
        C5_C
         ▼
S5_B ──┐
PP23 ──┼→ FA6 ──────────→ P<5>
C5_C ──┘
         │
        C6_C
         ▼
C6_B ──┐
PP33 ──┼→ FA7 ──────────→ P<6>
C6_C ──┘
         │
         └──────────────→ P<7>
```

---

# 18. 전체 HA/FA 연결표

Virtuoso에서 MUL4 schematic을 작성할 때 아래 표를 기준으로 각 instance의 pin을 연결한다.

| Instance | A Input | B Input | CIN    | SUM Output | COUT Output |
| -------- | ------- | ------- | ------ | ---------- | ----------- |
| `HA0`    | `PP10`  | `PP01`  | -      | `P<1>`     | `C2_A`      |
| `FA0`    | `PP20`  | `PP11`  | `C2_A` | `S2_A`     | `C3_A`      |
| `FA1`    | `PP30`  | `PP21`  | `C3_A` | `S3_A`     | `C4_A`      |
| `HA1`    | `PP31`  | `C4_A`  | -      | `S4_A`     | `C5_A`      |
| `HA2`    | `S2_A`  | `PP02`  | -      | `P<2>`     | `C3_B`      |
| `FA2`    | `S3_A`  | `PP12`  | `C3_B` | `S3_B`     | `C4_B`      |
| `FA3`    | `S4_A`  | `PP22`  | `C4_B` | `S4_B`     | `C5_B`      |
| `FA4`    | `C5_A`  | `PP32`  | `C5_B` | `S5_B`     | `C6_B`      |
| `HA3`    | `S3_B`  | `PP03`  | -      | `P<3>`     | `C4_C`      |
| `FA5`    | `S4_B`  | `PP13`  | `C4_C` | `P<4>`     | `C5_C`      |
| `FA6`    | `S5_B`  | `PP23`  | `C5_C` | `P<5>`     | `C6_C`      |
| `FA7`    | `C6_B`  | `PP33`  | `C6_C` | `P<6>`     | `P<7>`      |

또한

```text
P<0> = PP00
```

이다.

---

# 19. 최종 Output Label 정리

최종 output의 발생 위치는 다음과 같다.

```text
P<0> = PP00

P<1> = HA0.SUM

P<2> = HA2.SUM

P<3> = HA3.SUM

P<4> = FA5.SUM

P<5> = FA6.SUM

P<6> = FA7.SUM

P<7> = FA7.COUT
```

따라서 최종 output bus는

```text
P<7:0>
```

으로 구성한다.

---

# 20. 내부 Net Label 전체 정리

## Partial Product

```text
PP00
PP01
PP02
PP03

PP10
PP11
PP12
PP13

PP20
PP21
PP22
PP23

PP30
PP31
PP32
PP33
```

## Stage 1

```text
C2_A
S2_A

C3_A
S3_A

C4_A
S4_A

C5_A
```

## Stage 2

```text
C3_B
S3_B

C4_B
S4_B

C5_B
S5_B

C6_B
```

## Stage 3

```text
C4_C
C5_C
C6_C
```

## Output

```text
P<0>
P<1>
P<2>
P<3>
P<4>
P<5>
P<6>
P<7>
```

이와 같이 stage별로 suffix를

```text
_A
_B
_C
```

로 구분하면 schematic을 수정하거나 오류를 찾을 때 편리하다.

---

# 21. MUL4 Top-Level Pin 설정

최종 `MUL4` schematic에서 사용하는 pin은 다음과 같다.

## Input

```text
A<3:0>
B<3:0>
```

## Output

```text
P<7:0>
```

## Power

```text
VDD
VSS
```

이를 symbol로 생성하면 다음 형태가 된다.

```text
                  ┌─────────────────┐
A<3:0> ──────────→│                 │
                  │      MUL4       │──────────→ P<7:0>
B<3:0> ──────────→│                 │
                  └─────────────────┘
```

---

# 22. Virtuoso Schematic 배치 방향

Schematic에서는 계산 흐름을 **왼쪽에서 오른쪽**으로 배치하는 것이 좋다.

```text
┌──────────────────┐
│ Partial Product  │
│                  │
│ 16 × MUL1        │
└────────┬─────────┘
         │
         ▼
     ┌─────────┐
     │ Stage 1 │
     │ HA/FA   │
     └────┬────┘
          │
          ▼
     ┌─────────┐
     │ Stage 2 │
     │ HA/FA   │
     └────┬────┘
          │
          ▼
     ┌─────────┐
     │ Stage 3 │
     │ HA/FA   │
     └────┬────┘
          │
          ▼
       P<7:0>
```

권장 schematic 영역 구분:

```text
LEFT
────────────────────────────────────────────→ RIGHT

MUL1 Array     Stage 1      Stage 2      Stage 3      Output

PP Generation → HA/FA   →   HA/FA   →   HA/FA   →   P<7:0>
```

복잡한 wire는 가능한 한 길게 연결하기보다 **net label을 적극 활용**한다.

---

# 23. Functional Simulation

MUL4는 combinational circuit이므로 Clock이 필요하지 않다.

Testbench에서

```text
A<3:0>
B<3:0>
```

에 logic input을 넣고

```text
P<7:0>
```

을 측정한다.

---

# 24. 기본 Test Vector

## Test 1

```text
A = 0000
B = 1111
```

$$
0\times15=0
$$

예상 결과:

```text
P = 00000000
```

---

## Test 2

```text
A = 0001
B = 0001
```

$$
1\times1=1
$$

예상 결과:

```text
P = 00000001
```

---

## Test 3

```text
A = 0011
B = 0101
```

$$
3\times5=15
$$

예상 결과:

```text
P = 00001111
```

---

## Test 4

```text
A = 0111
B = 1001
```

$$
7\times9=63
$$

예상 결과:

```text
P = 00111111
```

---

## Test 5

```text
A = 1111
B = 1111
```

$$
15\times15=225
$$

예상 결과:

```text
P = 11100001
```

마지막 테스트는 많은 partial product와 carry가 동시에 발생하기 때문에 전체 multiplier의 carry propagation 검증에 유용하다.

---

# 25. Spectre Simulation 방향

초기 functional verification에서는 각 input bit에 `vdc`를 사용할 수 있다.

```text
Logic 0 = 0 V
Logic 1 = VDD
```

예를 들어

```text
A = 0011
```

을 입력하려면

```text
A<3> = 0
A<2> = 0
A<1> = VDD
A<0> = VDD
```

로 설정한다.

`B`도 동일하게 설정한다.

---

# 26. Transient Simulation

Propagation delay를 확인할 경우 `vpulse`를 사용한다.

입력이 변경된 시점과 `P<7:0>`의 출력이 안정화되는 시점을 비교하여 delay를 측정한다.

MUL4 내부에서는 일반적으로

```text
Input
  ↓
MUL1 / AND
  ↓
HA / FA
  ↓
FA
  ↓
FA
  ↓
Output
```

과 같이 여러 논리 단계를 지나기 때문에 최하위 bit보다 높은 bit의 delay가 더 커질 수 있다.

특히 `P<6>`와 `P<7>`을 생성하는 carry path가 critical path 후보가 된다.

---

# 27. Layout 설계 순서

MUL4 전체 layout을 바로 시작하지 않고 하위 cell부터 layout을 완료한다.

```text
INV
 ↓
NAND2 / AND2 / XOR2
 ↓
MUL1
 ↓
HA
 ↓
FA
 ↓
MUL4
```

각 단계에서

```text
Schematic
    ↓
Layout
    ↓
DRC
    ↓
LVS
```

를 완료한 뒤 다음 hierarchy로 올라간다.

---

# 28. MUL4 Layout 구조

4×4 Array Multiplier는 반복 구조가 많기 때문에 Full-Custom layout에 적합하다.

개념적인 floorplan은 다음과 같이 구성할 수 있다.

```text
┌──────────────────────────────────────────┐
│       Partial Product Generation         │
│                                          │
│ MUL1   MUL1   MUL1   MUL1                │
│ MUL1   MUL1   MUL1   MUL1                │
│ MUL1   MUL1   MUL1   MUL1                │
│ MUL1   MUL1   MUL1   MUL1                │
├──────────────────────────────────────────┤
│ Stage 1                                  │
│ HA0   FA0   FA1   HA1                    │
├──────────────────────────────────────────┤
│ Stage 2                                  │
│ HA2   FA2   FA3   FA4                    │
├──────────────────────────────────────────┤
│ Stage 3                                  │
│ HA3   FA5   FA6   FA7                    │
└──────────────────────────────────────────┘
```

실제 layout에서는 물리적 routing 길이를 고려하여 adder를 partial product array 옆에 배치할 수도 있다.

---

# 29. Layout 설계 시 고려사항

## VDD / VSS

각 하위 cell의 VDD/VSS rail 방향과 높이를 통일하면 반복 배치가 쉬워진다.

```text
VDD ─────────────────────────────

        PMOS Region

        NMOS Region

VSS ─────────────────────────────
```

---

## Cell Height

`MUL1`, `HA`, `FA` 등의 cell height를 가능한 한 일정하게 맞추면 array 배치가 쉬워진다.

---

## Diffusion Sharing

동일한 논리 gate 내부에서는 source/drain diffusion을 공유하여 area를 감소시킬 수 있다.

단, hierarchy를 반복 배치할 경우 지나친 cell 간 diffusion 공유보다는 **독립적인 검증 가능한 cell 구조**를 유지하는 편이 프로젝트 진행에는 유리할 수 있다.

---

## Routing

Multiplier 내부에는

```text
A<3:0>
B<3:0>
PPxx
Carry
Sum
```

신호가 많기 때문에 routing congestion을 고려해야 한다.

특히 carry는 인접 FA 사이를 이동하도록 배치하여 wire length를 줄이는 것이 좋다.

---

# 30. DRC / LVS

MUL4 layout을 완성한 후 다음 검증을 수행한다.

## DRC

Design Rule Check를 통해

* Metal spacing
* Poly spacing
* Active spacing
* Via enclosure
* Implant rule
* Well rule

등을 확인한다.

## LVS

Layout과 schematic의 연결이 동일한지 비교한다.

특히 다음 오류를 주의한다.

```text
PP label mismatch
P<7:0> ordering 오류
VDD/VSS 연결 오류
FA CIN/COUT 연결 오류
Bus bit 순서 오류
Floating net
```

---

# 31. 설계 검증 순서 정리

전체 작업 순서는 다음과 같다.

```text
1. INV 설계
   ↓
2. NAND2 / AND2 / XOR2 설계
   ↓
3. MUL1 설계
   ↓
4. HA 설계
   ↓
5. FA 설계
   ↓
6. MUL1 × 16으로 Partial Product 생성
   ↓
7. Stage 1 연결
   ↓
8. Stage 2 연결
   ↓
9. Stage 3 연결
   ↓
10. P<7:0> 구성
   ↓
11. MUL4 Symbol 생성
   ↓
12. Functional Simulation
   ↓
13. Transient Simulation
   ↓
14. MUL1 / HA / FA Layout
   ↓
15. DRC / LVS
   ↓
16. MUL4 Layout
   ↓
17. MUL4 DRC / LVS
   ↓
18. PEX / Post-Layout Simulation
```

---

# 32. 최종 MUL4 구조 요약

본 설계의 최종 specification은 다음과 같다.

```text
Name:
MUL4

Type:
4×4 Unsigned Array Multiplier

Input:
A<3:0>
B<3:0>

Output:
P<7:0>

Partial Products:
16

MUL1:
16

Half Adder:
4

Full Adder:
8
```

주요 출력은

```text
P<0> = PP00
P<1> = HA0.SUM
P<2> = HA2.SUM
P<3> = HA3.SUM
P<4> = FA5.SUM
P<5> = FA6.SUM
P<6> = FA7.SUM
P<7> = FA7.COUT
```

로 생성된다.

내부 계산 흐름은

```text
A<3:0>, B<3:0>
        ↓
16 × 1-bit Multiplication
        ↓
Partial Products
        ↓
Stage 1
        ↓
Stage 2
        ↓
Stage 3
        ↓
P<7:0>
```

이다.

---

# 33. 향후 MAC에서의 활용

완성된 `MUL4`는 이후 MAC의 multiplier block으로 그대로 사용할 수 있다.

```text
A<3:0> ─┐
         │
         ▼
      ┌──────┐
B<3:0>│ MUL4 │
──────→│      │
       └──┬───┘
          │
       P<7:0>
          │
          ▼
    Zero Extension
      8 → 10 bit
          │
          ▼
       ADD10
          │
          ▼
       REG10
          │
          ▼
      ACC<9:0>
```

이후 Dot Product Accelerator에서도 동일한 `MUL4` cell을 4개 사용함으로써 **동일한 multiplier를 기준으로 MAC 구조와 Parallel Dot Product 구조의 Area, Delay, Power 및 Energy를 비교**할 수 있다.
