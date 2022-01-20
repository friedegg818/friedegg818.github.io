---

title: "[Apache Arrow] 02.Relation to other projects" 

excerpt: Arrow 파일 형식 및 타 라이브러리와의 관계

categories: 
    - Apache Arrow
tags:
    - [Apache Arrow, Apache Pqrquet, Protobuf, Flatbuffer]

toc: true
toc_sticky: true

date: 2022-01-13
last_modified_At: 2022-01-14

---

## Apache Arrow vs. Apache Parquet

### Parquet 
- Runtime in-memory format이 아님 
- 고급 압축 및 인코딩 기술을 사용하여 공간 효율성을 극대화하도록 설계된 Storage format         
→ Gigabyte 이상의 데이터를 저장하면서 디스크 사용량을 최소화하려는 경우에 이상적 
- Parquet 데이터는 큰 chunk로 디코딩해야하기 때문에 메모리 읽기 비용이 상대적으로 많이 소요됨 

### Arrow 
- 연산 목적을 위해 직접적이고 효율적인 사용을 위한 In-memory format 
- Arrow 데이터는 압축되지 않고 (혹은 사전 인코딩을 사용할 때 가볍게 압축) CPU에 대해 natural format으로 배치되어 데이터가 임의의 위치에서 전속력으로 access 될 수 있음 

### 결론 
- Arrow와 Parquet은 완전히 다르고, 일반적으로 서로를 보완하며 응용 프로그램에서 함께 사용됨 
- Parquet을 사용하여 디스크에 데이터를 저장하고, Arrow 형식으로 메모리를 읽어들이면 컴퓨팅 하드웨어를 최대한 활용할 수 있음 


## Arrow files 

### IPC Mechanism
- Inter-process communication mechanism 
- Arrow columnar arrays 모음(레코드 배치)을 전송 
- Arrow **"stream format"** 을 사용하는 프로세스 간에 **동기식**으로 사용하거나, 
- Arrow **"file format"**을 사용하여 먼저 Storage에 데이터를 유지함으로써 **비동기식**으로 사용할 수 있음 
- Arrow in-memory format을 기반으로 하므로 on-disk 표현과 in-memory 표현 간에 변환이 필요하지 않음     
→ Arrow IPC 파일에 대한 분석을 수행하면 memory mapping을 사용하여 역직렬화 비용과 추가 copy를 피할 수 있음 

### Arrow IPC file format과 Parquet format 비교 시 고려 사항 
1. Parquet은 **장기 저장 및 보관 목적**으로 설계
   Arrow on-disk format은 안정적이며 향후 버전 라이브러리에서 읽을 수 있지만, 장기 보관 저장소의 요구 사항을 우선시하지는 않음 

2. Parquet 파일을 읽으려면 비교적 **복잡한 디코딩**이 필요
   Arrow IPC 파일을 읽는데에는 디코딩이 필요하지 않음 

3. Parquet 파일은 Parquet이 사용하는 columnar data **압축 전략**때문에 Arrow IPC 파일보다 **훨씬 작은 경우가 많음**         
→ 디스크 Storage나 네트워크가 느린 경우, 단기 Storage나 캐싱도 Parquet이 더 나은 선택이 될 수 있음 


## Feather file format 

### Feather v1 format 
- Arrow IPC file format이 개발되기 전에 Arrow format의 하위 집합을 디스크에 쓰기 위해 단순화된 사용자 정의 컨테이너

### Feather version 2 
- 정확한 Arrow IPC file format
- 이전 버전과의 호환성을 위해 **"Feather"** 이름과 API 유지 



## Arrow와 Protobuf / Flatbuffers

### Protobuf 
- Google의 프로토콜 버퍼 라이브러리 
- Runtime in-memory format이 아님 
- 데이터 처리에 적합하지 않음        
  → 데이터 처리를 위해 Arrow와 같은 In-memory 표현으로 역직렬화 되어야 함       
  → 이러한 역 직렬화를 수행하는 라이브러리가 있지만 공통 In-memory format을 목표로 하지는 않아서 한 언어에서 다른 언어로 데이터를 마샬링해야 함         
          
    ``` 
     * 마샬링: 한 객체의 메모리에서의 표현 방식을 
              저장 또는 전송에 적합한 다른 데이터 형식으로 변환하는 과정
    ``` 
    ``` 
      왜 안나올까요 
    ```

 - Arrow는 이러한 과정을 거치지 않아도 되지만, 대신 공간이 더 늘어나게 됨 
 - Protobuf는 유선상의 특정 종류의 데이터를 직렬화하는데 더 나은 선택이 될 수 있음           
   (ex.개별 레코드 또는 많은 선택적 필드가 있는 Spares data)

 **Parquet과 마찬가지로 Arrow와 Protobuf는 서로를 잘 보완하는 관계!**

 ### Flatbuffers
 - Binary data 직렬화를 위한 low-level building block 
 - 크고 구조화된 동질 데이터 표현에 적합하지 않으며, 데이터 분석 작업을 위한 올바른 추상화 계층에 있지 않음 


 - Arrow는 데이터 분석 요구 사항을 직접적으로 겨냥한 데이터 계층 
 - 분석에 필요한 포괄적인 data type 컬렉션을 제공 
 - "null" 값에 대한 내장 지원 
 - I/O 및 컴퓨팅의 확장 toolbox를 제공 


- **Arrow file format**은 Arrow binary IPC protocol을 구현하는데 필요한 스키마 및 기타 메타데이터를 직렬화하기 위해 hodd에서 Flatbuffers를 사용하지만, **Arrow data format**은 최적의 액세스 및 계산을 위해 자체 표현을 사용함

***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/>
