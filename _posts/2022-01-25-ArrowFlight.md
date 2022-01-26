---

title: "[Apache Arrow] 05.Apache Arrow Flight" 

excerpt: Apache Arrow Flight 관련 정보  

categories: 
    - Apache Arrow
tags:
    - [Apache Arrow, Apache Arrow Flight]

toc: true
toc_sticky: true

date: 2022-01-25
last_modified_At: 2022-01-25

---

## Apache Arrow Flight 
- 빠른 데이터 전송을 위한 새로운 범용 클라이언트-서버 프레임워크 
- gRPC를 통한 Arrow columnar format의 최적화된 전송에 중점을 둠        
  (gRPC와의 통합에 중점을 두긴 하지만, gRPC 전용은 아님)
- 다른 데이터 전송 프레임워크와의 가장 큰 차별점 → **Parallel transfer (병렬 전송)**

> - 데이터를 Server cluster 로/로부터 (to/from) 동시에 스트리밍 가능      
>    → 증가하고 있는 클라이언트 기반 서비스를 제공할 수 있는 확장 가능한 데이터 서비스를 보다 쉽게 만들 수 있음 

- 0.15.0 Apache Arrow Release 부터 사용 가능 



## 설계 목표 
- 개발자에게 제공되는 공개 API 뿐만 아니라, 무선 데이터 표현으로 Arrow columnar format을 사용하는 데이터 서비스를 위한 새로운 프로토콜을 만드는 것 
- 이는 데이터 전송과 관련된 직렬화 비용을 줄이거나 없애고 분산 데이터 시스템의 전반적인 효율성을 높임 



## Apache Arrow Flight 기본 사항 
- Arrow Flight 라이브러리는 데이터 스트림을 주고 받을 수 있는 서비스를 구현하기 위한 개발 프레임워크를 제공 

### Basic request 

> - **Handshake** : 클라이언트 인증 여부를 확인하고, 경우에 따라 향후 요청에 사용할 implementation-defined session token을 설정하기 위한 간단한 요청 
> - **ListFlights** : 사용 가능한 데이터 스트림 목록 반환 
> - **GetSchema** : 데이터 스트림에 대한 스키마 반환 
> - **GetFlightInfo** : 관심 데이터 세트에 대한 "access plan"을 반환하며, 여러 데이터 스트림을 사용해야 할 수도 있음 (ex. 특정 애플리케이션 매개 변수 등을 포함하는 사용자 지정 명력을 수락할 수 있음)
> - **DoGet** : 클라이언트에게 데이터 스트림 전송 
> - **DoPut** : 클라이언트로부터 데이터 스트림 수신 
> - **DoAction** : 구현별 action을 수행하고 결과를 반환 (ex.일반화된 함수 호출)
> - **ListActions** : 사용 가능한 action type list 반환
<p align="center"><img src="/assets/img/Flight_1.png"></p>

- gRPC의 HTTP/2 스트리밍을 기반으로 하는 gRPC의 고급 "양방향" 스트리밍 지원을 활용하여 요청이 처리되는 동안 클라이언트와 서버가 서로 데이터와 메타데이터를 동시에 전송할 수 있음 
- 간단한 Flight 설정은 클라이언트가 연결하여 `DoGet` 요청을 하는 단일 서버로 구성될 수 있음 



## gRPC를 통한 데이터 처리량 최적화 
- gRPC와 같은 범용 메시징 라이브러리를 사용할 경우 많은 이점이 있지만, 대규모 데이터 세트의 전송 성능 향상을 위해서는 일부 작업이 필요함 (대부분의 gRPC 사용자들은 상대적으로 작은 메시지만 다룸)

### Protocol Buffers 
- 가장 잘 지원되는 gRPC 사용 방법은 Protocol Buffers (Protobuf) `.proto` 파일에서 서비스를 정의하는  것 
- gRPC용 Protobuf 플러그인은 애플리케이션 구현에 사용할 수 있는 gRPC service stub을 생성함 
- RPC 명령 및 데이터 메시지는 Protobuf wire format을 사용하여 직렬화됨 
- Apache Arrow Flight는 **"vanilla gRPC and Protocol Buffers"** 를 사용       
  → Arrow columnar 형식을 모르는 gRPC 클라이언트도 Flight 서비스와 상호 작용하고 Arrow 데이터를 opaquely 하게 처리할 수 있음 

### FlightData 
- Flight의 주요 데이터 관련 Protobuf 유형 
- 일반적으로 Protobuf 메시지를 읽고 쓰는 것은 자유롭지 않기 때문에, 다음을 수행하기 위해 C++와 Java에서 gRPC에 몇 가지 low-level optimizations를 구현함 

> - 중간 메모리 복사 or 직렬화 단계를 처지지 않고 전송되는 Arrow 레코드 배치를 포함하여 `FlightData` 에 대한 Protobuf wire format을 생성 
>- 메모리 복사 or 역직렬화 없이 `FlightData`의 Protobuf 표현에서 Arrow 레코드 배치를 재구성 

- 이렇게 최적화된 Flight 구현은 더 나은 성능을 발휘할 수 있음 




***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/blog/2019/10/13/introducing-arrow-flight/>