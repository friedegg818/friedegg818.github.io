---

title: "[Apache Arrow] 08.Arrow C++ Library-2" 

excerpt: Memory Management 관련 사항 및 API

categories: 
    - Apache Arrow
tags:
    - [Apache Arrow, C++ Library, Memory Management, API]

toc: true
toc_sticky: true

date: 2022-02-10
last_modified_At: 2022-02-10

---

# Memory Management 

## Buffers 
- 다양하고 불명확한 lifetime rule로 raw data pointer를 전달하는 것을 방지하기 위해 Arrow는 <span style="color:	#7B68EE">arrow::Buffer</span>라는 generic abstraction을 제공 

### 버퍼는..
- Pointer와 data size를 캡슐화하고, lifetime을 기본 공급자의 lifetime과 연결          
- 삭제될 때까지 항상 유효한 메모리를 가리켜야 함 
- 유형화 되지 않음 (형식을 지정하지 않음)
- 의도한 의미나 해석과는 상관없이 물리적 메모리 영역을 나타냄 
- Arrow 자체 또는 third-party 루틴에 의해 할당됨 
> 예를 들어, Python bytesting 데이터를 Arrow 버퍼로 전달하여 필요에 따라 Python 객체를 활성 상태로 유지하는 것이 가능함 
- 변경 가능 여부, 크기 조정 여부 등 다양한 형태로 제공됨 
- 일반적으로 데이터를 구성할 때 변경 가능한 버퍼를 보유하고 나면, array와 같은 변경할 수 없는 컨테이너로 고정됨 

### 버퍼 메모리 액세스 
- 버퍼는 size()와 data() 접근자를 사용하여 기본 메모리에 대한 빠른 접근을 제공 
- 또는 변경 가능한 버퍼에 쓸 수 있는 액세스인 mutuable data() 사용 

### 관련 API 

#### <span style="color:#DC143C">class arrow::Buffer</span>


***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/docs/cpp/memory.html>
- <https://arrow.apache.org/docs/cpp/api/memory.html>