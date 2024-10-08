# Rough Size Estimation 

개략적인 규모 추정은 보편적으로 통용되는 성능 수치상에서 사고 실험을 행하여 추정치를 계산하는 행위 
- 어떤 설계가 요구 사항에 부합할 것인지 보기 위한 것 
- 규모 확장성을 표현하는 데 필요한 기본기에 능숙해야 함 
  - 2의 제곱수, 응답지연(latency)값, 가용성에 관계된 수치들 

## 2의 제곱수 

데이터 볼륨의 단위를 2의 제곱수로 표현하면 어떻게 되는지를 알아보자 
- 최소 단위 : 1byte(8bits) 
- ASCII 문자 하나가 차지하는 메모리 크기 : 1byte 

| 2의 x 제곱 | 근사치   | 이름        | 축약형 |
|---------|-------|-----------|-----|
| 10      | 1천    | 1Kilobyte | 1KB | 
| 20      | 1백만   | 1Megabyte | 1MB | 
| 30      | 10억   | 1Gigabyte | 1GB | 
| 40      | 1조    | 1Terabyte | 1TB | 
| 50      | 1000조 | 1Petabyte | 1PB | 

## 응답지연 값 

| 연산명      | 시간  | 
|----------|-----|
| L1 캐시 참조 | 0.5ns | 
| 분기 예측 오류 (branch mispredict) | 5ns | 
| L2 캐시 참조 | 7ns | 
| 뮤텍스(mutex) 락/언락 | 100ns | 
| 주 메모리 참조 | 100ns | 
| Zippy로 1 KB 압축 | 10,000ns = 10us | 
| 1 Gbps 네트워크로 2KB 전송 | 20,000ns = 20us | 
| 메모리에서 1 MB 순차적으로 read | 250,000ns = 250us | 
| 같은 데이터 센터 내에서의 메시지 왕복 지연시간 | 500,000ns = 500us | 
| 디스크 탐색(seek) | 10,000,000ns = 10ms | 
| 네트워크에서 1 MB 순차적으로 read | 10,000,000ns = 10ms | 
| 디스크에서 1 MB 순차적으로 read | 30,000,000ns = 30ms | 
| 한 패킷의 CA(캘리포니아)로부터 네덜란드까지 왕복 지연 시간 | 150,000,000ns = 150ms | 

> ns = nanosecond, us = microsecond, ms = millisecond
> 
> 1ns = 10^-9 sec, 1us = 10^-6 sec, 1ms = 10^-3 sec 

<br>

### 결론 

- 메모리는 빠르지만, 디스크는 아직도 느리다. 
- 디스크 탐색(seek)는 가능한 피하라. 
- 단순한 압축 알고리즘은 빠르다. 
- 데이터를 인터넷으로 전송하기 전에 가능하면 압축해라. 
- 데이터 센터는 보통 여러 지역(region)에 분산되어 있고, 센터들 간에 데이터를 주고받는 데는 시간이 걸린다. 

## 가용성에 관계된 수치들 

고가용성(high availability) : 시스템이 오랜 시간 동안 지속적으로 중단 없이 운영될 수 있는 능력 
- 고가용성을 표현하는 값은 퍼센트로 표현(%) 
- 100%는 시스템이 단 한 번도 중단된 적이 없음 
- 대부분 99% ~ 100% 사이의 값 

<br>

SLA(Service Level Agreement) : 서비스 사업자(service provider)가 보편적으로 사용하는 용어, 서비스 사업자와 고객 사이에 맺어진 합의 
- 이 합의에는 서비스 사업자가 제공하는 서비스의 가용시간(uptime)이 공식적으로 기술되어 있다. 
- 아마존, 구글, 마이크로소프트같은 사업자는 99% 이상의 SLA를 제공 
- 가용 시간은 관습적으로 9를 사용해 표현, 9가 많을 수록 좋다. 

### 9와 시스템 장애시간(downtime) 사이의 관계 

| 가용률      | 하루당 장애시간  | 주당 장애시간 | 개월당 장애시간 | 연간 장애시간 |
|----------|-----------|-------|-------|-------|
| 99%      | 14.40분    | 1.68시간 | 7.31시간 | 3.65일 |
| 99.9%    | 1.44분     | 10.08분 | 43.83분 | 8.77시간 |
| 99.99%   | 8.64초     | 1.01분 | 4.38분 | 52.60분 |
| 99.999%  | 864.00밀리초 | 6.05초 | 26.30초 | 5.26분 |
| 99.9999% | 86.40밀리초  | 604.80밀리초 | 2.63초 | 31.56초 |

## 예제: 트위터 QPS와 저장소 요구량 추정 

가정
- 월간 능동 사용자(monthly active user)는 3억(300million)명 
- 50%의 사용자가 트위터를 매일 사용 
- 평균적으로 각 사용자는 매일 2건의 트윗을 올림 
- 미디어를 포함하는 트윗은 10% 정도 
- 데이터는 5년간 보관 

<br>

추정 
- QPS(Query Per Second) 추정치 
  - 일간 능동 사용자(Daily Active User, DAU) = 3억 x 50% = 1.5억(150 million)
  - QPS = 1.5억 x 2 트윗 / 24 시간 / 3600초 = 약 3500
  - 최대 QPS (Peek QPS) = 2 x QPS = 약 7000 
- 미디어 저장을 위한 저장소 요구량 
  - 평균 트윗 크기 
    - tweet_id에 64바이트 
    - 텍스트에 140바이트 
    - 미디어에 1MB 
  - 미디어 저장소 요구량 = 1.5억 x 2 x 10% x 1MB = 30TB/일 
  - 5년간 미디어를 보관하기 위한 저장소 요구량 = 30TB x 365 x 5 = 약 55 PB
