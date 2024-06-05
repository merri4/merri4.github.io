---
layout: post
# related_posts:
#   - /legasovtape/legasov_1B/
title: "정규분포, 파이(π), 그리고 몬테 카를로"
date: 2024-05-22 11:00:00 +0900
categories: 
  - play
  - dailyyt
---

* toc
{:toc}

<iframe title="Why π is in the normal distribution (beyond integral tricks)" src="https://www.youtube.com/embed/cy8r7WSuT1I?feature=oembed" height="113" width="200" style="aspect-ratio: 1.76991 / 1; width: 100%; height: 100%;" allowfullscreen="" allow="fullscreen"></iframe>

Grant의 AI-assisted 한국어 더빙도 감상해보자.


## 정규분포에 π가 등장하는 이유는?
Bell curve의 원형인 $$e^{-x^2}$$를 3차원에서 적분하면 곡면 아래의 부피가 π임을 알게 된다.
π가 나오는 것은 곡면 아래의 부피가 곧 두께가 0에 수렴하는 원기둥들의 합이기 때문이다.

이렇게 우회적으로 정규분포 곡선 아래의 2차원 넓이를 구해야 하는 이유는,
사뭇 $$e^x$$ 만큼이나 간단하게 생긴 $$e^{-x^2}$$ 의 부정적분 형태를 **알 수가 없기 때문**이다.


## 그냥 적분하면 되는 거 아님?
이런 경우에 유용하게 쓸 수 있는 트릭 중에 몬테 카를로 방법이 있다.

보통 원의 넓이를 통해 π를 근사하는 문제를 [예시](https://en.wikipedia.org/wiki/Monte_Carlo_method)로 사용하는데, 처음 몬테 카를로를 접할 때 나는 이런 생각을 했다. 

점을 찍어서 샘플링된 비율로 원하는 값을 근사한다는 건 이해했음. 근데 점의 위치를 구분하려면 애초에 함수를 알아야 하고, 함수를 아는거면 면적이든 부피든 적분해서 구할 수 있는데 왜 굳이 이런 방법을 쓰는거임?
{:.message}

쉽게 말해 *함수를 알면 그냥 적분하면 되는거 아닌가?* 싶었던 것.
적분도 될 뿐더러 기하학적 공식도 이미 알고 있는 원이다보니 더 그런 생각이 들었던 듯 싶다.

그땐 몰랐다. $$e^{-x^2}$$ 처럼, 단순한 모양임에도 적분할 수 없는 함수들이 있다는 걸.


## 몬테 카를로 다시 설명하기
몬테 카를로의 핵심은 **적분할 수 없는 함수여도 된다**는 것이다.
그래서 부정적분 형태를 구할 수 없는 함수들에 매우 유용하다.
컴퓨터 계산에서 해석학적 오차나 연산 효율 등의 장점도 덤.

그런 의미에서 적분도 되고 공식도 밝혀져 있는 원으로 예시를 드는 건 몬테 카를로의 취지와 그 파워를 설명하기에 다소 부적절하다고 본다.
단순 bell curve 형태지만 적분할 수 없는 $$e^{-x^2}$$ 의 면적을 구하는 문제로 다시 풀어보자.

몬테 카를로의 간략한 흐름은 아래와 같다.
1. 구하려는 면적 $$A$$ 설정
> $$e^{-x^2}$$ 밑의 면적을 $$A$$라고 하자.

2. $$A$$를 포함하는 면적을 설정
> 좌우로 $$l$$의 길이로 뻗어나가고, $$e^{-x^2}$$의 최대 높이가 1므로 높이는 1인 직사각형을 설정한다(더 높게 설정해도 무방). 따라서 직사각형의 넓이는 $$2l*1=2l$$이 된다. 물론 $$e^{-x^2}$$는 좌우로 무한히 뻗어나가는 모양이므로 면적 $$A$$를 완벽하게 덮을 순 없으나, $$l$$을 늘릴수록 해석적 근사값에 가까워질 수 있다.

3. 전체 면적 내에서 랜덤한 점을 찍었을 때 그 점이 함수 아래 ($$A$$ 안에) 있을 확률 = $$A$$ / 전체 면적
> 이항하면 `좌항에서 점을 찍어 구한 확률 * 전체 면적 = A` 가 된다. 따라서 확률에 $$2l$$을 곱해 $$A$$를 구한다.


```python
import math
import random as rd
from tqdm import tqdm

# e^-x^
def func(x) :
    return math.exp(-x**2)

def monte_carlo(l : float, num_dot : int) -> float :

    in_dot = 0

    for i in tqdm(range(num_dot)) :
        x, y = rd.uniform(-l, l), rd.uniform(0, 1)
        if y < func(x) :
            in_dot += 1

    return in_dot / num_dot * 2 * l


if __name__ == "__main__" :

  # root pi = 1.77245에 가까워져야 함.
  print("root pi : {}".format(monte_carlo(10, 1000000)))  # 1.77796

```

설정한 면적 안에 **충분히 많은 점**이 **uniform**하게 샘플링되어야 한다는 점도 중요하다.

Monte Carlo method의 자세하고 다양한 설명은 [여기](https://studyingrabbit.tistory.com/33)를 추천.