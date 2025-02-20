---
layout: post
title: "stream did not contain valid UTF-8 오류"
date: 2025-02-20 00:00:00 +0900
categories:
  - tech
  - pytorch
---


```
from tokenizers import BertWordPieceTokenizer

data_path = "./input.txt"

# 토크나이저 세팅
tokenizer = BertWordPieceTokenizer()

# 토크나이저 학습
tokenizer.train(
    files=data_path,
    vocab_size=3000,
    min_frequency=2,
    )

# vocab 확인
vocab = tokenizer.get_vocab()
print(sorted(vocab, key=lambda x: vocab[x]))

```

BertWordPieceTokenizer를 train할 때, UTF-8 형식이 아닌 텍스트 데이터는 토크나이저 학습이 불가하다. 따라서 메모장 등의 편집기로 txt 파일을 열어 Save as (다른 이름으로 저장) 시에 포맷을 UTF-8으로 변경해주면 해결된다.

[참고](https://github.com/huggingface/tokenizers/issues/282#issuecomment-1513511288)