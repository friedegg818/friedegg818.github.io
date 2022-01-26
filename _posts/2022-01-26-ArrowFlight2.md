---

title: "[Apache Arrow] 05.Apache Arrow Flight-2" 

excerpt: Apache Arrow Flight 관련 정보  

categories: 
    - Apache Arrow
tags:
    - [Apache Arrow, Apache Arrow Flight]

toc: true
toc_sticky: true

date: 2022-01-26
last_modified_At: 2022-01-26

---

## Arrow Flight RPC 
- 데이터 응용 프로그램 생성을 간소화하기 위해 RPC 계층이 필요 
- Flight는 다른 서비스에서 다운로드 / 업로드 되는 Streams of Arrow record batches를 중심으로 구성됨

## Arrow Messaging Paradigm 

### Batch Streams 

<p align="center"><img src="/assets/img/BatchStreams.png"></p>

#### Primary Communication 
- A Stream of Arrow record batche
- 효율적인 이동을 목표로 하는 Bulk transfer 
- 효과적인 Peer to Peer 

#### Specific Method 
- Put Stream : 클라이언트가 서버로 Stream을 보냄 
- Get Stream : 서버가 클라이언트에게 Stream을 보냄 
- 모두 클라이언트에 의해 시작됨 

### Stream Management

<p align="center"><img src="/assets/img/Stream Management.png"></p>

#### 병렬 consumption 및 locality 인식
- Flight는 Stream으로 구성 
- 각 Stream에는 FlightEndpoint 존재 
- 시스템이 위치 정보를 사용하여 데이터 locality가 향상됨 

#### Flight는 2개의 reference system 보유 
- 간단한 서비스를 위한 Dotted path namespace (ex.marketing.yesterday.sales)
- Arbitray binary command descriptor (ex."select a,b from foo where c>10")

#### Stream Listing 지원 
- ListFlights (Criteria)
- GetFlightInfo (FlightDescriptor)

### Data as a Service Customization 
- Arrow Flight는 간단한 Generic Messaging 프레임워크 지원       
  → Arrow Flight context 내에서 사용자 정의 및 확장성 지원 

#### ListActions() 
- 각 데이터 서비스가 지원하는 action에 대한 설명과 함께 action을 노출할 수 있음
- 각 action은 action과 그에 상응하는 결과를 구조화하는 방법을 설명해야 함 
- 일반 HTTP2 예외는 오류 상태를 관리하는데 사용할 수 있음 

#### DoAction(Action) => Result 
- 데이터 서비스 실행 관련 작업을 수행할 수 있는 Generic Containers         
  (ex. forget stream, load stream from disk)

#### Actions and Results 
- 각각은 ActionType String token과 JSON body of instruction을 가지고 있음 
- Arrow Flight 클라이언트는 custom Actions/Results에 대한 지식 없이 작성 가능 
  + 필요에 따라 데이터 서비스용 Lightweight wrappers 구축 가능 
  + 또는 일반 API 위에 기존 JSON tooling만 하면 됨 



## RPC Methods 



***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/blog/2019/10/13/introducing-arrow-flight/>
- <https://arrow.apache.org/docs/format/Flight.html>
- <https://www.slideshare.net/JacquesNadeau5/apache-arrow-flight-overview>