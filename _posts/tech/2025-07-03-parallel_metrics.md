---
layout: post
title: "병렬처리 성능 분석 - Amdahl's Law, Gustafson's Law"
date: 2025-07-03 00:00:00 +0900
categories:
  - tech
---

병렬처리 시 성능을 분석하는 여러 metric이 있다.

## Speedup과 Efficiency
---
Speedup은 병렬처리 후 **몇 배 더 좋아졌는지**를 나타낸다. Efficiency는 **사용된 프로세서의 수 대비 효율**를 나타낸다.

프로그램을 serial하게 실행했을 경우의 실행 시간 $$T_{S}$$, $$p$$개의 프로세서로 parallel하게 실행했을 때의 실행 시간 $$T_{P}$$가 있을 때,
Speedup $$S$$와 Efficiency $$E$$는 다음과 같이 쓴다.

$$
S = \frac{T_{S}}{T_{P}}
$$

serial time이 100이고 parallel time이 20이면, 5배의 speedup이 된다.

$$
E = \frac{S}{p} = \frac{T_{S}}{p \cdot T_{P}}
$$

이 때, 5개의 processor를 사용했다면 효율은 1.0, 즉 100%가 된다.

다만 이건 이상적인 경우로, 보통은 프로세서가 늘어나더라도 다음의 요인들에 speedup은 둔화되고 효율은 점점 감소하게 된다.
- 아무리 병렬화를 하더라도 병렬적으로 처리할 수 없는 serial part
- multithreading 시 발생하는 overhead
- synchronization 문제


## Amdahl's Law
---
병렬화를 할 수 없는 자투리 serial part 때문에 발생하는 포화를 반영한 계산 방식이 Amdahl's Law다. 아래 사진과 같이 병렬처리 가능한 부분을 아무리 많은 processor로 쪼개더라도 serial work 시간은 줄일 수 없게 된다.

![](https://codingforspeed.com/wp-content/uploads/2015/07/amdahl.jpg)

이에 따라 parallel execution time $$T_{P}$$를 다음과 같이 다시 정의한다.

$$
T_{P} = t_{ser} + \frac{T_{S} - t_{ser}}{p}
$$

다시 말해 병렬처리 불가 시간(serial part) + 병렬처리 가능한 시간(parallel part)와 같다. 이를 Speedup에 대입하면,

$$
S = \frac{T_{S}}{t_{ser} + \frac{T_{S} - t_{ser}}{p}}
$$

p가 커질수록 병렬처리 가능한 시간은 작아지지만, serial part $$t_{ser}$$는 분모에 남는다.

$$
\lim_{p \to \infty}S = \frac{T_{S}}{t_{ser}}
$$

따라서 아무리 프로세서를 많이 증가시키더라도 S는 이 이상으로 커질 수 없다.


## Gustafson's Law
---
프로세서를 늘렸을 때의 장점을 Amdahl보다 더 긍정적으로 바라본 경우의 계산법이다. $$a$$는 serial part가 전체에서 차지하는 비중, $$p$$는 프로세서의 개수이다. 

$$
S = p - a \cdot (p-1) = p \cdot (1-a) + a
$$

Amdahl과의 차이점은 serial part를 절대적인 숫자가 아닌 상대적인 비율로 바라보아, processor의 개수가 늘어날 수록 그 영향이 미미해질 수 있는 요인으로 취급한다는 점이다. 

동일한 조건에서 계산해보았을 때 Amdahl's Law에 따른 speedup은 일정 숫자에서 포화되나, Gustafson's Law에 따른 speedup은 linear하게 늘어난다.
