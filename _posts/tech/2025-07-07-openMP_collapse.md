---
layout: post
title: "OpenMP - collapse 지시문"
date: 2025-07-07 00:00:00 +0900
categories:
  - tech
---

`#pragma omp for` 을 사용 시, nested loop일 경우 가장 바깥의 루프부터 병렬화를 한다.
추가적으로 `collapse(n)` 를 사용하면 가장 바깥 루프부터 loop가 병렬화된다.  
단, 중간에 break가 있는 등 loop의 전체 작업 길이를 예측할 수 없으면 사용할 수 없다.

## 예시
---
collapse를 사용하지 않은 상태의 반복 코드를 보자.

```cpp
#include <iostream>
#include <omp.h>
#include <unistd.h>
int main() {

    omp_set_num_threads(6);

    #pragma omp parallel for
    for (int i=0; i<3; i++){
        for (int j=0; j<4; j++) {
            sleep(1);
            printf("Thread : %d\ti : %d\tj : %d\n", omp_get_thread_num(), i, j);
        }
    }
}
```
i에서 3개, j에서 4개, 총 12회의 반복이 수행되는데, 이를 `#pragma omp parallel for`로 병렬화하였다.
병렬화가 가장 바깥 loop에만 적용되므로, 3개의 작업이 multithreading된다.
(아무것도 쓰지 않은 것은 `collapse(1)`과 동작이 같다.)

```
$ time ./collapse

Thread : 0    i : 0   j : 0
Thread : 0    i : 0   j : 1
Thread : 0    i : 0   j : 2
Thread : 0    i : 0   j : 3

Thread : 1    i : 1   j : 0
Thread : 1    i : 1   j : 1
Thread : 1    i : 1   j : 2
Thread : 1    i : 1   j : 3

Thread : 2    i : 2   j : 0
Thread : 2    i : 2   j : 1
Thread : 2    i : 2   j : 2
Thread : 2    i : 2   j : 3

real    0m 4.014s
```

![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/openMP/diagram1.png)

병렬화 수준이 가장 바깥 루프의 i 인덱스까지이므로, 3개의 스레드가 i 인덱스를 따라 j 반복을 수행하고 있다.
총 가용 가능한 스레드가 6개임에도, 3개의 스레드만이 작업을 할당받아 각각 4개의 작업을 수행한다.




스레드의 utilization을 최대로 하기 위해서, `collapse(2)`를 사용해보자.

```cpp
#include <iostream>
#include <omp.h>
#include <unistd.h>
int main() {

    omp_set_num_threads(6);

    #pragma omp parallel for collapse(2)
    for (int i=0; i<3; i++){
        for (int j=0; j<4; j++) {
            sleep(1);
            printf("Thread : %d\ti : %d\tj : %d\n", omp_get_thread_num(), i, j);
        }
    }
}
```
`collapse(2)`에 의해 j까지 병렬화되어 총 작업의 개수가 12개가 된다.
결과는 아래와 같다.
```
$ time ./collapse

Thread : 5    i : 2   j : 2
Thread : 5    i : 2   j : 3

Thread : 3    i : 1   j : 2
Thread : 3    i : 1   j : 3

Thread : 2    i : 1   j : 0
Thread : 2    i : 1   j : 1

Thread : 1    i : 0   j : 2
Thread : 1    i : 0   j : 3

Thread : 4    i : 2   j : 0
Thread : 4    i : 2   j : 1

Thread : 0    i : 0   j : 0
Thread : 0    i : 0   j : 1

real    0m 2.019s
```

![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/openMP/diagram2.png)

이번에는 0부터 5번까지의 6개의 스레드가 각각 2개씩 task를 수행한다.


## 3차원에서 collapse
---
3차원으로 늘려서 테스트해볼 수도 있다. 아래와 같이 i 반복을 3번, j 반복을 4번, k 반복을 2번 총 24개의 반복을 만든다.
```cpp
#include <iostream>
#include <omp.h>
#include <unistd.h>
int main() {

    omp_set_num_threads(6);

    #pragma omp parallel for
    for (int i=0; i<3; i++){
        for (int j=0; j<4; j++) {
            for (int k=0; k<2; k++) {
                sleep(1);
                printf("Thread : %d\ti : %d\tj : %d\tk : %d\n", omp_get_thread_num(), i, j, k);
            }
        }
    }
}
```
collapse에 대한 표기가 없을 경우 가장 바깥 차원인 i에 대해서만 병렬화를 수행한다.
그림과 같이 i에 대해서만 병렬화가 수행되어, 실행에 총 8초가 소요된다.

![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/openMP/diagram3.png)


이제 `collapse(2)`로 바깥 2개 반복 레벨에 대한 병렬화를 수행한다. 24개의 작업이 각각 6개의 스레드에 4개씩 나눠져 수행된다. 

```
$ time ./collapse

Thread : 0    i : 0   j : 0   k : 0
Thread : 0    i : 0   j : 0   k : 1
Thread : 0    i : 0   j : 1   k : 0
Thread : 0    i : 0   j : 1   k : 1

Thread : 5    i : 2   j : 2   k : 0
Thread : 5    i : 2   j : 2   k : 1
Thread : 5    i : 2   j : 3   k : 0
Thread : 5    i : 2   j : 3   k : 1

Thread : 2    i : 1   j : 0   k : 0
Thread : 2    i : 1   j : 0   k : 1
Thread : 2    i : 1   j : 1   k : 0
Thread : 2    i : 1   j : 1   k : 1

Thread : 1    i : 0   j : 2   k : 0
Thread : 1    i : 0   j : 2   k : 1
Thread : 1    i : 0   j : 3   k : 0
Thread : 1    i : 0   j : 3   k : 1

Thread : 3    i : 1   j : 2   k : 0
Thread : 3    i : 1   j : 2   k : 1
Thread : 3    i : 1   j : 3   k : 0
Thread : 3    i : 1   j : 3   k : 1

Thread : 4    i : 2   j : 0   k : 0
Thread : 4    i : 2   j : 0   k : 1
Thread : 4    i : 2   j : 1   k : 0
Thread : 4    i : 2   j : 1   k : 1

real    0m 4.015s
```

![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/openMP/diagram4.png)