---
layout: post
title: MacOS 에서 Spark 설치하기
date: 2021-03-02 19:20:23 +0900
category: Spark
---

MacOS 기반 Spark 설치
---

###1. Spark란

Spark는 데이터를 병렬로 처리하여 여러 컴퓨터 혹은 단일 컴퓨터에서 분산하여 처리하게 설계된 플랫폼 입니다.

병렬 분산 처리 플랫폼을 생각하면 데이터 엔지니어링에 대해 공부한 사람들은 가장 먼저 Hadoop의 MapReduce를 생각합니다. Spark보다 Hadoop이 먼저 개발 되었고 또한 HDFS라는 데이터 분산 저장 방식을 사용하고 있기에 더욱 유명하기 때문입니다.

같은 Apache에서 만들어진 두 시스템은 기본적으로 같은 느낌의 병렬 분산 처리를 제공합니다. 하지만 Spark는 Hadoop의 MapReduce의 상위 플랫폼임은 이미 많이 알려진 사실입니다. 그 차이점에 대해 알아보겠습니다.

####1 - 1) HDFS

~~Hadoop은 제작자의 아이의 코끼리 인형 이름입니다.~~

Hadoop 클러스터의 모든 구조에 대해 공부하면 좋겠지만 Hadoop의 가장 기본적이고 큰 개념만을 다뤄보도록 하겠습니다.

Hadoop은 HDFS라는 데이터 분산 저장 플랫폼과 MapReduce라는 데이터 병렬 분산 처리 플랫폼으로 나누어집니다.

HDFS는 데이터를 분산하여 저장하는 플랫폼으로써 기본적으로 다중 서버를 가정하고 만들어진 방식이지만 단일 서버에서도 사용은 할 수 있습니다. ( 물론 효율적이지 않습니다. )

Name Node에서 Data Node로 데이터를 블럭의 수와 용량을 기준으로 분할하여 저장하며 그러한 저장 상태를 메타데이터에 저장해놓고 불러 올 경우 메타데이터의 정보를 기반으로 Data Node를 탐색하는 꽤나 훌륭한 저장하는 방식입니다.

이런 방식을 사용하면 DataNode중 하나가 끊기는 일이 생기더라도 다른 DataNode에서 원하는 정보를 불러 올 수 있습니다. 서버가 충분히 많고 용량과 블럭이 골고루 분산되어있다면 말이죠.

심지어 NameNode가 고장날 수 있다는 가정으로 Secondary NameNode라는 여분의 NameNode마저도 만들어서 안정성을 확보합니다.

Hadoop은 100테라의 저장공간을 갖는 서버 1개보다 10테라 저장공간을 가진 10개 서버를 구축하는 것이 저렴하기에 발전되었습니다. 즉 비용적인 측면과 안정성때문에 만들어진 것이죠. 일반적인 DB에 파일 데이터를 저장하면 I/O가 무거워지는건 너무 당연합니다. NoSQL이 등장하여 파일 데이터의 저장 측면이 조금은 나아졌지만 안정성이 낮아 NoSQL에서도 요즘 분산 저장을 권장하고 있죠.

#### 1 - 2) MapReduce?

저장을 했다면 데이터를 분석해야 할 차례가 왔습니다. 분산하여 저장 할 정도의 데이터 량이면 상당히 큰 데이터일 확률이 높습니다.

그런 데이터를 통째로 Memory에 불러오는게 가능할까요? RAM은 최근 발달되어 64GB까지도 업그레이드가 가능하지만 다루려는 데이터가 64GB를 넘지 않는다고 장담할 수 없습니다. 심지어 연산을 위한 공간이 필요하니 모든 공간을 할당받을 수도 없죠.

그래서 등장한 방식이 MapReduce 방식입니다. 정확히는 Map과 Reduce입니다. Map은 단순한 Filtering정도라고 생각하시면 됩니다. Reduce 작업 시 사용할 데이터만을 뽑아오는 작업으로 원래의 경우 모든 데이터를 메모리에 올린 뒤 전체 데이터에 대해 Mapping을 시도 합니다. 이러한 작업을 각각의 Task로 나누어 분산 된 서버에 할당한다면 어떻게 될까요?

100GB의 데이터를 10개의 작업으로 나누면 각 작업의 크기는 10GB정도가 될 것입니다. 심지어 연산 속도도 빨라지겠죠. 마지막에 그 값을 하나로 모아주면 전체 데이터에 대한 Mapping이 끝났을 것 입니다.

일반적으로 Map 과정의 리턴값은 {key:value}의 형태를 갖습니다.

Map 과정이 끝난 데이터는 각각 Combiner를 거쳐가며 키를 기준으로 value를 배열처럼 묶어놓고 다음 과정으로 넘겨줍니다. 이런 과정을 거치는 이유는 모든 데이터를 각각의 리턴값을 모두 넘겨버리면 너무 많은 데이터( 거의 모든 데이터 )를 넘기면서 Mapping의 효율이 좋지 않기 때문입니다.

그 후로 Shuffle 단계로 Combiner의 결과로 나온 데이터들을 한꺼번에 모아서 각 키의 hash값을 기준으로 각각을 Reducer로 분배합니다. 여기서 hash값은 key값의 해쉬값에 리듀서의 개수를 기준으로 나누고 남은 값입니다.

그러면 각 리듀스에는 해당하는 키 값들이 전부 모이게 됩니다. 예를 들면 "장난감"이라는 단어의 hash값도 2이고 "아기"의 hash값이 2라면 키 값이 장난감과 아기인 값은 모두 2번 리듀서에 들어오게 됩니다. 리듀서에서 그 키 값을 기준으로 정렬하면 각각의 키 값에 대한 모든 요소를 순차적으로 접근 할 수 있게 되었죠.

리듀스는 일반적으로 return 이라고 생각하면 됩니다. output이 일어난 것이죠.

#### 1 - 3) Spark

드디어 Spark입니다. 이전에 MapReduce를 설명 한 이유는 바로 Spark를 설명하기 위해서 입니다. 둘의 차이점을 알면 이해하기 쉽기 때문에 미리 설명을 드렸습니다.

기본적으로 병렬 분산 처리라는 점은 동일합니다. 하지만 Spark는 Map / Reduce가 아닌 Transformation / Action으로 나누어 집니다.

MapReduce의 과정에서 Map과정의 결과값이 {key:value}으로 넘기며 동시에 Filtering 과정을 거치는 것처럼 Transformation 또한 조건에 맞는 Filtering 과정을 거치며 기본적으로는 {key:value}로 만드는게 일반적입니다. 하지만 강제는 아닙니다.

여기서 차이점은 중간 과정을 저장하지 않는다는 점입니다. 이 점이 중요한 특징 중 하나인데, MapReduce에서는 Map을 거치고 Combiner를 거친 중간과정 값을 디스크에 저장을 하게 됩니다. 그리고 여기서 디스크 I/O가 일어나기에 속도를 많이 소모하죠.

하지만 Spark는 다릅니다. Spark의 큰 장점 중 하나인 인메모리(In-Memory)를 유지하면서 Transformation 진행합니다. 여기서 의문이 들죠

**어떻게 많은 양의 데이터 한꺼번에 메모리에 올릴 수 있지?**

답은 **"올리지 않는다"** 입니다. 말 그대로 스파크는 Transformation을 사용했을 때 실제로 메모리에 올리지 않습니다. 바로 Action이 실행되었을 경우에만 Transformation을 실행하는 거죠.

Spark에서 Transformation을 실행하면 내부의 메타데이터에 해당
Transformation을 순차적으로 쌓아놓습니다. 자료구조로 생각하면 큐에 저장된다고 생각하시면 편할 것 같습니다. 그 후 Action이 실행되는 순간 메타데이터의 해당 큐에 저장 된 명령들을 차례로 실행하며 Filtering 혹은 Mapping을 진행합니다.

이 방식을 이용하면 굳이 디스크에 중간과정을 저장하면서 다시 읽어오는 디스크 I/O가 일어나지 않고 한 번만 읽어들이더라도 충분합니다.

필요하다면 메모리에 영속화(persist)를 하여 메모리에 남겨놓을 수도 있죠.

당연히 디스크 I/O가 빠진 Spark는 MapReduce보다 월등한 성능을 보입니다.

### 2. [MacOS] Spark 설치

Mac에서 Spark를 쉽게 다운로드 받기 위해서는 brew를 사용하는게 가장 쉽습니다.

![ex_screenshot](./post_img/Spark-cmd.png)

```
brew install apache-spark
```

다음 명령어를 통해 아파치 스파크를 다운받으면 곧바로 spark를 사용할 수 있습니다. pyspark를 사용하려면 **.zshrc** 파일에 다음 문장을 추가해주세요 PATH 설정입니다.

```
export PYTHONPATH=$SPARK_HOME/python:$SPARK_HOME/python/build:$PYTHONPATH
export PYTHONPATH=$SPARK_HOME/python/lib/py4j-0.10.4-src.zip:$PYTHONPATH
```

추가적으로 pyspark에 접근 시 주피터 노트북을 이용하고 싶으시면 다음의 문장을 추가해주시면 곧바로 접근 할 수 있습니다.

```
export PYSPARK_DRIVER_PYTHON=jupyter
export PYSPARK_DRIVER_PYTHON_OPTS='notebook'
```

다음의 과정을 거치고 cmd를 통해 pyspark에 접근하면 성공적으로 jupyter notebook을 통해 spark를 사용 할 수 있습니다.

![ex_screenshot](./post_img/Spark-Jupyter.png)

포스트 추가
