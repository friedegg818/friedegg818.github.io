---

title: "[Apache Arrow] 06.Apache Arrow Flight-2" 

excerpt: Apache Arrow Flight 관련 정보  

categories: 
    - Apache Arrow
tags:
    - [Apache Arrow, Apache Arrow Flight]

toc: true
toc_sticky: true

date: 2022-01-26
last_modified_At: 2022-01-27

---

## Arrow Flight RPC 
- 데이터 응용 프로그램 생성을 간소화하기 위해 RPC 계층이 필요 
- Flight는 다른 서비스에서 다운로드 / 업로드 되는 Streams of Arrow record batches를 중심으로 구성됨

## Arrow Messaging Paradigm 

### 1. Batch Streams 

<p align="center"><img src="/assets/img/BatchStreams.png"></p>

#### Primary Communication 
- A Stream of Arrow record batche
- 효율적인 이동을 목표로 하는 Bulk transfer 
- 효과적인 Peer to Peer 

#### Specific Method 
- Put Stream : 클라이언트가 서버로 Stream을 보냄 
- Get Stream : 서버가 클라이언트에게 Stream을 보냄 
- 모두 클라이언트에 의해 시작됨 

<br>

### 2. Stream Management

<p align="left"><img src="/assets/img/Stream Management.png"></p>

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

<br>

### 3. Data as a Service Customization 
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
- Flight는 
  + 데이터 업로드/다운로드, 
  + 데이터 스트림에 대한 메타데이터 검색, 
  + 사용 가능한 데이터 스트림 나열 및 응용 프로그램별 RPC 메서드 구현을 위한 RPC 메서드 집합을 정의 

- Flight service는 이러한 메소드 중 일부를 구현하지만, Flight Client는 어떤 메소드든 호출할 수 있음       
  → Flight Client는 모든 Flight service에 연결하여 기본 운영 가능 

- 데이터 스트림은 descriptor (path or an arbitrary binary command) 로 식별됨 

### Download the data 

1. 관심 있는 데이터 세트에 대한 `FlightDescriptor`을 구성하거나 얻음         
  클라이언트는 자신이 원하는 descriptor를 이미 알고 있거나, `ListFlights`와 같은 메서드를 사용하여 해당 설명자를 검색할 수 있음 

2. 데이터가 있는 위치 (ex.타 메타데이터, 스키마 및 데이터 세트 크기 추정치) 에 대한 세부 정보가 포함된 `FlightInfo` 메시지를 가져오려면 `GetFlightInfo(FlightDescriptor)`를 호출 

> Flight에서는 데이터가 메타데이터와 동일한 서버에 있을 필요가 없음           
> 이 호출은 연결할 다른 서버를 보여줄 수 있음            
> `FlightInfo` 메시지에는 서버가 요청 중인 정확한 데이터 세트를 식별하는데 사용하는 Ticket (An opaque binary token) 이 포함됨 

3. 다른 서버에 연결 (필요한 경우)

4. Arrow record batches 스트림을 가져오려면 `DoGet(Ticket)` 호출 

### Upload the data

1. 이전과 같이 `FlightDescriptor`를 구성하거나 얻음 

2. `DoPut(FlightData)`을 호출하고 Arrow 레코드 배치 스트림을 업로드      
    첫 번째 메시지와 함께 `FlightDescriptor`도 포함됨 



## Error Handling 
- Arrow Flight는 자체 오류 코드 집합을 정의 
- 구현은 언어마다 다름         
  (ex.C++에서 구현되지 않는 것은 일반적인 Arrow 오류 상태지만, Java에서는 Flight-specific exception)

### Error code 

|Error Code|Description|
|:---:|:---:|
|UNKNOWN|알 수 없는 오류. 다른 오류가 적용되지 않는 경우의 기본값|
|INTERNAL|서비스 구현 내부에 오류 발생|
|INVALID_ARGUMENT|클라이언트가 잘못된 인수를 RPC에 전달|
|TIMED_OUT|작업 시간 or 마감 시간 초과
|NOT_FOUND|요청한 리소스(action,data stream)를 찾을 수 없음|
|ALREADY_EXISTS|리소스가 이미 존재|
|CANCELLED|클라이언트 or 서버에 의해 작업 취소|
|UNAUTHENTICATED|클라이언트가 인증되지 않음|
|UNAUTHORIZED|클라이언트가 인증되었지만 요청한 작업에 대한 사용 권한이 없음|
|UNIMPLEMENTED|RPC가 구현되지 않음| 
|UNAVAILABLE|서버 사용 불가. 연결상의 이유로 클라이언트가 내보낼 수 있음|


## Protocol Buffer Definitions

### Service
```java
service  FlightService 
```
- Arrow 데이터를 검색하거나 저장하기 위한 endpoint 
- Arrow Flight Protocol을 사용하여 액세스할 수 있는 미리 정의된 endpoint를 하나 이상, 이용 가능한 일련의 actions를 노출시킬 수 있음 


### RPC
```java
rpc Handshake(stream HandshakeRequest) returns (stream HandshakeReponse) {}
```
- 클라이언트 - 서버 간의 Handshake
- 서버에서는 향후 작업에 사용할 토큰을 결정하기 위해 handshake가 필요할 수 있음 
- request / response는 모두 인증 메커니즘에 따라 여러 round-trips를 허용하는 스트림 

```java
rpc ListFlights(Criteria) returns (stream FlightInfo) {}
```
- 특정 조건이 지정된 사용 가능한 스트림 목록을 가져옴 
- 대부분의 Flight service는 쉽게 검색할 수 있는 하나 이상의 스트림을 노출함 
- 사용자가 Criteria를 제공할 수도 있음 → 이 기준은 이 인터페이스를 통해 나열될 수 있는 스트림의 하위 집합을 제한할 수 있음 
- 각 Flight service는 how to consume criteria에 대한 자체 정의를 허용 

```java
rpc GetFlightInfo(FlightDescriptor) returns (FlightInfo) {}
```
- 주어진 FlightDescriptor의 경우 flight 사용 방법에 대한 정보를 얻음 
- 이는 인터페이스 소비자가 이미 사용할 특정 flight를 식별할 수 있는 경우, 유용
- 이 인터페이스는 또한 소비자가 지정된 descriptor를 통해 flight 스트림을 생성할 수 있게 함 

```java
rpc GetSchema(Flight Descriptor) returns (SchemaResult) {} 
```
- 지정된 FlightDescriptor의 경우, Schema.fbs::Schema에 설명된대로 스키마를 가져옴 
- 이것은 사용자가 Flight 스트림의 스키마가 필요할 때 사용됨 
- `GetFlightInfo`와 마찬가지로 이 인터페이스는 `ListFlight`에서 이전에 사용할 수 없었던 new flight를 생성할 수 있음 

```java
rpc DoGet(Ticket) returns (stream FlightData) {}
```
- 참조된 ticket과 연관된 특정 descriptor와 연결된 단일 스트림을 검색 
- Flight는 flight service가 스트림 모음을 관리하기 위한 별도의 opaque ticket을 사용하여 각각의 스트림을 검색하는 하나 이상의 스트림으로 구성될 수 있음

```java
rpc DoPut(stream FlightData) returns (stream PutResult) {}
```
- 특정 flight 스트림과 관련된 flight service에 스트림을 push 
- 이를 통해 flight service의 클라이언트는 데이터 스트림을 업로드할 수 있음 
- 특정 flight service에 따라, 클라이언트 사용자는 descriptor 또는 unlimited number만큼 단일 스트림을 업로드할 수 있음 
- 후자의 경우, 서비스는 모든 스트림이 업로드 되면 descriptor에 적용할 수 있는 'seal' action을 구현할 수 있음

```java
rpc DoExchange(stream FlightData) returns (streamFlightData) {}
```
- 지정된 descriptor에 대한 양방향 데이터 채널 open 
- 이를 통해 클라이언트는 단일 논리 스트림에서 임의의 Arrow 데이터와 애플리케이션별 메타데이터를 주고 받을 수 있음 
- `DoGet`/`DoPut`과 달리, 이 기능은 연산을 flight 서비스로 (storage가 아니라) offload하는 클라이언트에게 더 적합 

```java
rpc DoAction(Action) returns (stream Result) {}
```
- Flight 서비스는 잠재적인 `listFlights`, `GetFlightInfo`, `DoGet`, `DoPut` 연산 외에도 임의의 수의 간단한 작업을 지원할 수 있음 
- `DoAction`은 flight 클라이언트가 flight 서비스에 대해 특정 action을 수행할 수 있도록 함 
- Action에는 수행 중인 유형의 작업과 관련된 opaque request/response object가 포함됨

```java
rpc ListActions(Empty) returns (stream ActionType) {}
```
- Flight 서비스는 사용할 수 있는 모든 action 유형을 description과 함께 보여줌 
- 이를 통해 다양한 flight 사용자들은 flight 서비스의 기능을 이해할 수 있음 

### Message

```java
message HandshakeRequest { 

    // 정의된 프로토콜 버전 
    unit protocol_version = 1; 

    // 임의 인증/handshake 정보
    bytes payload = 2;
}
```
- 클라이언트가 handshake 시 서버에 제공하는 request

```java
message HandshakeResponse { 
  unit protocol_version = 1;
  bytes payload = 2;
}
```

```java
message BasidAuth {
  string username = 2;
  string password = 3;
}
```
- 간단한 인증을 위한 message 

```java
messageEmpty{}
```

```java
messageActionType {
  string type = 1; 
  string description = 2; 
}
```
- 실행에 사용되는 이름과 action의 목적에 대한 shot description을 포함하여 사용할 수 있는 action에 대해 설명 

```java
message Criteria {
    bytes expression = 1;
}
```
- 사용 가능한 Arrow Flight 스트림의 제한된 집합을 반환하는데 사용할 수 있는 서비스별 표현식 

```java 
message Action {
  string type = 1;
  bytes body = 2;
}
```
- 서비스에 대한 opaque action 

```java
message Result {
    bytes body = 1;
}
```
- action 실행 후 받는 opaque result 

```java
message SchemaResult {

  // Schema.fbs::Schema에서 설명된대로, 데이터 세트의 스키마
  bytes schema = 1; 
}
```
- getSchema call의 결과 

```java
message FlightDescriptor {

    // 정의된 descriptor 유형 
    enum DescriptorType {

        // 사용되지 않은 Protobuf 패턴 
        UNKNOWN = 0; 

        // 데이터 집합을 식별하는 named path.
        // path는 데이터 세트를 설명하는 문자열 or 문자열 목록으로 구성됨 
        // 개념적으로 파일 시스템 내부의 경로와 유사 
        PATH = 1;

        // 데이터 세트를 생성하기 위한 opaque command 
        CMD = 2; 
    }

    DescriptorType type = 1; 

    // 명령을 표현하는데 사용되는 opaque value 
    // type = CMD인 경우에만 정의해야 함 
    bytes cmd = 2; 

    // 특정 데이터 집합을 식별하는 문자열 목록 
    // type = PATH인 경우에만 정의해야 함 
    repeated string path = 3;
}
```

```java
message FlightInfo {

  // Schema.fbs::Schema에서 설명된대로, 데이터 세트의 스키마 
  bytes schema = 1; 

  // 이 정보와 관련된 descriptor
  FlightDescriptor flight_descriptor = 2; 

  // Flight와 관련된 endpoint list. flight를 완전히 사용하기 위해서는 모든 endpoint를 사용해야 함 
  repeated FlightEndpoint endpoint = 3;

  // 알 수 없는 경우에는 -1로 설정 
  int64 total_records = 4; 
  int64 total_bytes = 5
}
```
- 데이터 세트를 검색하기 위한 access 좌표 
- `FlightInfo`를 통해 사용자가 데이터 세트를 검색하는 방법을 결정할 수 있음 

```java
message FlightEndpoint {

  // 이 스트림을 찾기 위해 사용되는 token 
  Ticket ticket = 1; 

  // 이 Ticket을 사용할 수 있는 URI list
  // 리스트가 비어있는 경우 ticket이 생성된 현재 서비스에서만 ticket을 교환할 수 있음
  repeated location location = 2;
}
```
- Flight와 관련된 특정 스트림 or 분할(split)

```java
message Location {
  string uri = 1;
}
```
- Flight service가 ticekt이 주어진 특정 스트림의 검색을 허용하는 위치 

```java
message Ticket {
  bytes ticket = 1; 
}
```
- 서비스에서 스트림의 특정 부분을 검색하는데 사용할 수 있는 opaque identifier

```java
message FlightData {

  // descriptor of data. 클라이언트가 새 DoPut 스트림을 시작할때만 관련 
  FlightDescriptor flight_descriptor = 1;

  // Message.fbs::Message에서 설명한대로 message 데이터의 header
  bytes data_header = 2;

  // 애플리케이션 정의 메타데이터 
  bytes app_metadata = 3; 

  // Arrow 데이터의 실제 batch 
  // Sidecar patter를 돕기 위해 정의에서 마지막에 나오는 최소 복사본으로 처리하는 것이 좋음 
  // (일부 구현은 추가 메모리 복사본을 피가히 위해 특수화된 코드로 이 필드를 wire에서 가져올 것으로 예상됨)
  bytes data_body = 1000;
}
```
- batch 스트림의 부분으로써의 Arrow batch

```java
message PutResult { 
  bytes app_metadata = 1;
}
```
- `DoPut` 제출과 관련된 response message 

***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/blog/2019/10/13/introducing-arrow-flight/>
- <https://arrow.apache.org/docs/format/Flight.html>
- <https://www.slideshare.net/JacquesNadeau5/apache-arrow-flight-overview>