---

title: "[Apache Arrow] 09.Arrow C++ Library-3" 

excerpt: Arrays / Data Types / Tabular Data

categories: 
    - Apache Arrow
tags:
    - [Apache Arrow, C++ Library, Arrays, Data Types, Tabular Data]

toc: true
toc_sticky: true

date: 2022-02-23
last_modified_At: 2022-03-02

---

# Arrays 
- <span style="color:	#00FFFF">arrow::Array</span> → Arrow의 central type 
- Array(배열)는 모두 같은 유형을 가진 값의 시퀀스
- 내부적으로 이러한 값은 배열의 데이터 유형에 따라 수와 의미가 달라지는 하나 이상의 버퍼로 표현됨 (Arrow data layout specification에 명시)
  + 이러한 버퍼는 값 데이터 자체와, 배열 항목이 null 값임을 나타내는 선택적 비트맵 버퍼로 구성 
  + 배열에 null 값이 없을 경우 비트맵 버퍼는 완전히 생략 가능 
- 배열의 개별 값에 액세스하는데 도움이 되는 각 유형에 대한 <span style="color:	#00FFFF">arrow::Array</span>의 구체적인 하위 클래스 존재 


## Building an Array 

### Availabe strategies  
- Arrow 객체는 변경할 수 없기 때문에, <span style="color:	#00FFFF">std::vector</span>처럼 직접 채울 수는 없음 
- 그 대신, 다음과 같은 방법을 사용 
  + 데이터가 이미 알맞은 레이아웃으로 메모리에 존재하는 경우          
   : 해당 메모리를 <span style="color:	#00FFFF">arrow::Buffer</span> 인스턴스 내부에 래핑한 후 배열을 설명하는 <span style="color:	#00FFFF">arrow::ArrowData</span>를 구성 
  + 그렇지 않으면,          
  : <span style="color:	#00FFFF">arrow::ArrayBuilder</span> 기본 클래스와 그것의 구체적인 하위 클래스가 배열 데이터를 점진적으로 구축하는데 도움이 됨 (Arrow 형식의 세부사항들을 직접 처리하지 않고도)

### ArrayBuilder 및 하위 클래스 사용 
- Int64 Arrow 배열을 만들기 위해, <span style="color:	#00FFFF">arrow::Int64Builder</span> 클래스 사용 
- 예시 :: 값 4를 보유해야하는 요소가 null인 1~8 사이의 배열 생성 

```java
 arrow::Int64Builder builder;
 builder.Append(1);
 builder.Append(2);
 builder.Append(3);
 builder.AppendNull();
 builder.Append(5);
 builder.Append(6);
 builder.Append(7);
 builder.Append(8);

 auto maybe_array = builder.Finish();
 if (!maybe_array.ok()) {
    // ... do something on array building failure
  }
  std::shared_ptr<arrow::Array> array = *maybe_array;
```
- 해당 값에 액세스하려는 경우, 구체적인 <span style="color:	#00FFFF">arrow::Int64Array</span> 하위 클래스로 캐스팅 할 수 있는 result Array는 두 개의 <span style="color:	#00FFFF">arrow::Buffers</span>로 구성됨 
  + 첫 번째 버퍼               
   : 1|1|1|1|0|1|1|1 비트가 있는 단일 바이트로 구성된 null bitmap 보유                     
     least-significant bit(LSB) numbering을 사용하므로 배열의 4번째 항목이 null임을 나타냄 
  + 두 번째 버퍼         
  : 단순하게 위의 모든 값을 포함하는 int64_t 배열                      
    네 번째 항목이 null이므로 버퍼의 해당 위치에 있는 값은 정의되지 않음 

- 구체적인 배열의 내용에 액세스하는 방법 

```java
// Cast the Array to its actual type to access its data
auto int64_array = std::static_pointer_cast<arrow::Int64Array>(array);

// Get the pointer to the null bitmap
const uint8_t* null_bitmap = int64_array->null_bitmap_data();

// Get the pointer to the actual data
const int64_t* data = int64_array->raw_values();

// Alternatively, given an array index, query its null bit and value directly
int64_t index = 2;
if (!int64_array->IsNull(index)) {
   int64_t value = int64_array->Value(index);
}	
```

## Performance
- 최고의 성능을 얻으려면 구체적인 <span style="color:	#00FFFF">arrow::ArrayBuilder</span> 하위 클래스에서 bulk 추가 메서드 (보통 <span style="color:#FF8C00">AppendValues</span>) 를 사용하는 것이 좋음 
- 요소의 수를 미리 알고 있다면, Resize()나 Reserve() 메소드를 호출하여 작업 영역의 크기를 미리 조정하는 것도 권장
- 예시 :: 값 4를 보유해야하는 요소가 null인 1~8 사이의 배열 생성 (위의 API 활용)

```java
arrow::Int64Builder builder;
 // Make place for 8 values in total
 builder.Reserve(8);
 // Bulk append the given values (with a null in 4th place as indicated by the
 // validity vector)
 std::vector<bool> validity = {true, true, true, false, true, true, true, true};
 std::vector<int64_t> values = {1, 2, 3, 0, 5, 6, 7, 8};
 builder.AppendValues(values, validity);

 auto maybe_array = builder.Finish();
```

- 값을 하나씩 추가해야 하는 경우, 일부 구체적인 빌더 하위 클래스에는 <span style="color:#FF8C00">"Unsafe"</span>로 표시된 메서드가 존재함
- 이는 작업 영역의 크기가 올바르게 사전 설정되었다고 가정하는 대신, 더 높은 성능을 제공함 

```java
arrow::Int64Builder builder;
 // Make place for 8 values in total
 builder.Reserve(8);
 builder.UnsafeAppend(1);
 builder.UnsafeAppend(2);
 builder.UnsafeAppend(3);
 builder.UnsafeAppendNull();
 builder.UnsafeAppend(5);
 builder.UnsafeAppend(6);
 builder.UnsafeAppend(7);
 builder.UnsafeAppend(8);

 auto maybe_array = builder.Finish();
```

## 크기 제한 및 권장사항 
- 일부 배열 유형은 구조적으로 32비트 크기로 제한됨 
  +  List arrays (최대 2^31개의 요소를 포함할 수 있음)
  +  String arrays 
  +  Binary arrays (최대 2GB의 이진 데이터를 포함할 수 있음)
- 일부 다른 배열 유형은 C++ 구현에서 최대 2^63개의 요소를 보유할 수 있지만, 다른 Arrow 구현도 해당 배열 유형에 대해 32비트 크기 제한을 가질 수 있음 
- 이러한 이유로, 대용량 데이터보다는 합리적인 크기의 하위 집합으로 chunk 하는 것이 좋음 

## Chunked Arrays 
-  <span style="color:	#00FFFF">arrow::ChunkedAray</span>는 배열과 마찬가지로 값의 논리적 시퀀스 
- 그러나 단순한 배열과 달리 chunked array (청크 배열) 는 전체 시퀀스가 메모리에서 물리적으로 연속적일 필요가 없음 
- 또한 청크 배열의 구성요소는 크기가 같을 필요는 없지만, 모두 같은 데이터 유형을 가져야 함 
- 청크 배열은 임의의 수의 배열을 집계하여 구성됨 
- 예시 :: 값 4를 보유해야하는 요소가 null인 1~8 사이의 배열과 같은 논리 값을 가지는 청크 배열 만들기 / 두 개의 분리된 chunk로 

```java
std::vector<std::shared_ptr<arrow::Array>> chunks;
std::shared_ptr<arrow::Array> array;

// Build first chunk
arrow::Int64Builder builder;
builder.Append(1);
builder.Append(2);
builder.Append(3);
if (!builder.Finish(&array).ok()) {
  // ... do something on array building failure
}
chunks.push_back(std::move(array));

// Build second chunk
builder.Reset();
builder.AppendNull();
builder.Append(5);
builder.Append(6);
builder.Append(7);
builder.Append(8);
if (!builder.Finish(&array).ok()) {
  // ... do something on array building failure
}
chunks.push_back(std::move(array));

auto chunked_array = std::make_shared<arrow::ChunkedArray>(std::move(chunks));

assert(chunked_array->num_chunks() == 2);
// Logical length in number of values
assert(chunked_array->length() == 8);
assert(chunked_array->null_count() == 1);
```

## Slicing
- 물리적 메모리 버퍼와 마찬가지로, 데이터의 일부 논리적 하위 시퀀스를 참조하는 배열 or 청크 배열을 얻기 위해, 배열 및 청크 배열의 zero-copy slices를 만드는 것이 가능 
- <span style="color:	#00FFFF">arrow::Array::Slice()</span> 및 <span style="color:	#00FFFF">arrow::ChunkedArray::Slice()</span> 메소드를 각각 호출하여 수행 




# Data Types 
- 데이터 유형은 물리적 데이터가 해석되는 방식을 제어함 
- 데이터 유형의 specification은 다른 프로그래밍 언어 및 런타임을 포함하여, 다른 Arrow 구현 간의 바이너리 상호 운용성을 허용
- 예를 들어, <span style="color:#FF8C00">pyarow.jvm bridge module</span>을 사용하여 Python과 Java 모두에서 복사 없이 동일한 데이터ㅔ 액세스할 수 있음 

## C++의 데이터 유형에 대한 정보를 나타내는 방법 
1. arrow::DataType 인스턴스 사용 (ex.함수 인수로)
2. arrow::DataType 구체적인 하위 클래스 사용 (ex.템플릿 매개변수로)
3. arrow::Type::type enum value 사용 (ex.switch 문의 조건으로)

- 1번이 가장 관용적이며 유연함. 런타임 매개변수 유형은 DataType 인스턴스로만 완전히 표현할 수 있음 
- 성능이 중요한 경우 (동적 타이핑과 다형성을 피하기 위해), 2번과 3번의 형식을 사용할 수 있으나, 매개변수 유형에 대해 어느정도의 런타임 전환이 필요할 수 있음
- Arrow 데이터 유형은 임의의 중첩을 허용하기 때문에, 컴파일 타임에 가능한 모든 유형을 구체화하는 것은 불가능 

## 데이터 유형 생성 
- 데이터 유형을 인스턴스화 하려면, 제공된 팩토리 함수를 호출 

```java
  std::shared_ptr<arrow::DataType> type;

	// A 16-bit integer type
	type = arrow::int16();
	// A 64-bit timestamp type (with microsecond granularity)
	type = arrow::timestamp(arrow::TimeUnit::MICRO);
	// A list type of single-precision floating-point values
	type = arrow::list(arrow::float32());
```


***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/docs/cpp/arrays.html>
- <https://arrow.apache.org/docs/cpp/datatypes.html>
- <https://arrow.apache.org/docs/cpp/tables.html>
