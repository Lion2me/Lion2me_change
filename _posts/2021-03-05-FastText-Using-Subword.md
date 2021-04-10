---
layout: post
title: FastText Using SubWord
date: 2021-03-05 19:20:23 +0900
categories: FastText Embedding
---

SubWord를 이용한 FastText 학습
---

#### 1. FastText 란?

FastText는 FaceBook의 AI개발팀에서 만든 Word Embedding 방법입니다. 이 방식은 이전에 다루었던 Word2Vec과 유사하므로 한번 참고하고 오면 이해하기 쉬울 수 있습니다.

##### 1 - 1 ) 왜 FastText가 만들어졌나

FaceBook과 같은 SNS에서는 완전한 단어로 만들어진 문장이 아닌 자연어로 이루어진 경우가 많습니다. 이러한 문장에서는 단어를 Token화 시킬 경우 동일한 의미를 갖는 단어임에도 다르게 분류되기 마련입니다.

예를 들어 **['나이값','나잇값','나잇갑','나이갑']** 이라는 단어들을 사용한 문장이 있다고 가정합니다. 모두 나잇값의 의미를 갖는 단어들이지만 모두 스펠이 다르다는 이유로 다른 단어로 생각하게 됩니다.

BagOfWords의 경우에는 [1,0,0,0] [0,1,0,0] [0,0,1,0] [0,0,0,1] 처럼 모든 단어를 독립성을 가진 채로 표현합니다. 이 경우 예시의 모든 단어는 각자 아예 다른 단어로 인식하게 되겠네요.

Word2Vec을 기준으로 살펴봅시다. 네 단어 모두 같은 의미를 가지고 있으며 사용하는 상황이 일치합니다. 오히려 오타에 가까운 수준이죠. 그렇가면 네 단어는 모두 비슷한 벡터를 갖게 되는 것을 쉽게 생각할 수 있습니다만 위 예시에서 정답에 가까운 단어는 나잇값입니다. 그러면 나머지 세 단어는 낮은 빈도로 등장하게 되죠.

이 경우 Word2Vec의 단점이 나타납니다. 너무 낮은 빈도로 등장한 단어들에 대해 제대로 된 학습이 진행되지 않는 단점입니다.

학습을 진행하는 과정에서 낮은 빈도로 등장하는 단어도 다른 단어와 마찬가지로 학습이 진행됩니다. CBOW 든 Skip-Gram이든 해당 단어는 유사한 단어를 향해 나아가는 것은 맞습니다만 문제는 그 학습의 빈도로 극도로 낮다는 점입니다.

학습의 타겟이 되는 빈도는 극도로 낮지만 Word2Vec에서 정답이 아닌 단어에 대해 거리를 넓히는 과정에서 negative Sampling에 걸릴 확률은 언제나 있습니다. 예를 들어 1M개의 문장에서 "나이갑"이라는 단어가 단 2번 등장했다면 1M 번 학습 중 단 2문장에서 학습이 진행되며 나머지 (1M - 2) 문장에서는 나이값이라는 단어를 먼 벡터로 떨어뜨리기 위한 negative Sampling에 포함 될 확률이 있습니다.

이 경우 "나이갑"이라는 단어의 벡터가 올바르지 않는 자리에 있을 확률이 꽤나 높습니다. 그러면 만약 나이갑이라는 단어가 분류하고자 하는 input값으로 들어올 경우 잘못된 답을 내놓을 수 있고, 군집화도 제대로 되지 않을 수 있습니다.

또한 **['나이값','나잇값','나잇갑','나이갑']** 이라는 단어들은 모두 공통된 의미이며 누군가는 이런 값들의 입력으로 **"나잋값"** 이라고 입력 할 수 있습니다. 물론 이 경우 국어를 먼저 배우는 과정이 필요하겠지만, 자연어는 모든 일이 일어날 수 있는 세상이므로 가정하겠습니다. 이 경우 기존의 Embedding을 통해 나온 모든 모델은 OOV에러를 발생시킵니다.

OOV에러는 Out Of Vocabulary 에러를 나타내며 즉 학습한 모델에는 해당 단어가 벡터화 되지 않았다는 것을 나타냅니다. "나잋값"이라는 단어를 학습 시 사용되지 않았으니 값을 찾지 못한다는 말입니다.

이러한 두 문제점을 해결하기 위해 FastText는 등장했습니다.

##### 1 - 2 ) FastText 방식

FastText는 유사한 형태의 단어들을 유사한 벡터로 표현하기 위해서 또한 OOV 문제를 해결하기 위해 단어를 더욱 작은 단위(글자 혹은 자모)로 나누어서 학습하는 방식을 제안합니다.

예를 들면 **"아이스크림"** 이라는 단어를 Embedding 할 경우 기존에는 통째로 학습했었다면 FastText는 **["<아이","아이스","이스크","스크림","크림>"]** 과 같은 window 크기를 기준으로 잘라서 학습시킵니다. 문장의 처음과 마지막에 있는 **["<",">"]** 을 사용하는 이유는 단어의 시작과 끝을 나타내주기 위해서 입니다.

그리고 모든 단어들의 벡터의 평균을 내주면 아이스크림의 벡터값이 나오는 방식으로 진행됩니다. **단어의 부분집합을 이용하여 해당 단어의 벡터값** 을 구하는 방식이라고 생각할 수 있습니다.

이러한 방식을 이용하면 다음과 같은 예시에서 효과적인 벡터를 얻을 수 있습니다.

**아이스크림 아이스커피 아이스크림빵**

**["<아이","아이스","이스크","스크림","크림>"]**
**["<아이","아이스","이스커","스커피","커피>"]**
**["<아이","아이스","이스크","스크림","크림빵","림빵>"]**

아이스크림과 아이스커피는 2개의 단어가 겹치는 부분이 있으므로 어느정도 연관성이 있다는 것을 알 수 있고, 마찬가지로 벡터로 표현이 될 것입니다. 그리고 아이스크림과 아이스크림빵은 4개의 단어가 겹치므로 더 큰 연관성이 있을 것임을 알 수 있습니다.

하지만 더 작은 부분조합으로 나타낼 수 있습니다. 예를 들면 자음과 모음을 각자 분리시키면 어떨까요? 자음과 모음으로 나누면 다음과 같은 형태가 됩니다. 받침이 없는 경우 - 로 표현합니다.

**['ㅇ','ㅏ','-','ㅇ','ㅣ','-','ㅅ','ㅡ','-','ㅋ','ㅡ','-','ㄹ','ㅣ','ㅁ']**
**['ㅇ','ㅏ','-','ㅇ','ㅣ','-','ㅅ','ㅡ','-','ㅋ','ㅓ','-','ㅍ','ㅣ','-']**
**['ㅇ','ㅏ','-','ㅇ','ㅣ','-','ㅅ','ㅡ','-','ㅋ','ㅡ','-','ㄹ','ㅣ','ㅁ','ㅃ','ㅏ','ㅇ']**

이렇게 구분했을 경우에는 **['ㅇ','ㅏ','-','ㅇ','ㅣ','-','ㅅ','ㅡ','-','ㅋ']** 까지는 모두 같은 벡터를 갖게 됩니다. 마찬가지로 학습을 시킬 경우에는 양 옆에 '<' , '>' 로 구분하여 부분집합 형태로 잘려져서 학습을 거치게 됩니다.

유사한 단어가 더 세밀하게 벡터 연산을 거치게 되어 유사한 정도를 더욱 잘 표현 할 수 있습니다.

나잇값과 유사한 단어 셋도 공통적으로 나이라는 단어가 들어갑니다. "잇"이나 "잋"도 자음 모음으로 나누어보면 ['ㅇ','ㅣ','ㅅ'] 과 ['ㅇ','ㅣ','ㅊ'] 으로 나눌 수 있으니 결국 모두 나이라는 단어가 포함되어 있다고 볼 수 있습니다.

만약 "나이"라는 단어 자체에 대해 점수를 줄 수 있다면 "나잇"도 유사한 점수를 줄 수 있고 "나잋"이라는 단어도 점수를 줄 수 있습니다. 어찌됬건 유사한 것은 마찬가지니까요.

#### 2. FastText 실행

이 단계에서는 실제로 FastText를 실행해보록 하겠습니다.

##### 2 - 1 ) FastText 설치

가장 먼저 해야 할 일은 FastText를 설치하는 작업입니다. 파이썬을 이용하여 작업 할 예정이므로 파이썬 기준으로 말씀드리면

```
pip3 install fasttext
```

명령어를 통해 다운받아 주시면 됩니다.

##### 2 - 2 ) FastText 학습 데이터

그리고 사용할 데이터 셋은 국민청원의 데이터로 lovit님의 국민청원 데이터 셋[링크]을 사용하여 구현해보겠습니다.

https://github.com/lovit/petitions_dataset

```python
# 국민청원 데이터를 다운 받을 수 있습니다.
from petitions_dataset import fetch
fetch()

#빠른 학습을 위해 2017년 8월 부터 2018년 3월까지의 데이터를 불러옵니다.
from petitions_dataset import Petitions
petitions = Petitions(begin_date='2017-08-01', end_date='2018-03-30')

#원하는 컬럼의 값을 키로 설정합니다.
petitions.set_keys('content')

len(petitions)
#149450
```

##### 2 - 3 ) 데이터 전처리

아까전에 설명했던 자음과 모음으로 텍스트를 분해하는 함수는 lovit님이 사용하신 자모분리 함수에 제가 겪은 문제를 해결하기 위한 약간의 소스를 추가하여 사용했습니다. FastText를 위한 자모분리 함수는 많은 사람들이 만든 소스가 있으니 쉽게 구하고, 또 이해할 수 있습니다.

```python
# 이 소스는 lovit님의 블로그에서 가져온 소스입니다.

# soynlp 는 다운로드 받으셔야 합니다.

from soynlp.hangle import decompose

doublespace_pattern = re.compile('\s+')

korean_pattern = re.compile('[가-힣ㄱ-ㅎㅏ-ㅠ]')

english_pattern = re.compile('[a-zA-Z]')

def is_korean(char):
    if korean_pattern.match(char):
        return True
    else:
        return False

def is_english(char):
    if english_pattern.match(char):
        return True
    else:
        return False

def jamo_sentence(sent):

    def transform(char):
        if char == ' ':
            return char
        # 자모로 자르는 데 문제는 영어일 경우
        if(is_korean(char)):
            cjj = decompose(char)
        # 영어 함수의 경우에는 변경해주어야 할 것 같다.
        elif(is_english(char)):
            cjj = (char)
        # 저는 사용하는 데이터 중 한자가 사용되는 문제가 생겨서 다음의 방식으로 공백처리를 했습니다.
        else:
            return (' ')
        if len(cjj) == 1:
            return cjj
        cjj_ = ''.join(c if c != ' ' else '-' for c in cjj)
        return cjj_
    sent_ = ''.join(transform(char) for char in sent)
    sent_ = doublespace_pattern.sub(' ', sent_)
    return sent_

jamo_sentence('어쩌다 마주친 그대 모습에 내 마음은')
# 'ㅇㅓ-ㅉㅓ-ㄷㅏ- ㅁㅏ-ㅈㅜ-ㅊㅣㄴ ㄱㅡ-ㄷㅐ- ㅁㅗ-ㅅㅡㅂㅇㅔ- ㄴㅐ- ㅁㅏ-ㅇㅡㅁㅇㅡㄴ'
```

해당 함수에 문장을 파라미터로 넘겨 줄 경우에 해당 문장은 자음 모음으로 구분되어 나타나게 됩니다.

##### 2 - 4 ) FastText 모델 학습

```python
import fasttext

# 데이터를 text 형태로 변환합니다. 원하는 컬럼은 데이터 셋을 불러오는 과정에서 추가할 수 있습니다.
petitions_text = []
for content in petitions:
    petitions_text.append(content)

# 모든 문장을 자모로 분할시킵니다.
for idx,text in enumerate(petitions_text):
    petitions_text[idx] = jamo_sentence(text)

# MacOS에서 file open을 통해 텍스트를 저장 할 시 유니코드가 읽어지지 않는 에러가 발생했습니다.
# pandas를 통한 csv 저장 시 에러가 발생하지 않아서 다음과 같은 방법을 사용했습니다.
# 저와 같은 에러가 없는 경우 txt 형식으로 저장하는 것이 가장 옳습니다.
data = pd.DataFrame()
data['text'] = petitions_text
data.to_csv('./petition_text.txt')

# 모델을 학습 시킵니다.
# fasttext 라이브러리의 예제가 대부분 기존 데이터를 불러오는 방식이라 어떤 데이터를 사용하는지 잘 나타나있지 않습니다.
# fasttext의 train_unsupervised 함수는 공백과 \n(줄 바꿈) 으로 구분 되어 있는 텍스트를 기반으로 분석합니다.
model = fasttext.train_unsupervised('./petition_text.txt')
```

#### 3. 결과 확인

모델을 학습하는 과정에서 사용 된 함수 명(train_unsupervised)을 보면 알 수 있듯 이 방식은 단어의 대표 벡터(representation Vector)를 찾는 과정일 뿐 명확한 결과를 수치화 할 수는 없습니다.

그래서 단어를 입력하고 어떤 결과가 나왔는지를 주로 살펴보도록 하겠습니다.

```python
input_ = "청년들이 취업 걱정없는 세상을 만들어주세요."
input_ = jamo_sentence(input_).split(' ')

```

자음 모음으로 분리된 값들을 다시 단어로 재조립하는 방법은 다음의 함수를 사용했습니다. 스스로 코드를 작성해봤지만 코드가 조금 길어져서 이해도가 조금 떨어지는 관계로 좋은 소스를 작성해준 분의 소스를 사용했습니다.

``` python
def jamo_to_word(jamo):
  # idx를 기반으로 순차접근 방식으로 접근하면서 저는 소스가 흐트러졌는데 깔끔합니다.
  jamo_list, idx = [], 0
  while idx < len(jamo):
    if not character_is_korean(jamo[idx]):
      jamo_list.append(jamo[idx])
      idx += 1
    else:
      jamo_list.append(jamo[idx:idx + 3])
      idx += 3
      word = ""
  for jamo_char in jamo_list:
    if len(jamo_char) == 1:
      word += jamo_char
    elif jamo_char[2] == "-":
      word += compose(jamo_char[0], jamo_char[1], " ")
    else: word += compose(jamo_char[0], jamo_char[1], jamo_char[2])
    return word

출처: https://joyhong.tistory.com/137
```

```python
print(input_[0])
print(jamo_to_word(input_[0]))
print([jamo_to_word(word) for score , word in model.get_nearest_neighbors(input_[0])])

# ㅊㅓㅇㄴㅕㄴㄷㅡㄹㅇㅣ-
# 청년들이
# ['청년들은', '젊은청년들이', '청년들이나', '젊은이들이', '청년들도', '청년들을', '청년들만', '청년들', '청년들로', '젊은이들도']

```

결과를 보면 청년과 유사한 의미를 가진 **젊은** 과 같은 단어를 성공적으로 얻을 수 있음을 알 수 있습니다. 예시를 조금 꼬아서 생각해보겠습니다.

"젊은청년들이" 단어를 "젊은청연들이"로 바꾸어보겠습니다. 실제로 이 단어는 학습 데이터에 없는 단어이므로 일반적인 Word2Vec과 BagOfWords에서는 OOV문제를 일으킬 수 있는 단어입니다.

그리고 결과는 다음과 같이 나왔습니다.

```
ㅈㅓㄻㅇㅡㄴㅊㅓㅇㅇㅕㄴㄷㅡㄹㅇㅣ-
젊은청연들이
['사연들이', '젊은청년들이', '남편들이', '사연들을', '젊은청년', '누나들이', '쭉빵까페에서', '트윗터등으로', '많은일들이', '보신분들이']
```

오타로 인해 실제 데이터에 없는 단어임에도 정확하지 않지만 젊은청년 이라는 단어가 어느정도 유사하다는 결과를 성공적으로 도출해내고 있습니다.

다음에는 이 FastText를 이용하여 형태소 분석을 포함한 효과적인 방식을 찾아보는 것에 초점을 맞추어 포스팅을 진행하겠습니다.
