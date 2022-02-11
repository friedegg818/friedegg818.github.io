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

### <span style="color:#A9A9A9">관련 API</span> 

#### <span style="color:#FF8C00">class arrow::Buffer</span>
- 특정 크기의 연속적인 메모리에 대한 pointer를 포함하는 개체 
- size / capacity 라는 두가지 관련 개념 
> - size: 유효한 데이터를 가질 수 있는 바이트 수 
> - capacity: 총 버퍼에 할당된 바이트 수 
- 버퍼 기본 클래스는 메모리를 소유하지 않지만, 종종 subclass가 소유 
- Size <= Cpacity는 항상 참 
- *arrow::cuda::CudaBuffer, arrow::MutableBuffer, arrow::py::NumPyBuffer, arrow::py::PyBuffer, arrow::py::PyForeignBuffer* 에 의해 서브클래싱 

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

## Slicing
- 기본 데이터의 일부 연속적인 하위 집합을 참조하는 버퍼를 얻기 위해, 버퍼의 zero-copy slice를 만드는 것이 가능함 
- <span style="color:#00FFFF">arrow::SliceBuffer()</span>, <span style="color:#00FFFF">arrow::SliceMutableBuffer()</span> 함수를 호출하여 수행

### <span style="color:#A9A9A9">관련 API</span>

```java
    static inline std::shared_ptr<Buffer>
    SliceBuffer(const std::shared_ptr<Buffer> &buffer,
    const int64_t offset, const int64_t length)
```
- 주어진 offset과 length로 버퍼에 view를 생성 
- 이 함수는 실패할 수 없으며 오류를 확인하지 않음 (debug build 제외)

<br>

```java
    static inline std::shared_ptr<Buffer>
    SliceBuffer(const std::shared_ptr<Buffer> &buffer, const int64_t offset)
```
- 버퍼의 끝까지 주어진 offset에서 버퍼의 view를 생성 
- 이 함수는 실패할 수 없으며 오류를 확인하지 않음 (debug build 제외)

<br>

```java
    Result<std::shared_ptr<Buffer>> 
    SliceBufferSafe(const std::shared_ptr<Buffer> &buffer, int64_t offset)
```
- SliceBuffer의 입력 확인 버전 
- 요청된 슬라이스가 범위를 벗어나면 Invalid Status 반환 

<br>

```java
    Result<std::shared_ptr<Buffer>>
    SliceBufferSafe(const std::shared_ptr<Buffer> &buffer, int64_t offset, int64_t length)
```
- SliceBuffer의 입력 확인 버전 
- 요청된 슬라이스가 범위를 벗어나면 Invalid Status 반환 
- Slice Buffer와 달리 length는 사용 가능한 버퍼 크기로 고정되지 않음 

<br>

```java
    std::shared_ptr<Buffer> 
    SliceMutableBuffer(const std::shared_ptr<Buffer> &buffer,
    const int64_t offset, const int64_t length)
```
- SliceBuffer와 비슷하지만, 가변 버퍼 슬라이스를 구성 
- 상위 버퍼가 변경가능하지 않으면 동작이 정의되지 않음 (debug build에서 중단될 수 있음 )

<br>

```java
    static inline std::shared_ptr<Buffer> 
    SliceMutableBuffer(const std::shared_ptr<Buffer> &buffer, const int64_t offset)
```
- SliceBuffer와 비슷하지만, 가변 버퍼 슬라이스를 구성 
- 상위 버퍼가 변경가능하지 않으면 동작이 정의되지 않음 (debug build에서 중단될 수 있음)

<br>

```java
    Result<std::shared_ptr<Buffer> 
    SliceMutableBufferSafe(const std::shared_ptr<Buffer> &buffer, int64_t offset)
```
- SliceMutableBuffer의 입력 확인 버전 
- 요청된 슬라이스가 범위를 벗어나면 Invalid Status 반환 

<br>

```java
    Result<std::shared_ptr<Buffer>>
    SliceMutableBufferSafe(const std::shared_ptr<Buffer>&buffer, int64_t offset, int64_t length)
```
- Slice Mutable Buffer의 입력 확인 버전 
- 요청된 슬라이스가 범위를 벗어나면 Invalid Status 반환 
- SliceBuffer와 달리 length는 사용 가능한 버퍼 사이즈로 고정되지 않음 

<br>

## 버퍼 할당 
-  <span style="color:#00FFFF">arrow::AllocateBuffer()</span>,  <span style="color:#00FFFF">arrow::AllocateResizableBuffer()</span> 오버로드 중 하나를 호출하여 버퍼를 직접 할당할 수 있음 

```java
    arrow::Result<std::unique_ptr<Buffer>> maybe_buffer = arrow::AllocateBuffer(4096); 
    if (!maybe_buffer.ok()) {
        // ... handle allocation error
    }

    std::shared_ptr<arrow::Buffer> buffer = *std::move(maybe_buffer);
    unit8_t* buffer_data = buffer->mutable_data();
    memcpy(buffer_data, "hello world", 11);
```
- 이 방법으로 버퍼를 할당하면 **Arrow 메모리 사양에서 권장하는대로 버퍼가 64바이트로 정렬되고 채워짐 

>  Arrow 메모리 권장 사항 - Buffer alignment & padding
- 구현은 정렬된 주소 (8 or 64바이트의 배수)에 메모리를 할당하고 8 or 64바이트의 배수의 길이로 채우는 것이 좋음 
- 프로세스간 통신을 위해 Arrow 데이터를 직렬화할 때 이러한 정렬 및 패딩 요구사항이 적용됨
- 가능하면 64바이트 정렬 및 패딩을 사용하는 것이 좋음 

### <span style="color:#A9A9A9">관련 API</span>
- 특정 메모리 풀에서 버퍼를 할당하는 함수들 

```java
    Result<std::unique_ptr<Buffer>> 
    AllocateBuffer(const int64_t size, MemoryPool *pool = NULLPTR)
```
- 메모리 풀에서 고정된 크기의 가변 버퍼를 할당하고 패딩을 0으로 만듦 
- 매개변수 
  + size - [in] 할당할 버퍼의 크기 
  + pool - [in] 메모리 풀 

  <br>

```java
    Result<std::unique_ptr<ResizableBuffer>> 
    AllocateResizableBuffer(const int64_t size, MemoryPool *pool = NULLPTR)
```
- 메모리 풀에서 크기 조정 가능한 버퍼를 할당하고 패딩을 0으로 만듦 
- 매개변수 
  + size - [in] 할당할 버퍼의 크기 
  + pool - [in] 메모리 풀 

  <br>

```java
    Result<std::shared_ptr<Buffer>> 
    AllocateBitmap(int64_t length, MemoryPool *pool = NULLPTR)
```
- 값이 보장되지 않는 메모리 풀에서 비트맵 버퍼를 할당 
- 매개변수 
  + length - [in] 할당할 비트맵의 비트 크기 
  + pool - [in] 메모리를 할당할 메모리 풀 

  <br>

```java
    Result<std::shared_ptr<Buffer>> 
    AllocateEmptyBitmap(int64_t length, MemoryPool *pool = NULLPTR)
```
- 메모리 풀에서 0으로 초기화된 비트맵 버퍼를 할당
- 매개변수 
  + length - [in] 할당할 비트맵의 비트 크기 
  + pool - [in] 메모리를 할당할 메모리 풀 

  <br>

```java
    Result<std::shared_ptr<Buffer>>
    ConcatenateBuffers(const BufferVector &buffers, MemoryPool *pool = NULLPTR)
```
- 여러 버퍼를 단일 버퍼로 연결 
- 매개변수 
  + buffers - [in] 연결될 버퍼들 
  + pool 0 [in] 새로운 버퍼를 할당할 메모리 풀 

  <br>

## 버퍼 생성 
 - <span style="color:#00FFFF">arrow::BufferBuilder</span> API를 사용하여 점진적으로 버퍼를 할당하고 빌드할 수 있음 

 ```java
    BufferBuilder builder;
    builder.Resize(11);     // reserve enough space for 11 bytes 
    builder.Append("hello ", 6);
    builder.Append("world", 5);

    auto maybe_buffer = builder.Finish();
    if (!maybe_buffer.ok()) {
        // ... handle buffer allocation error
    }
    std::shared_ptr<arrow::Buffer> buffer = *maybe_buffer;
 ```
- 버퍼가 지정된 고정 너비 유형의 값을 포함하도록 되어 있는 경우 (ex.List 배열의 32-bit offset), <span style="color:#00FFFF">arrow::TypeBufferBuilder</span> API 템플릿을 사용하는 것이 더 편리할 수 있음 

```java
    TypedBufferBuilder<int32_t> builder;
    builder.Reserve(2);     // reserve enough space for two int32_t values
    bulider.Append(0x12345678);
    builder.Append(-0x765643210);

    auto maybe_buffer = builder.Finish();
    if (!maybe_buffer.ok()) {
        // ... handle buffer allocation error
    }
    std::shared_ptr<arrow::Buffer> buffer = *maybe_buffer'
```

### <span style="color:#A9A9A9">관련 API</span>

#### <span style="color:#FF8C00">class arrow::BufferBuilder</span>
- In-memory 데이터의 연속적인 chunk를 점진적으로 구축하기 위한 클래스 

##### Public Functions 

```java
    inline explicit 
    BufferBuilder(std::shared_ptr<ResizableBuffer> buffer, MemoryPool *pool = default_memory_pool())
```
- Finish/Rest이 호출될 때까지 제공된 버퍼를 사용하여 시작하는 새로운 builder를 생성 
- 버퍼의 크기는 조정되지 않음 

<br>

```java
    inline Status Resize(const int64_t new_capacity, bool shrink_to_fit = true) 
```
- 버퍼 크기를 64바이트의 가장 가까운 배수로 조정 
- 매개변수 
  + new_capacity - 빌더의 새 용량. 패딩을 위해 64바이트의 배수로 반올림 
  + shrink_to_fit - 새 용량이 기존 용량보다 작으면 내부 버퍼를 재할당. 빌더를 축소할 때 재할당 하지 않으려면 false로 설정 

  <br>

```java
    inline Status Reserve(const int64_t additional bytes)
```
- 빌더가 할당할 필요 없이 추가 바이트 수를 수용할 수 있는지 확인 
- 매개변수
  + additional_bytes - [in] 공간을 만들 추가 바이트 수 
- Returns > Status 

  <br>

```java
    inline Status Append(const void *data, const int64_t length)
```
- 버퍼에 주어진 데이터를 추가 
- 필요한 경우 버퍼가 자동으로 확장됨 

  <br>

```java
    inline Status Append(const int64_t num_copies, uint8_t value)
```
- 버퍼의 값에 copy를 추가 
- 필요한 경우 버퍼가 자동으로 확장됨 

  <br>

```java
    inline Status Finish(std::shared_ptr<Buffer> *out, bool shrink_to_fit = true)
```
- 빌더의 결과를 버퍼의 객체로 반환 
- 빌더가 재설정되고, 나중에 다시 사용할 수 있음 
- 매개변수 
  + out - [out] 최종 버퍼 객체 
  + shrink_to_fit - 버퍼 크기가 용량보다 작은 경우, 메모리에 좀 더 잘 맞게끔 재할당. 재할당을 방지하려면 false로 설정 (더 많은 메모리 소비)
- Returns > Status 

  <br>

```java
    inline Result<std::shared_ptr<Buffer>> 
    FinishWithLength(int64_t final_length, bool shrink_to_fit = true)
```
- Finish와 비슷하지만, 최종 버퍼 크기를 override (재정의)
- Append method를 호출하지 않고 빌더 메모리에 직접 데이터를 쓴 후에 유용함         
  (주로 메모리 할당에 Bufferbuilder를 사용할 때)

  <br>

```java
    inline void Rewind(int64_t position)
```
- 빌더 내용을 수정하지 않고 크기를 더 작은 값으로 설정 
- 재사용 가능한 BufferBuilder 클래스에 사용 
- 매개변수 
  + position - [in] 음수가 아니어야하고, 현재 length() 이하여야 함 

  <br>

##### Public Static Functions 

```java
    static inline int64_t GrowByFactor(int64_t current_capacity, int64_t new_capacity)
```
- 원하는 growth factor만큼 확장된 용량을 반환 

  <br>

## Memory pool 
- Arrow C++ API를 사용하여 버퍼를 할당할 때, 버퍼의 기본 메모리는 <span style="color:#00FFFF">arrow::MemoryPool</span> 인스턴스에 의해 할당됨 
- 일반적으로 이것은 프로세스 전반의 메모리 풀이지만, 여러가지 Arrow API를 사용하면 내부 할당을 위해 다른 Memory Pool 인스턴스를 전달할 수 있음 
- 메모리 풀은 array 버퍼와 같은 수명이 긴 대용량 데이터에 사용됨 
- 작은 C++ 개체 및 임시 작업공간과 같은 기타 데이터는 보통 일반 C++ 할당자를 거침 

### 기본 Memory pool 
- 기본 메모리 풀은 Arrow C++가 컴파일된 방식에 따라 다름 

> 선택 알고리즘 
 - if enabled at compile time, a **jemalloc** heap;
 - otherwise, if enabled at compile time, a **mimalloc** heap;
 - otherwise, the C library **malloc** heap.

### 기본 Memory pool Overriding 
- <span style="color:#00FFFF">ARROW_DEFAULT_MEMORY_POOL</span> 환경 변수를 jemalloc, mimalloc, system 중 하나로 선택하여 위의 선택 알고리즘을 재정의할 수 있음 
- 이 변수는 Arrow C++가 메모리에 로드될 때 한 번 검사됨 (ex.Arrow C++ DLL이 로드될 때)

### <span style="color:#A9A9A9">관련 API</span>

```java
    MemoryPool *arrow::default_memory_pool()
```
- 프로세스 전체의 기본적인 메모리 풀 반환 

<br>

```java
    Status arrow::jemalloc_memory_pool(MemoryPool **out)
```
- jemalloc을 기반으로 하는 프로세스 전체 메모리 풀 반환 
- jemalloc을 사용할 수 없는 경우 NotImplemented 반환 

<br>

```java
    Status arrow::mimalloc_memory_pool(MemoryPool **out)
```
- mimalloc을 기반으로 하는 프로세스 전체 메모리 풀 반환 
- mimalloc을 사용할 수 없는 경우 NotImplemented 반환 

<br>

```java
    MemoryPool *arrow::system_memory_pool()
```
- 시스템 할당자를 기반으로 하는 프로세스 전체 메모리 풀 반환 

#### <span style="color:#FF8C00">class arrow::MemoryPool</span>
- CPU 메모리 할당을 위한 기본 클래스 
- 할당된 바이트 수를 추적하는 것 외에도, 할당자는 요구되는 64바이트 정렬을 처리해야 함 
-  <span style="color:#00FFFF"> *arrow::dataset::jni::ReservationListenableMemoryPool, arrow::LoggingMemoryPool, arrow::ProxyMemoryPool, arrow::stl::STLMemoryPool< Allocator >* </span> 에 의해 서브클래싱

##### Public Functions 

```java
    virtual Status Allocate(int64_t size, uint8_t **out) = 0
```
- 최소 크기 바이트의 새로운 메모리 영역 할당
- 할당된 영역은 64바이트로 정렬됨 

<br>

```java
    virtual Status Reallocate(int64_t old_size, int64_t new_size, uint8_t **ptr) = 0
```
- 이미 할당된 메모리 섹션의 크기 조정 
- 기본적으로 플랫폼 대부분의 기본 할당자는 정렬된 재할당을 지원하지 않으므로, 이 기능에는 기본 데이터의 복사본이 포함될 수 있음 

<br>

```java
    virtual void Free(uint8_t *buffer, int64_t size) = 0
```
- 할당된 영역을 해제 
- 매개변수 
  - buffer - 할당된 메모리 영역의 시작에 대한 Pointer 
  - size - 버퍼에 있는 할당된 크기. 할당자 구현은 할당된 바이트 양을 추적하고, 백엔드에서 지원하는 경우 더 빠른 할당 해제를 위해 사용할 수 있음 

  <br>

```java
    inline virtual void ReleaseUnused()
```
- 사용하지 않은 메모리를 OS에 반환 
- 사용하지 않은 메모리를 보유하는 할당자에만 적용 

<br>

```java
    virtual int64_t bytes_allocated() const = 0
```
- 이 할당자를 통해 할당되었지만 아직 해제되지 않은 바이트 수 

<br>

```java
    virtual int64_t max_memory() const 
```
- 이 메모리 풀에서 최대 메모리 할당을 반환 
- Returns > 할당된 최대 바이트 수. 알 수 없거나 구현되지 않은 경우 -1 반환 

<br>

```java
    virtual std::string backend_name() const = 0
```
- 이 메모리 풀에서 사용되는 백엔드의 이름 

<br>

##### Public Static Functions 

```java
    static std::unique_ptr<MemoryPool> CreateDefault()
```
- 기본 메모리 풀의 새로운 인스턴스를 만듦 

<br>

#### <span style="color:#FF8C00">class arrow::LoggingMemoryPool : public arrow::MemoryPool</span>

##### Public Functions 

```java
    virtual Status Allocate(int64_t size, uint8_t **out) override 
```
- 최소 크기의 바이트의 새로운 메모리 영역을 할당 
- 할당된 영역은 64 바이트로 정렬됨 

<br>

```java
    virtual Status Reallocate(int64_t old_size, int64_t new_size, uint8_t **ptr) override 
```
- 이미 할당된 메모리 섹션의 크기 조정 
- 기본적으로 플랫폼 대부분의 기본 할당자는 정렬된 재할당을 지원하지 않으므로, 이 기능에는 기본 데이터의 복사본이 포함될 수 있음 

<br>

```java
    virtual void Free(uint8_t **buffer, int64_t size) override
```
- 할당된 영역을 해제함 
- 매개변수 
  + buffer - 할당된 메모리 영역의 시작에 대한 Pointer 
  + size - 버퍼에 있는 할당된 크기. 할당자 구현은 할당된 바이트의 양을 추적하고, 백엔드에서 지원하는 경우 더 빠른 할당해제를 위해 사용할 수 있음 

  <br>

```java
    virtual int64_t bytes_allocated() const override 
```
- 이 할당자를 통해 할당되었지만 아직 해제되지 않은 바이트 수 

<br>

```java
   virtual int64_t max_memory() const override 
```
- 이 메모리 풀에서 최대 메모리 할당을 반환 
- Returns > 할당된 최대 바이트. 알 수 없거나 구현되지 않은 경우 -1 반환 

<br>

```java
    virtual std::string backend_name() const override
```
- 이 메모리 풀에서 사용되는 backend의 이름 

<br>

#### <span style="color:#FF8C00">class arrow::ProxyMemoryPool : public arrow::MemoryPool</span>
- 메모리 할당을 위한 파생 클래스 
- 직접 호출을 통해 할당된 바이트 수와 최대 메모리를 추적 
- 실제 할당은 MemoryPool class에 위임됨 

##### Public Functions 

```java
    virtual Status Allocate(int64_t size, uint8_t **out) override 
```
- 최소 크기의 바이트의 새로운 메모리 영역을 할당 
- 할당된 영역은 64 바이트로 정렬됨 

<br>

```java
    virtual Status Reallocate(int64_t old_size, int64_t new_size, uint8_t **ptr) override 
```
- 이미 할당된 메모리 섹션의 크기 조정 
- 기본적으로 플랫폼 대부분의 기본 할당자는 정렬된 재할당을 지원하지 않으므로, 이 기능에는 기본 데이터의 복사본이 포함될 수 있음 

<br>

```java
    virtual void Free(uint8_t **buffer, int64_t size) override
```
- 할당된 영역을 해제함 
- 매개변수 
  + buffer - 할당된 메모리 영역의 시작에 대한 Pointer 
  + size - 버퍼에 있는 할당된 크기. 할당자 구현은 할당된 바이트의 양을 추적하고, 백엔드에서 지원하는 경우 더 빠른 할당해제를 위해 사용할 수 있음 

  <br>

```java
    virtual int64_t bytes_allocated() const override 
```
- 이 할당자를 통해 할당되었지만 아직 해제되지 않은 바이트 수 

<br>

```java
   virtual int64_t max_memory() const override 
```
- 이 메모리 풀에서 최대 메모리 할당을 반환 
- Returns > 할당된 최대 바이트. 알 수 없거나 구현되지 않은 경우 -1 반환 

<br>

```java
    virtual std::string backend_name() const override
```
- 이 메모리 풀에서 사용되는 backend의 이름 

<br>

```java
    std::vector<std::string> arrow::SupportedMemoryBackendNames()
```
- 이 Arrow build에서 지원하는 backend의 이름 반환

<br>

## STL 통합
- Arrow memory pool을 사용하여 STL 컨테이너의 데이터를 할당하려면 <span style="color:#00FFFF">arrow::stl::allocator</span> wrapper를 사용하면 됨 
- 반대로 <span style="color:#00FFFF">arrow::stl::STLMemoryPool</span> 클래스를 사용하면 STL 할당자를 사용하여 Arrow 메모리를 할당할 수 있음 
- 그러나, STL 할당자는 resizing 작업이 없어 성능이 떨어질 수 있음 

### <span style="color:#A9A9A9">관련 API</span>

#### <span style="color:#FF8C00">template<class T> class arrow::stl::allocator</span>
- Arrow MemoryPool에 할당을 위임하는 STL 할당자 

##### Public Functions 

```java
    inline allocator() noexcept
```
- 기본 메모리풀에서 할당자를 구성

<br>

```java
    inline explicit allocator(MemoryPool *pool) noexcept
```
- 지정된 메모리풀에서 할당자를 구성 

<br>

#### <span style="color:#FF8C00">template<typename Allocator = std::alocator<uint8_t>> class arrow::stl::STLMemoryPool : public arrow::MemoryPool</span>
- STL 할당자에게 할당을 위임하는 메모리풀 구현 
- STL 할당자는 크기 조정 작업을 제공하지 않으므로, 버퍼 크기 조정은 전체 재할당 및 복사를 수행함

##### Public Functions 

```java
    inline explicit STLMemoryPool (const Allocator &alloc)
```
- 지정된 할당자에서 메모리 풀을 구성 

<br>

```java
    inline virtual Status Allocate(int64_t size, uint8_t **out) override
```
- 최소 크기의 바이트의 새 메모리 영역을 할당 
- 할당된 영역은 64 바이트로 정렬됨 

<br>

```java
   inline virtual Status Reallocate(int64_t old_size, int64_t new_size, uint8_t *ptr) override 
```
- 이미 할당된 메모리의 섹션의 크기 조정 
- 기본적으로 플랫폼 대부분의 기본 할당자는 정렬된 재할당을 지원하지 않음로, 이 기능에는 기본 데이터의 복사본이 포함될 수 있음 

<br>

```java
    virtual void Free(uint8_t *buffer, int64_t size) override
```
- 할당된 영역을 해제함 
- 매개변수 
  + buffer - 할당된 메모리 영역의 시작에 대한 Pointer 
  + size - 버퍼에 있는 할당된 크기. 할당자 구현은 할당된 바이트의 양을 추적하고 백엔드에서 지원하는 경우, 더 빠른 할당 해제를 위해 사용할 수 있음 

<br>

```java
    virtual int64_t bytes_allocated() const override
```
- 이 할당자를 통해 할당되었지만 아직 해제되지 않은 바이트 수 

<br>

```java
    virtual int64_t max_memory() const override
```
- 이 메모리 풀에서 최대 메모리 할당을 반환 
- Returns > 할당된 최대 바이트. 알 수 없거나 구현되지 않은 경우 -1 반환 

<br>

```java
    virtual std::string backend_name() const override
```
- 이 메모리 풀에서 사용되는 bacend의 이름 

<br>

## Devices 
- 많은 Arrow 애플리케이션은 호스트 (CPU) 메모리에만 액세스 
- 그러나, 어떤 경우에는 호스트 메모리뿐만 아니라 on-device 메모리 (ex.GPU의 on-board 메모리)를 처리하는 것이 바람직 
- Arrow는 <span style="color:#00FFFF">arrow::Device abstraction</span>을 사용하는 CPU 및 기타 장치를 나타냄 
- 연관된 클래스인 <span style="color:#00FFFF">arrow::MemoryManager</span>는 지정된 장치에 할당하는 방법을 구체화 함 
- 각 장치에는 기본 메모리 관리자가 있지만, 추가 인스턴스를 구성할 수 있음 (ex.사용자 지정 arrow::MemoryPool CPU wrapping)
- 주어진 장치에 메모리를 할당하는 방법을 지정하는 <span style="color:#00FFFF">arrow::MemoryManager</span> 인스턴스 

### Device-Agnostic 프로그래밍 
- Third-party code에서 버퍼를 수신하는 경우, is_cpu() 메서드를 호출하여 CPU-readable 여부를 쿼리할 수 있음 
- <span style="color:#00FFFF">arrow::Buffer::View()</span>나 <span style="color:#00FFFF">arrow::Buffer::ViewOrCopy()</span>를 호출하여 일반적인 방식으로 지정된 장치의 버퍼를 볼수 있음 
- 소스 및 대상 장치가 동일하지 않을 경우에는 작동하지 않음 
- 그렇지 않으면, 장치 종속 메커니즘이 버퍼 내용에 대한 액세스를 제공하는 대상 장치에 대한 메모리 주소를 구성하려고 시도함 → 버퍼 내용을 읽을 때, 실제 장치 간 전송이 지연될 수 있음 
- 마찬가지로, CPU가 읽을 수 있는 버퍼를 가정하지 않고 버퍼에서 I/O를 수행하려면 <span style="color:#00FFFF">arrow::Buffer::GetReader()</span> 및 <span style="color:#00FFFF">arrow::Buffer::GetWriter()</span>를 호출할 수 있음 
- 예를 들어, CPU view 또는 임의의 버퍼의 copy를 얻으려면 다음과 같이 하면 됨 

```java
    std::shared_ptr<arrow::Buffer> arbitrary_buffer = ...; 
    std::shared_ptr<arrow::Buffer> cpu_buffer = arrow::Buffer::ViewOrCopy(
        arbitrary_buffer, arrow::default_cpu_memory_manager()
    );
```

<br>

### <span style="color:#A9A9A9">관련 API</span>

#### <span style="color:#FF8C00">class arrow::Device : public astd::enable_shared_from_this<Device>, public arrow::util::EqualityComparable<Device></span>
- 하드웨어 장치에 대한 추상 인터페이스 
- 이 개체는 일부 메모리 공간에 액세스할 수 있는 장치를 나타냄 
- 버퍼 또는 raw memory 주소를 처리할 때, raw memory 주소를 해석해야하는 context를 결정할 수 있음 
- <span style="color:#00FFFF">*arrow::CPUDevice, arrow::cuda::CudaDevice*</span>로 서브클래싱 

##### Public Functions

```java
    virtual const char *type_name() const 0
```
- 장치 유형의 약어 
- 반환된 값은 각 장치 클래스에 따라 다르지만, 지정된 클래스의 모든 인스턴스에 대해 동일
- RTTI의 대체로 사용 가능 

<br>

```java
    virtual std::string ToString() const = 0
```
- 사람이 읽을 수 있는 장치 설명 
- 반환된 값은 필요한 경우 서로 다른 인스턴스를 구별할 수 있을 만큼 상세해야 함 

<br>

```java
    virtual bool Equals(const Device&) const = 0
```
- 인스턴스가 다른 인스턴스와 동일한 장치를 가리키는지 여부 

<br>

```java
    inline bool is_cpu() const
```
- 장치가 주 CPU 장치인지 여부 
- 메모리 주소가 CPU에 액세스할 수 있는지 여부를 결정할 때에 매우 유용 

<br>

```java
    virtual std::shared_ptr<MemoryManager> default_memory_manager() = 0
```
- 장치에 연결된 MemoryManager 인스턴스를 반환 
- 반환된 인스턴스는 이 장치 유형의 MemoryManager 구현에 대한 기본 매개변수를 사용 
- 일부 장치에서는 기본값이 아닌 매개변수를 사용하여 MemoryManager 인스턴스를 구성할 수도 있음 

<br>

#### <span style="color:#FF8C00">class arrow::CPUDevice : public arrow::Device</span>

##### Public Functions 

```java
    virtual const char *type_name() const override
```
- 장치 유형의 약어 
- 반환된 값은 각 장치 클래스에 따라 다르지만, 지정된 클래스의 모든 인스턴스에 대해 동일 
- RTTI의 대체로 사용 가능 

<br>

```java
    virtual std::string ToString() const override
```
- 사람이 읽을 수 있는 장치 설명 
- 반환된 값은 필요한 경우 서로 다른 인스턴스를 구별할 수 있을만큼 상세해야 함 

<br>

```java
    virtual bool Equals(const Device&) const override
```
- 인스턴스가 다른 인스턴스와 동일한 장치를 가리키는지 여부 

<br>

```java
    virtual std::shared_ptr<MemoryManager> default_memory_manager() override
```
- 장치에 연결된 MemoryManager 인스턴스를 반환 
- 반환된 인스턴스는 이 장치 유형의 MemoryManager 구현에 대한 기본 매개변수를 사용 
- 일부 장치에서는 기본값이 아닌 매개변수를 사용하여 MemoryManger 인스턴스를 구성할 수도 있음 

<br>

##### Public Static Function 

```java
    satic std::shard_ptr<Device> Instance()
```
- 전역 CPUDevice 인스턴스 반환 

<br>

```java
    static std::shared_ptr<MemoryManager> memory_manager(MemoryPool *pool)
```
- MemoryManager 생성 
- 반환된 MemoryManager는 할당을 위해 지정된 MemoryPool 사용 

<br>

```java
    std::shared_ptr<MemoryManager> arrow::default_cpu_memory_manager()
```
- 기본 CPU MemoryManager 인스턴스 반환 
- 반환된 싱글톤 인스턴스는 기본 MemoryPool 사용 
- CPUDevice::Instance() -> default_memory_manager(0)의 축약 (faster spelling)


***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/docs/cpp/memory.html>
- <https://arrow.apache.org/docs/cpp/api/memory.html>