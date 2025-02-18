---
layout: post
title: "one of the variables needed for gradient computation has been modified by an inplace operation 오류"
date: 2025-02-18 19:00:00 +0900
categories:
  - tech
  - pytorch
---

> one of the variables needed for gradient computation has been modified by an inplace operation : [torch.cuda.FloatTensor [8192, 128]], which is output 0 of AsStridedBackward0, is at version 2; expected version 1 instead. Hint: enable anomaly detection to find the operation that failed to compute its gradient, with torch.autograd.set_detect_anomaly(True).

+=나 -=처럼 줄여서 쓴 할당 연산자를 없애고 풀어서 쓰면 된다. 
병렬처리되는 텐서를 inplace로 연산하면 멀티스레딩 하다가 숫자가 안 맞아서 오류가 생긴다.
