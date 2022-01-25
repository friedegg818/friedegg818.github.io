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
- 지원하는 basic request 

> - Handshake: 클라이언트 인증 여부를 확인하고, 경우에 따라 향후 요청에 사용할 implementation-defined session token을 설정하기 위한 간단한 요청 



***

### <span style="color:#00CCCC">References</span>
