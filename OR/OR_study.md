# CMOS OR Gate Layout 설계 및 고찰

## 1. 설계 개요

CMOS OR Gate는 **NOR Gate의 출력에 Inverter를 연결하여 구현**하였다.

OR Gate의 논리식은

\[
Y=A+B
\]

이며, NOR Gate의 출력은

\[
X=\overline{A+B}
\]

이므로 여기에 Inverter를 연결하면

\[
Y=\overline{X}
=\overline{\overline{A+B}}
=A+B
\]

가 된다.

따라서 전체 구조는 다음과 같다.

```text
A ──┐
    ├── NOR ── X ── Inverter ── Y
B ──┘
