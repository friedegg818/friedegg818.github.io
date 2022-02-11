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
last_modified_At: 2022-02-11

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

#### <span style="color:#FF8C00">class arrow::Buffer</span>
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

  <br>

```java
    inline explicit Buffer(util::string_view data) 
```
- 메모리를 복사하지 않고 string_view에서 구성 
- 매개변수 
  + data - [in] string_view 객체
  
  <br>

```java
    inline Buffer(const std::shared_ptr<Buffer> &parent, const int64_t offset, const int64_t size)
```
- 다른 버퍼가 소유한 데이터의 offset이지만, parent buffer에 대한 다른 shared_ptr이 파괴된 이후에도 유효한 pointer를 유지하기를 원함 
- 이 방법은 버퍼의 정렬이나 패딩에 대한 assertion을 만들지 않지만 일반적으로 버퍼가 64바이트로 정렬되고 채워질 것으로 예상함 
- 앞으로 버퍼가 이 규칙을 충족하는지 확인하는데 도움이 되는 유틸리티 메서드를 추가할 수 있음 

  <br>

```java
    std::string ToHexString() 
```
- 버퍼의 16진수 표현으로 새로운 std::string을 구성 
- Returns > std::string 

  <br>

```java
    bool Equals(const Buffer &other, int64_t nbytes) const
```
- 두 버퍼가 동일한 크기이고, 비교된 바이트 수까지 동일한 바이트를 포함하는 경우 true를 반환 

  <br>

```java
    bool Equals(const Buffer &other) const
```
- 두 버퍼가 같은 크기이고 동일한 바이트를 포함하는 경우 true 반환 

  <br>

```java
    Result<std::shared_ptr<Buffer>> CopySlice(const int64_t start, const int64_t nbytes, 
    MemoryPool *pool = default_memory_pool()) const
```
- 버퍼의 section을 새 버퍼에 복사 

  <br>

```java
    inline void ZeroPadding() 
```
- Zero bytes in padding, 즉 size_와 capacity_ 사이의 바이트 

  <br>

```java
    std::string ToString() const 
```
- 버퍼 내용을 새로운 std::string에 복사 
- Returs > std::string 

  <br>

```java
    inline explicit operator util::string_view() const 
```
- 버퍼 내용을 util::string_view로 봄 
- Returns > utill::string_view 

  <br>

```java
    inline explicit operator util::bytes_view() const
```
- 버퍼 내용을 util::bytes_view()로 봄 
- Returns > util::bytes_view

  <br>

```java
    inline const uint8_t *data() const 
```
- 버퍼의 데이터에 대한 pointer를 반환 
- 버퍼는 CPU 버퍼여야 함 <span style="color:#00FFFF">(is cpu()가 true)</span>
- 그렇지 않으면 assertion이 발생하거나 null pointer가 반환될 수 있음 
- 장치에 관계없이 버퍼의 데이터 주소를 얻으려면 <span style="color:#00FFFF">address()</span>호출 

  <br>

```java
    inline uint8_t *mutable_data()
```
- 버퍼의 데이터에 대한 writable pointer를 반환 
- 버퍼는 변경 가능한 CPU 버퍼여야 함 <span style="color:#00FFFF">(is cpu() / is mutable()이 true)</span>
- 그렇지 않으면 assertion이 발생하거나 null pointer가 반환될 수 있음 
- 장치에 관계없이 버퍼의 변경 가능한 데이터 주소를 얻으려면 <span style="color:#00FFFF">mutable_address()</span>호출 

  <br>

```java
    inline uintptr_t address() const 
```
- 버퍼 데이터의 장치 주소를 반환 

  <br>

```java
    inline uintptr_t mutable_address() const 
```
- 버퍼의 데이터에 writable한 장치 주소를 반환 
- 버퍼는 변경 가능한 버퍼여야 함 <span style="color:#00FFFF">(is mutable()이 true)</span>
- 그렇지 않으면 assertion이 발생하거나 0이 반환될 수 있음 

  <br>

```java
    inline int64_t size() const 
```
- 버퍼의 크기를 바이트 단위로 반환 

  <br>

```java
    inline int64_t capacity() const
```
- 버퍼의 용량(할당된 바이트 수)을 반환 

  <br>

```java
    inline bool is_cpu() const
```
- 버퍼가 CPU에 직접 액세스 할 수 있는지 여부 
- 함수가 true를 반환하면, 버퍼의 data() pointer에서 직접 읽을 수 있음 
- 그렇지 않으면 View()나 Copy() 해야 함 

  <br> 

```java
    inline bool is_mutable() const
```
- 버퍼가 변경 가능한지 여부 
- 함수가 true를 반환하면, mutable_data()나 mutable_address()에 의해 반환된 pointer를 사용하여 버퍼 내용을 수정할 수 있음 

  <br>

##### Public Static Functions 

```java
    static std:: shared_ptr<Buffer> FromString(std::string data)
```
- std::string을 복사하지 않고 소유권을 갖는 불변 버퍼를 생성
- 매개변수 
  + data - [in] 소유할 문자열 
- Returns > 새로운 버퍼 인스턴스 

  <br>

```java
    template<typename T, typename SizeType = int64 t>
    static inline std::shared_ptr<Buffer> Wrap(const T *data, SizeType length)
```
- 복사하지 않고 일정 길이의 형식화된 메모리를 참조하는 버퍼를 생성
- 매개변수 
  + data - [in] C 배열로 입력된 메모리 
  + length - [in] 배열의 값의 수 
- Returns > 새로운 shared_ptr<Buffer>

  <br>

```java
    template<typename T>
    static inline std::shared_ptr<Buffer> Wrap(const std::vector<T> &data)
```
- 복사하지 않고 일정한 길이로 std::vector를 참조하는 버퍼 생성 
- 매개변수 
  + data - [in] 참조할 벡터. 벡터가 변경되면 버퍼가 무효화될 수 있음 
- Returns > 새로운 shared_ptr<Buffer>

  <br>

```java
    static Result<std::shared_ptr<io::RandomAccessFile>>
    GetReader(std::shared_ptr<Buffer>)
```
- 버퍼를 읽기 위한 RandomAccessFile을 가져옴 
- 반환된 파일 객체는 버퍼의 기본 메모리에서 읽음 

  <br>

```java
    static Result<std::shared_ptr<io::OutputStream>>
    GetWriter(std::shared_ptr<Buffer>)
```
- 버퍼에 쓰기 위한 OutputStream을 가져옴 
- 버퍼는 변경 가능해야 함. 반환된 스트림 객체는 버퍼의 기본 메모리에 씀 (크기는 조정되지 않음)

  <br>

```java
    static Result<std::shared_ptr<Buffer>> Copy(std::shared_ptr<Buffer> source,
    const std::shared_ptr<MemoryManager> &to)
```
- 버퍼를 복사 
- 버퍼 내용은 지정된 MemoryManager에 의해 할당된 새 버퍼에 복사됨 
- 장치간 복사를 지원 

  <br>

```java
    static Result<std::shared_ptr<Buffer>> View(std::shared_ptr<Buffer> source,
    const std::shared_ptr<MemoryManager> &to)
```
- View 버퍼 
- 내용을 명시적으로 복사하지 않고, 잠재적으로 다른 장치에서 볼 수 있는 해당 버퍼를 반영하는 버퍼를 반환 
- 기본 메커니즘은 커널 또는 장치 드라이버에 의해 구현되며 대상 장치의 메모리에 있는 버퍼 내용 부분의 lazy caching을 포함할 수 있음 
- 지정된 장치의 버퍼에 대해 non-copy view가 지원되지 않으면 nullptr이 반환됨 
- 일부 하위 수준의 작업 (ex.메모리 부족 상태) 이 실패하면 오류가 반환될 수 있음 

  <br>

```java
    static Result<std::shared_ptr<Buffer>> ViewOrCopy(std::shared_ptr<Buffer>source,
    const std::shared_ptr<MemoryManager> &to)
```
- 버퍼를 보거나 복사 
- 주어진 MemoryManager의 장치에서 버퍼 내용을 보려고 시도하지만, no-copy view가 지원되지 않으면 복사로 대체 

  <br>

#### <span style="color:#FF8C00">class arrow::MutableBuffer : public arrow::Buffer</span>
- 내용을 변경할 수 있는 버퍼 
- 데이터를 소유하거나 소유하지 않을 수 있음 
- *Arrow::cuda::CudaHostBuffer, arrow::ResizableBuffer* 에 의해 서브클래싱 

##### Public Static Functions 

```java
    template<typename T, typename SizeType = int64_t>
    static inline std::shared_ptr<Buffer> Wrap(T *data, SizeType length)
```
- 일정 길이의 형식화된 메모리를 참조하는 버퍼를 생성 
- 매개변수 
  + data - [in] C 배열로 입력된 메모리 
  + length - [in] 배열의 값 수 
- Returns > 새로운 shared_ptr<Buffer>

  <br>

#### <span style="color:#FF8C00">class arrow::ResizableBuffer : public arrow::MutableBuffer</span>
- 크기를 조정할 수 있는 변경 가능한 버퍼 

##### Public Functions 

```java
    virtual Status Resize (const int64_t new_size, bool shrink_to_fit) = 0
```
- 버퍼 reported size를 표시된 크기로 변경하고, 필요한 경우 메모리를 할당 
- 이렇게 하면 버퍼의 용량이 Layout.md에 정의된 대로 64바이트의 배수가 됨 
- Arrow 레이아웃 사양을 준수하려면 후에 ZeroPadding을 사용하는 것이 좋음 
- 매개변수 
  + new_size - 버퍼의 새로운 크기 
  + Shrink_to_fit - 새로운 크기 < 현재 크기인 경우, 용량 축소 여부 

  <br>

```java
    virtual Status Reserve(const int64_t new_capacity) = 0
```
- 버퍼에 표시된 용량에 맞게 할당된 충분한 메모리가 있는지 확인 (Layout.md의 64바이트 패딩 요구사항을 충족하는지)
- 버퍼의 reported size를 변경하지 않으면 패딩을 0으로 만들지 않음 

  <br>

### Slicing
- 기본 데이터의 일부 연속적인 하위 집합을 참조하는 버퍼를 얻기 위해, 버퍼의 zero-copy slice를 만드는 것이 가능함 
- <span style="color:#00FFFF">arrow::SliceBuffer()</span>, <span style="color:#00FFFF">arrow::SliceMutableBuffer()</span> 함수를 호출하여 수행



***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/docs/cpp/memory.html>
- <https://arrow.apache.org/docs/cpp/api/memory.html>