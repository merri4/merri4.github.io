---
layout: post
title: "OpenMP로 몬테 카를로 연산 병렬화하기"
date: 2025-07-08 00:00:00 +0900
categories:
  - play
  - diary
related_posts:
  # - _posts/legasovtape/2024-06-11-legasovtape_2.md
---

OpenMP를 사용해 몬테 카를로 연산을 병렬화해본다. 원을 통해 PI값을 근사해볼 것이다. 


## 수학
---
반지름 $$r=1$$인 원을 커버하는 정사각형을 그린다. 따라서 정사각형의 넓이는 $$2*2=4$$, 원의 넓이는 π$$\cdot r^{2} =$$π가 된다. 실제 π값을 구하려면 이 범위 안에서 랜덤한 점을 무수히 많이 찍었을 때 원 안에서 찍히는 점의 빈도가 정사각형 넓이 대비 원의 넓이가 되어야 한다.

$$
\frac{A_{circle}}{A_{rectangle}} = \frac{π}{4}
$$

하지만 랜덤 좌표 $$(x,y)$$의 범위를 -1에서 1까지로 잡는 것보다 0에서 1로 잡는 것이 점을 더 촘촘하게 찍을 수 있기 때문에 1사분면에 대해서만 계산을 수행할 것이다. 수식은 똑같다.

$$
\frac{A_{target}}{A_{total}} = \frac{\frac{π}{4}}{1}
$$

따라서 π는 아래와 같이 다시 쓸 수 있다.

$$
π = \frac{A_{circle}}{A_{rectangle}} \cdot 4
$$


## 코드
---
serial 버전은 아래와 같다.

```cpp
#include <iostream>
#include <omp.h>
#include <cmath>
#include <random>
#include <vector>
using namespace std;

#define LOWER 0.0   // 0에서 
#define UPPER 1.0   // 1까지 

int main(int argc, char* argv[]) {

    // 오류 출력
    if (argc == 1) {
        printf("Usage : ./<progarm> <n>\n");
        return 0;
    }

    // n 받음
    int n = stoi(argv[1]);

    // 변수 설정
    double in_cnt = 0.0;
    double start_time, end_time;
    double x,y,pi;
    
    // RNG와 dist 생성
    mt19937 rng;
    rng.seed(random_device{}());
    uniform_real_distribution<double> dist(LOWER, UPPER);

    // 시작 시간
    start_time = omp_get_wtime();

    for (int i = 0; i < n; i++) {
        x = dist(rng);
        y = dist(rng);
        if (x*x + y*y < 1.0) in_cnt += 1.0;
    }
    
    // pi 계산
    pi = (in_cnt / n) * 4.0;
    
    // 종료 시간 
    end_time = omp_get_wtime();

    // verbose
    printf("n = %d\tPI : %.6f\tElapsed Time : %f sec\n", n, pi, end_time - start_time);
}
```


OpenMP를 적용한 parallel 버전은 아래와 같다.
```cpp
#include <iostream>
#include <omp.h>
#include <cmath>
#include <random>
#include <vector>
using namespace std;

#define LOWER 0.0   // 0에서 
#define UPPER 1.0   // 1까지 

int main(int argc, char* argv[]) {

    // 오류 출력
    if (argc == 1) {
        printf("Usage : ./<progarm> <n>\n");
        return 0;
    }
    
    // n 받음
    int n = stoi(argv[1]);
    double start_time, end_time;
    double x, y, pi, in_cnt;

    // 스레드 개수에 따라
    vector<int> pl = {2,4,6,8,10};
    
    for (const int& p : pl) {
        
        omp_set_num_threads(p); // 개수 설정

        in_cnt = 0.0;
        start_time = omp_get_wtime();

        #pragma omp parallel private(x,y)
        {
            // 스레드마다 private한 RNG, dist
            mt19937 rng;
            rng.seed(random_device{}() + omp_get_thread_num());
            uniform_real_distribution<double> dist(LOWER, UPPER);

            double local_cnt = 0.0; // 별도의 cnt값 생성

            #pragma omp for
            for (int i = 0; i < n; i++) {
                x = dist(rng);
                y = dist(rng);
                if (x*x + y*y < 1.0) local_cnt += 1.0;
            }

            #pragma omp atomic
            in_cnt += local_cnt; // 외부 in_cnt에 누산
        }

        pi = (in_cnt / n) * 4.0;
        end_time = omp_get_wtime();

        printf("n = %d\t# of threads : %d\tPI : %.6f\tElapsed Time : %f sec\n", n,  p, pi, end_time - start_time);
    }
}
```

결과는 다음과 같다.
```
$ g++ -fopenmp ./monte_carlo_serial.cpp -o monte_carlo_serial
$ ./monte_carlo_serial 100000000
n = 100000000   PI : 3.141604   Elapsed Time : 17.79739 sec

$ g++ -fopenmp ./monte_carlo_parallel.cpp -o monte_carlo_parallel
$ ./monte_carlo_parallel
n = 100000000   # of threads : 2    PI : 3.141615   Elapsed Time : 8.816397 sec
n = 100000000   # of threads : 4    PI : 3.141568   Elapsed Time : 4.541871 sec
n = 100000000   # of threads : 6    PI : 3.141200   Elapsed Time : 2.942815 sec
n = 100000000   # of threads : 8    PI : 3.141732   Elapsed Time : 2.244222 sec
n = 100000000   # of threads : 10   PI : 3.141583   Elapsed Time : 1.793509 sec

```

## Speedup과 Efficiency
---
Speedup은 병렬처리 후 **몇 배 더 좋아졌는지**를 나타낸다. Efficiency는 **사용된 프로세서의 수 대비 효율**를 나타낸다.

프로그램을 serial하게 실행했을 경우의 실행 시간 $$T_{S}$$, $$p$$개의 프로세서로 parallel하게 실행했을 때의 실행 시간 $$T_{P}$$가 있을 때,
Speedup $$S$$와 Efficiency $$E$$는 다음과 같이 쓴다.

$$
S = \frac{T_{S}}{T_{P}}
$$

$$
E = \frac{S}{p} = \frac{T_{S}}{p \cdot T_{P}}
$$

이에 따라 그린 speedup과 efficiency 플롯은 아래와 같다.

![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/openMP/omp_mc_fig1.png)

스레드의 개수가 늘어날수록 speedup 또한 linear하게 좋아진다.

![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/openMP/omp_mc_fig2.png)

Efficiency는 98% ~ 101% 사이를 유지하며 스레드를 늘리더라도 거의 100%의 효율을 보여준다.

plotting 코드는 python으로 작성하였고, 아래와 같다.
```python
import matplotlib.pyplot as plt

threads = [2, 4, 6, 8, 10]
time_parallel = [8.816397, 4.541871, 2.942815, 2.244222, 1.793509]
serial_time = 17.79739

speedup_parallel = [serial_time/t for t in time_parallel]
efficiency_parallel = [s/p for s, p in zip(speedup_parallel, threads)]
ideal_speedup = threads

# Speedup
plt.figure()
plt.plot(threads, speedup_parallel, marker='o', label='atomic')
plt.plot(threads, ideal_speedup, linestyle='--', color='gray', label='ideal')
plt.xlabel('Number of Threads')
plt.ylabel('SpeedUp')
plt.title('SpeedUp by Number of Threads')
plt.legend()
plt.grid(True)
plt.show()

# Efficiency
plt.figure()
plt.plot(threads, efficiency_parallel, marker='o', label='reduction')
plt.xlabel('Number of Threads')
plt.ylabel('WallClock Time (s)')
plt.title('Execution Time by Number of Threads')
plt.legend()
plt.grid(True)
plt.show()
```