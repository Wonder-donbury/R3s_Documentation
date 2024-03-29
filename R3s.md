# R3s 정리 - by 동환

## History  

<span style="font-size:50">
2024-01-11 : 작성 시작 (0.01) - 이동환 </br>
2024-01-15 : 1/11 교육분 작성 완료 - 이동환
</span>

---

## 목차

---

> [1. 개요](#1-개요)  
> [2. 컴포넌트](#2-컴포넌트)  
>> <details>
>> <ul>
>>   <li><a href="#assumptions">Assumptions</a></li>
>>  <li><a href="#batches">Batches</a></li>
>>  <li><a href="#categories">Categories</a></li>
>>  <li><a href="#currencies">Currencies</a></li>
>>  <li><a href="#data-processes">Data processes</a></li>
>>  <li><a href="#database-views">Database views</a></li>
>>  <li><a href="#events">Events</a></li>
>>  <li><a href="#filters">Filters</a></li>
>>  <li><a href="#groupings">Groupings</a></li>
>>  <li><a href="#initialization-modules">Initialization modules</a></li>
>>  <li><a href="#layer-modules">Layer modules</a></li>
>>  <li><a href="#life-tables">Life Tables</a></li>
>>  <li><a href="#loops">Loops</a></li>
>>  <li><a href="#models">Models</a></li>
>>  <li><a href="#mtf-views">MtF Views</a></li>
>>  <li><a href="#output-reports">Output reports</a></li>
>>  <li><a href="#projected-data-processes">Projected data processes</a></li>
>>  <li><a href="#run-time-parameter-sets">Run-time parameter sets</a></li>
>>  <li><a href="#stochastic-processes">Stochastic processes</a></li>
>>  <li><a href="#user-functions">User functions</a></li>
>> </ul>
>> </details>
> [3. 변수](#3-변수)  
> [4. 모듈](#4-모듈)  
>
>> + 공통모듈
>> + 스탠다드 코드  
>> + 제안사항
>
> [5. 컴파일](#5-컴파일)  
> [6. 런 관련](#6-런-관련)  
> [7. Excel 익스텐션](#7-excel-익스텐션)  
> [8. 기타](#8-기타)  
---

## 1. 개요  

### 구조에 대해서

> 예를 들면, 부채 모댈의 경우에는 데이터 -> CF -> PV 레이어로 구성되어 있고,  
> 해약환급금 계산 작업의 경우에는 삼이원 모듈을 이용한다.
>
>>``` mermaid
>> flowchart LR
>>    a[Data \n Layer]
>>    b[CF \n Layer]
>>    c[PV \n Layer]
>>    a --> b -->c
>>```
>

### 데이터 소스와 컴파일/런타임 소요시간 간의 관계성

> ``` mermaid
> flowchart LR
>   a[데이터 소스]
>   b[Category]
>   c[컴파일 ++ 런타임 --]
>   d[컴파일 -- 런타임 ++]
>   a-->b
>   b--->|많아짐 ++|c  
>   b--->|적어짐 --|d  
> ```
>
> 일반적으로 일반적인 프로그램을 짜려고 할 수록 조건 제어문이 많아지기 때문에, 논리연산이 많아지면서 계산소요시간이 증가한다.

> ### [카테고리](#categories)란?
>
> ```mermaid
> flowchart TB
>   subgraph Category A
>       a[Data1] --> b[Program1] 
>   end
>   subgraph Category B
>       c[Data2] --> d[Program2]
>       e[Data3] --> d
>       f[Data4] -->  d
>       f --> b
>   end
>
>   subgraph Category C
>       g[Data5] --> h[Program3]
>       g-->b
>        g-->d
>   end
>  
> ```
>
> 데이터와 프로그램 간의 묶음쌍이라고 생각하면 된다.

### 프로그램 작성시 목적적합한 [모듈](#4-모듈)을 지정할 것

> 필요한 [모듈](#4-모듈)만으로 프로그램 구성

---

## 2. 컴포넌트  

### Assumptions  

> Model, Data Source, Projection layer, Module 모두에서 참조가 가능하다.
>>
>> ### 불러오는 법
>>
>> 1. `External assumption connection string` 클릭
>> 2. `locate database connect string`
>> 3. `데이터 연결 속성`
>> 4. `ODBC 선택` - 엑셀 Base 시
>> 5. `데이터 원본 선택` - 컴퓨터 데이터 원본
>> 6. `리더 선택` : (R3s : Algo FM, csv : CSV Reader, xlsx : Excel Files)
>> 7. `연결 테스트`

>> ### Assumption Set 내 설정
>>
>> 1. 주로 `.xlsx` 형식 사용 
>> 2. 엑셀 파일 내 이름 지정된 영역의 이름을 `Assumption Table`에 입력   
>> 2-1 혹은 `locate table`(우클릭) 사용  
>> 2-2 혹은 `formula` 직접 입력
>> 3. `Formula` 창에 인덱싱 및 `specify dimension`
>>
>>> **EX)**  
>>> |Dimension|Name|Start Position|Data Type|
>>> |:---------:|----|--------------:|:---------:|
>>> |1 차원|name 1|1|char|
>>> |2 차원|name 2|1|char|
>>> |...|...|...|...|
>>> |n 차원|name N|1|char|  
>>> 
>>> 최근자 코드부터 `Start Position` = 0이 표준임.


>> ### 주의사항
>>
>> 각 컴포넌트에서 호출 시마다 Assumption 데이터를 참조하게 되어 데이터 로드에 걸리는 시간이 길다.  
>>> 우회법: Initialization module에 한 번 호출 후 필요한 값에 대해서 계속 유지시키는 방법이 있다.  
>>`Assumption Variables` : `Portfolio` = N by default  
>>`Assumption` 변수 : (임의) Assumption Set과 관련된 엑셀파일 상대주소명.

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Batches  

>
>#### Batch 내 모델 간 리턴값 I/O  
>
>```mermaid
>   flowchart LR
>       subgraph Batch
>       model1 --> model1_result
>       model1_result --> model2
>       model2 --> model2_result
>       model2_result --> model3
>       end
>```
>
>한 모델 출력/저장 후 해당 파일을 다음 모델로 입력하는 방식
<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Categories


<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Currencies

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Data processes

> ### 데이터 불러오기 기능
>
>> `절대참조` : 주소 리터럴 기준.   
>> `상대참조` : wvr 파일 저장폴더 기준, 시스템 변수 `<Workspace_Folder>`를 사용함.   
>>> **Ex:** `<Workspace_Folder>`data.csv  ▶️ 해당 시스템 변수 스트링 -> 백슬래시(\\)로 끝남.
>>
>> #### Beware
>>
>> `Header lines to skip` : 헤더 있으면 헤더 행 수 만큼 skip  
>> `Decimal point` : 자릿수 구분 문자 [Comma - ','] , [Period - '.'] 중 택일.  
>> `유니코드 에러` : 영어, 숫자 사용 권장.  
>> `Record Start/End date`   :   
>> `Record Identifier` : Record Identifier로 지정된 열은 char로 읽어온다.

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---


### Database views


<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Events

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Filters

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Groupings

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Initialization modules

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>
---

### Layer modules

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Life Tables

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Loops

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Models

>#### DB 업로드
>
> 모델 속성 : `Database output` : `Internal`(엑셀), `Database`(DB)
<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### MtF Views

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Output reports

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Projected data processes

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Run-time parameter sets

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### Stochastic processes

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

### User functions

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

## 3. 변수

>### Rollback 변수
>
> 반대 스텝 방향으로 돌아가는 변수이다. **t시점**부터 **0시점**까지 -1스텝 씩 돌아간다.  

>### 포트폴리오 변수
>
> ![portfolio_00](portfoilo_00.jpg)
> <center> 모듈과 함께 돌아가는 변수  </center> </br>
>
> 두가지 의의가 있다.  
>
> + 각종 컴포넌트간에서 해당 데이터를 참조하기 위해 포트폴리오 변수로 지정해주어야 한다.
> + 직전기 정보를 저장하고 참조하기 위해서는 포트폴리오 변수로 지정해주는 것이 필수이다.  

>
>### 변수 참조 속성
>
>
> ```mermaid
>   %%{init: { 'logLevel': 'debug', 'theme': 'base', 'timeline': {'disableMulticolor': false}}}%%
>   timeline
>       title 변수 참조 방법
>       t-1 : B.Start 
>       t : B.End / A.start
>       t+1: A.End
>```
>
>
>#### `<변수명>.Prev`
>
> 포트폴리오 속성이 Y일 때, 직전 스텝의 값을 참조한다.
>
>#### `<변수명>.Start`
>
>#### `<변수명>.End`
>
>#### `<변수명>.Curr`
>
>#### `<변수명>.Total`

>#### 특정 시기의 정보를 참조하고 싶을 때
>
>> 해당 레이어 전체 복제 후 계산결과에서 참조.  
>> or 어레이를 생성하여 스택(Stack) 방식으로 `t-n` 시기의 값을 저장한다. 
>> 
>> 예시:
>>> 변수 5개 만들어서 5개 스텝 값 순환하며 리프레시  
>>> 아니면 Array 생성하여 5칸 계산


>### 시스템 변수
>

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

## 4. 모듈

변수들의 집합. 매 변수들에 값 혹은 수식을 할당함.

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

## 5. 컴파일

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

## 6. 런 관련

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

## 7. Excel 익스텐션

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---

## 8. 기타

<span style="font-size: 10px;">

[목차로 돌아가기](#목차)

</span>

---
