---
title : "Recent Software Flash Challenge at work"

---
파라미터, 오랜 문제
===
현 직장에서 SW 개발에 가장 비효율을 낳고 있는 것은 무엇일까. 나는 누가 물어보면 바로 '파라미터 관리'라고 얘기할 것이다.

우리 회사는 파라미터 관리를 위해 #ifdef 문법을 사용한다. 그게 무슨 말인고 하면,
```C
//#define MODEL_A
#define MODEL_B 
//#define MODEL_C
...
{
    uint8_t parameter_a;

#ifdef MODEL_A
    parmemter_a=5;
#endif

#ifdef MODEL_B
    parameter_a=8;
#endif

#ifdef MODEL_C
    parameter_a=138;
#endif
...
}
```
이런 식으로 #define 으로 SW에 기종명을 정의하고, 그에 따라 #ifdef, #endif 를 사용하는 식이다. 이 자체로는 문제가 없지만...

우리 회사는, 

사실 1: 매크로 상수를 수작업으로 수정하고 Build 한다.  
사실 2: 파라미터를 한 개의 .c 파일에 몰아 넣어, 그 파일이 수 만줄이 넘는다.    
사실 3: Parameter를 수정하려면 Build를 수행해야한다.  

그야말로 실수하기 쉽고, 관리가 어려운 상태인 것이다.


 문제 해결 방안
===
#### 1. 빌드 비효율 문제
makefile을 개선하면 Build 전에 직접 #define 을 하고 빌드하는 행위를 자동화할 수 있고, 여러 기종의 빌드를 동시에 진행할 수 있다.   
지금까지는 makefile을 팀에서 전수해준 그대로 보전하는 데에 힘을 썼지만... 

```
CFLAGS += -DMODEL_B

all: my_program

my_program: my_program.o
	$(CC) $(CFLAGS) -o my_program my_program.o

my_program.o: my_program.c
	$(CC) $(CFLAGS) -c my_program.c

```

이렇게하면, -D 플래그를 사용하여 MODEL_B 매크로를 정의하는 것이다. 

그러고보니 makefile에 대한 이해가 많이 부족했다. makefile 에 대한 공부도 추가로 했다.
나중에 따로 정리해야지...   

#### 2. PARAMETER 관리 비효율 문제
로직 image File 과 파라미터 image File 을 분리하면 파라미터만 변경해야하는 상황에서 일일이 로직을 수정할 필요가 없을 것이다.

SepaPartial flash 라는 기능인데 나의 경우 타겟 보드 메이커인 Freesacle의 flash controller 에서는 이렇게 작성이 되어있다. 

![PARTIALWRITING](/assets/images/2024-03/3_PartialWriting.png)

출처 : https://www.nxp.com/docs/en/user-guide/56800EFPUG.pdf

또한 타겟 보드의 Data Sheet를 보면 Flash memory를 4MB 제공하고, 현재 양산 기종들의 image file 이 2MB 이하이므로 충분히 Logic image와 Parameter image를 분리할 수 있다.   

임베디드 SW 개발에는 굉장히 일반적인 기능이라고 하는데, 우리 회사에서는 전혀 도입이 되어있지 않은 상태.

공부를 해서 좀 알아보니 시스템 메모리를 확인한 뒤에는 아래 과정을 따라야 한다.

1. 빌드 타겟 설정
프로젝트 설정에서 Logic Image와 Parameter Image를 분리하여 빌드하도록 설정한다.

2. 링크 스크립트 준비
링커 스크립트 파일인 .ld 파일을 LogicLinker.ld와 ParameterLinker.ld 로 분리하고 메모리 영역을 각각 지정한다.

3. 빌드 설정
LogicBuild 타겟과 ParameterBuild 타겟을 각각 선택하여 빌드한다.

그리곤 설정한 링커 스크립트에 따라 flash tool로 메모리 주소만 설정하여 flash 하면 되는 것이다.

당장 양산 코드에서 검토를 해보고 싶은데, 팀장님은 SW 담당자가 실수 안 하면 된다고 생각하시는듯 하다.

한번 직접 실습해보려고 STM 보드를 살까 했지만... 나중에 실전 기회가 있으면 그 때 잘 할 수 있을 것 같다.