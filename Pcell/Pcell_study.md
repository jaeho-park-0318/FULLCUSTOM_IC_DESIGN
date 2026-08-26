# PCell 설계 및 Stretch / Finger Parameter 적용

## 1. PCell 개요

**PCell(Parameterized Cell)**은 Layout의 크기나 반복 구조를 Parameter로 정의하여, Parameter 값에 따라 Layout 구조가 자동으로 변경되도록 만든 Cell이다.

일반적인 Layout은 Geometry가 고정되어 있지만 PCell에서는 다음과 같은 값을 Parameter로 설정할 수 있다.

```text
length
width
FINGER
```

따라서 하나의 Layout을 이용하여 다양한 크기와 구조의 MOSFET Layout을 생성할 수 있다.

```text
일반 Cell
Geometry → 고정

PCell
Parameter → Geometry 자동 변경
```

이번 실습에서는 Cadence Virtuoso의 PCell 기능을 이용하여 다음 기능을 구현하였다.

- X축 Stretch를 이용한 `length` 조절
- Y축 Stretch를 이용한 `width` 조절
- X축 Repeat를 이용한 `FINGER` 조절
- Y축 Repeat를 이용한 Contact 개수 자동 조절

---

## 2. 기본 Layout 구성

PCell을 생성하기 전에 기준이 되는 MOSFET Layout을 구성하였다.

주요 Layer는 다음과 같다.

```text
Active
Poly
Contact
Metal
```

Poly가 Active 영역을 가로지르도록 배치하고 Source/Drain 영역에 Contact를 형성하였다.

PCell은 이 기본 Layout을 기준으로 Stretch와 Repeat가 이루어지므로 초기 Geometry의 크기와 Design Rule을 정확하게 설정하는 것이 중요하다.

---

## 3. X축 Stretch - Length

MOSFET의 X 방향 Dimension을 조절하기 위해 **Horizontal Stretch**를 설정하였다.

PCell Parameter Summary에서 다음과 같이 설정하였다.

```text
Stretch Type      : Horizontal
Parameter         : length
Stretch Direction : right
Reference         : 0.95
```

`length` 값이 증가하면 Stretch Line을 기준으로 오른쪽 Geometry가 이동하면서 X축 방향의 크기가 변화한다.

```text
            length 증가 →

┌─────────────────────────┐
│ Active                  │
│          │ Poly         │
│          │              │
└─────────────────────────┘
           ↑
      Stretch 기준
```

이를 통해 Layout을 직접 다시 그리지 않고 `length` Parameter만 변경하여 Geometry를 조절할 수 있다.

---

## 4. Y축 Stretch - Width

MOSFET의 Y 방향 크기를 조절하기 위해 **Vertical Stretch**를 설정하였다.

```text
Stretch Type      : Vertical
Parameter         : width
Stretch Direction : up
Reference         : 0.5
```

`width` 값이 증가하면 설정한 Stretch Line을 기준으로 위쪽 방향으로 Geometry가 확장된다.

```text
        ↑
        │ width 증가
        │
    ┌───────┐
    │       │
────┼───────┼────
    │       │
    └───────┘
```

따라서 X축과 Y축 Stretch를 함께 적용하여 `length`와 `width`를 각각 독립적으로 제어할 수 있도록 하였다.

---

## 5. FINGER Parameter

MOSFET에서는 큰 Width를 하나의 긴 Gate로 구현하는 대신 여러 개의 Gate로 분할하는 **Multi-Finger 구조**를 사용할 수 있다.

이를 PCell에서 자동으로 생성하기 위해 `FINGER` Parameter를 추가하였다.

```text
FINGER = 1

S ──│── D
    G
```

FINGER를 증가시키면 다음과 같이 Poly Gate가 반복된다.

```text
FINGER = 3

S ──│── D ──│── S ──│── D
    G        G        G
```

따라서 `FINGER` 값을 변경하는 것만으로 Multi-Finger MOS Layout을 생성할 수 있다.

---

## 6. X축 Repeat

FINGER 구현을 위해 Poly 및 관련 Geometry를 X축 방향으로 반복하도록 설정하였다.

PCell Parameter Summary에서는 다음과 같이 설정하였다.

```text
Objects Repeated in   : X
Stepping Distance     : (2 + length)
Number of Repetitions : FINGER
```

또한 전체 반복 구조의 위치를 결정하기 위해 Parameter가 포함된 수식을 적용하였다.

```text
2 + length + (2 * (FINGER - 1)) + (length * (FINGER - 1))
```

따라서 `FINGER` 값이 증가하면 Poly Gate가 일정한 간격으로 X 방향으로 반복되고 전체 Active 영역 역시 이에 맞게 확장된다.

```text
FINGER 증가
     ↓
Poly Gate 반복
     ↓
Source / Drain 영역 증가
     ↓
전체 Cell의 X 방향 크기 증가
```

---

## 7. Y축 Contact Repeat

`width`가 증가하면 Active 영역의 높이만 증가하는 것이 아니라 Source/Drain Contact 수도 함께 증가해야 한다.

이를 위해 Contact에 Y축 Repeat를 적용하였다.

```text
Number of Y Repetitions:

((width / pcStepY) + 1)
```

따라서 다음과 같은 관계를 갖는다.

```text
width 증가
    ↓
Active 높이 증가
    ↓
Y 방향 Contact 개수 증가
```

이를 통해 Width가 큰 MOS에서도 Source/Drain 영역에 여러 Contact가 자동으로 배치되도록 구성하였다.

---

## 8. Parameter 간 관계

최종 PCell에서는 `length`, `width`, `FINGER` 세 Parameter가 Layout Geometry를 결정한다.

| Parameter | 역할 |
|---|---|
| `length` | X축 Stretch 및 Gate 관련 Dimension 조절 |
| `width` | Y축 Stretch를 통한 MOS Width 조절 |
| `FINGER` | X축 Gate 반복 개수 조절 |
| `width + Y Repeat` | Source/Drain Contact 개수 자동 조절 |

전체적인 관계는 다음과 같다.

```text
length
  │
  └──→ X축 Stretch

width
  │
  ├──→ Y축 Stretch
  └──→ Contact Y Repeat

FINGER
  │
  └──→ Gate X Repeat
```

---

## 9. 최종 PCell 설정

최종적으로 다음 Parameter를 정의하였다.

```text
FINGER
length
width
```

Stretch 설정은 다음과 같다.

```text
Horizontal Stretch
Parameter : length
Direction : right
Reference : 0.95
```

```text
Vertical Stretch
Parameter : width
Direction : up
Reference : 0.5
```

Repeat 기능은 다음과 같이 구성하였다.

```text
X Repeat
→ FINGER 값에 따른 Gate 및 Geometry 반복

Y Repeat
→ width 값에 따른 Contact 반복
```

즉 **Stretch와 Repeat 기능을 결합하여 MOSFET의 크기와 Finger 수를 자동으로 조절할 수 있는 PCell**을 구현하였다.

---

## 10. PCell의 장점

PCell을 사용하면 Parameter가 달라질 때마다 새로운 Layout을 직접 작성할 필요가 없다.

일반적인 방식은 다음과 같다.

```text
W = 1 → Layout 1
W = 2 → Layout 2
W = 3 → Layout 3
```

PCell을 이용하면 하나의 Layout을 기반으로 Parameter만 변경하면 된다.

```text
             PCell
               │
       ┌───────┼───────┐
       ↓       ↓       ↓
     W=1     W=2     W=3
```

따라서 다음과 같은 장점이 있다.

- Layout 재사용성 향상
- 반복적인 Layout 작업 감소
- 설계 시간 단축
- MOS 크기 변경 용이
- Multi-Finger 구조 자동 생성
- Cell Library 관리 효율 향상

---

## 11. PCell 설계 시 주의사항

PCell에서는 특정 Parameter에서만 Layout이 정상적으로 동작해서는 안 된다.

`length`, `width`, `FINGER` 값을 변경해도 다음과 같은 Design Rule을 만족하도록 구성해야 한다.

```text
Minimum Width
Minimum Spacing
Contact Enclosure
Poly Spacing
Active Enclosure
```

특히 Repeat Step을 잘못 설정하면 다음과 같은 문제가 발생할 수 있다.

```text
Step이 너무 작음
→ Geometry Overlap
→ DRC Error

Step이 너무 큼
→ 불필요한 Layout Area 증가

수식 설정 오류
→ FINGER 변경 시 Geometry 위치 불일치
```

따라서 PCell을 완성한 후에는 여러 Parameter 값을 적용하여 Layout이 정상적으로 변화하는지 확인하고 DRC를 수행하는 것이 중요하다.

---

## 12. 설계 고찰

이번 실습에서는 고정된 MOSFET Layout을 설계하는 것에서 나아가 **Layout Geometry 자체를 Parameter화하는 PCell 설계 방법**을 학습하였다.

먼저 `length` Parameter를 이용하여 X축 Horizontal Stretch를 설정하고, `width` Parameter를 이용하여 Y축 Vertical Stretch를 설정하였다.

이후 `FINGER` Parameter와 X축 Repeat 기능을 이용하여 FINGER 값에 따라 Poly Gate가 자동으로 반복되는 Multi-Finger 구조를 구현하였다.

또한 MOS Width 증가에 맞추어 Source/Drain Contact가 Y축 방향으로 자동 증가하도록 Repeat 기능을 추가하였다.

최종적인 Parameter 관계는 다음과 같이 정리할 수 있다.

```text
length → X축 Stretch
width  → Y축 Stretch
FINGER → X축 Repeat
width  → Y축 Contact Repeat
```

이를 통해 PCell 설계에서 중요한 것은 단순히 특정 Geometry의 크기를 변경하는 것이 아니라, **Parameter 변화에 따라 서로 연관된 Layout 요소가 함께 이동하고 반복되도록 관계를 정의하는 것**임을 확인하였다.

---

## 13. 최종 정리

전체 PCell 설계 과정은 다음과 같다.

```text
기본 MOS Layout 작성
        ↓
PCell 생성
        ↓
length / width / FINGER 정의
        ↓
X축 Stretch → length
        ↓
Y축 Stretch → width
        ↓
X축 Repeat → FINGER
        ↓
Y축 Repeat → Contact
        ↓
Parameter 변경 확인
        ↓
DRC 검증
        ↓
최종 PCell 완성
```

결과적으로 하나의 고정된 MOS Layout을

```text
Fixed Layout
     ↓
Parameterized Cell
     ↓
length / width / FINGER 변경
     ↓
Layout 자동 변경
```

이 가능한 구조로 확장하였다.

이번 실습을 통해 **Cadence Virtuoso에서 Stretch와 Repeat 기능을 이용한 PCell 생성 방법과 MOSFET의 Length, Width, Finger를 Parameter화하는 기본적인 방법**을 확인할 수 있었다.
