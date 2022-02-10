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
- 다양하고 불명확한 lifetime rule로 raw data pointer를 전달하는 것을 방지하기 위해 Arrow는 <span style="color:	#00FFFF">arrow::Buffer</span>라는 generic abstraction을 제공 

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

#### <span style="color:#FF1493">class arrow::Buffer</span>
- 특정 크기의 연속적인 메모리에 대한 pointer를 포함하는 개체 
- size / capacity 라는 두가지 관련 개념 
> - size: 유효한 데이터를 가질 수 있는 바이트 수 
> - capacity: 총 버퍼에 할당된 바이트 수 
- 버퍼 기본 클래스는 메모리를 소유하지 않지만, 종종 subclass가 소유 
- Size <= Cpacity는 항상 참 
- arrow::cuda::CudaBuffer, arrow::MutableBuffer, arrow::py::NumPyBuffer, arrow::py::PyBuffer, arrow::py::PyForeignBuffer 로 서브클래싱 

##### Public functions 
```java
    inline Buffer(const uint8_t *data, int64_t size) 
```
- 메모리를 복사하지 않고 버퍼 및 크기에서 구성 
- 매개변수 
  + data - [in] 메모리 버퍼 
  + size - [in] 버퍼 크기 

```java
    inline explicit Buffer(util::string_view data) 
```
- 메모리를 복사하지 않고 string_view에서 구성 
- 매개변수 
  + data - [in] string_view 객체

```java
    inline Buffer(const std::shared_ptr<Buffer> &parent, const int64_t offset, const int64_t size)
```
- 다른 버퍼가 소유한 데이터의 offset이지만, parent buffer에 대한 다른 shared_ptr이 파괴된 이후에도 유효한 pointer를 유지하기를 원함 
- 이 방법은 버퍼의 정렬이나 패딩에 대한 assertion을 만들지 않지만 일반적으로 버퍼가 64바이트로 정렬되고 채워질 것으로 예상함 
- 앞으로 버퍼가 이 규칙을 충족하는지 확인하는데 도움이 되는 유틸리티 메서드를 추가할 수 있음 

```java
    std::string ToHexString() 
```
- 버퍼의 16진수 표현으로 새로운 std::string을 구성 
- Returns > std::string 

```java
    bool Equals(const Buffer &other, int64_t nbytes) const
```
- 두 버퍼가 동일한 크기이고, 비교된 바이트 수까지 동일한 바이트를 포함하는 경우 true를 반환 

```java
    bool Equals(const Buffer &other) const
```
- 두 버퍼가 같은 크기이고 동일한 바이트를 포함하는 경우 true 반환 

```java
    Result<std::shared_ptr<Buffer>> CopySlice(const int64_t start, const int64_t nbytes, MemoryPool *pool = default_memory_pool()) const
```
- 버퍼의 section을 새 버퍼에 복사 

```java
    inline void ZeroPadding() 
```
- Zero bytes in padding, 즉 size_와 capacity_ 사이의 바이트 

```java
    std::string ToString() const 
```
- 버퍼 내용을 새로운 std::string에 복사 
- Returs > std::string 

```java
    inline explicit operator util::string_view() const 
```
- 버퍼 내용을 util::string_view로 봄 
- Returns > utill::string_view 

```java
    inline explicit operator util::bytes_view() const
```
- 버퍼 내용을 util::bytes_view()로 봄 
- Returns > util::bytes_view

```java
    inline const uint8_t *data() const 
```
- 버퍼의 데이터에 대한 pointer를 반환 
- 버퍼는 CPU 버퍼여야 함 <span style="color:#00FFFF">(is cpu()가 true)</span>
- 그렇지 않으면 assertion이 발생하거나 null pointer가 반환될 수 있음 
- 장치에 관계없이 버퍼의 데이터 주소를 얻으려면 <span style="color:#00FFFF">address()</span>호출 

***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/docs/cpp/memory.html>
- <https://arrow.apache.org/docs/cpp/api/memory.html>