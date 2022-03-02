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
> - 이러한 버퍼는 값 데이터 자체와, 배열 항목이 null 값임을 나타내는 선택적 비트맵 버퍼로 구성 
> - 배열에 null 값이 없을 경우 비트맵 버퍼는 완전히 생략 가능 
- 배열의 개별 값에 액세스하는데 도움이 되는 각 유형에 대한 <span style="color:	#00FFFF">arrow::Array</span>의 구체적인 하위 클래스 존재 


## Building an Array 

### Availabe strategies  
- Arrow 객체는 변경할 수 없기 때문에, <span style="color:	#00FFFF">std::vector</span>처럼 직접 채울 수는 없음 
- 그 대신, 다음과 같은 방법을 사용 
> - 데이터가 이미 알맞은 레이아웃으로 메모리에 존재하는 경우          
>   : 해당 메모리를 <span style="color:	#00FFFF">arrow::Buffer</span> 인스턴스 내부에 래핑한 후 배열을 설명하는 <span style="color:	#00FFFF">arrow::ArrowData</span>를 구성 
> - 그렇지 않으면,          
>   : <span style="color:	#00FFFF">arrow::ArrayBuilder</span> 기본 클래스와 그것의 구체적인 하위 클래스가 배열 데이터를 점진적으로 구축하는데 도움이 됨 (Arrow 형식의 세부사항들을 직접 처리하지 않고도)

### ArrayBuilder 및 하위 클래스 사용 
- Int64 Arrow 배열을 만들기 위해, <span style="color:	#00FFFF">arrow::Int64Builder</span> 클래스 사용 
- 예시::값 4를 보유해야하는 요소가 null인 1~8 사이의 배열 생성 

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
> - 첫 번째 버퍼: 1|1|1|1|0|1|1|1 비트가 있는 단일 바이트로 구성된 null bitmap 보유           
>                least-significant bit(LSB) numbering을 사용하므로 배열의 4번째 항목이 null임을 나타냄 
> - 두 번째 버퍼: 단순하게 위의 모든 값을 포함하는 int64_t 배열         
>                네 번째 항목이 null이므로 버퍼의 해당 위치에 있는 값은 정의도지 않음 

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




***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/docs/cpp/arrays.html>
- <https://arrow.apache.org/docs/cpp/datatypes.html>
- <https://arrow.apache.org/docs/cpp/tables.html>
