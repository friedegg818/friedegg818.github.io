---

title: "[Apache Arrow] 09.Arrow C++ Library-4" 

excerpt: Compute Functions / 학습 중

categories: 
    - Apache Arrow
tags:
    - [Apache Arrow, C++ Library, Compute Functions]

toc: true
toc_sticky: true

date: 2022-02-25
last_modified_At: 2022-03-03

---

## The generic Compute API 

### 함수 및 함수 레지스트리 
- 함수: 다양한 유형의 input에 대한 계산 작업 
- 내부적으로 함수는 구체적인 input type에 따라 하나 이상의 "커널"로 구현됨         
  (예시: 두개의 입력 값을 추가하는 함수는 input이 정수 or 부동 소수점인지에 따라 다른 커널을 가질 수 있음)
- 함수는 이름으로 조회할 수 있는 전역 <span style="color:#FF8C00">FunctionRegistry</span>에 저장됨

### Input shapes 
- Computation input은 Scalar, Array, ChunkedArray 같은 여러 데이터 모양의 태그가 지정된 통합 general Datum 클래스로 표시됨 
- 대부분의 Compute 함수는 배열(chunk or not) 및 스칼라 입력을 모두 지원하지만, 일부는 둘 중 하나만을 요구    
  (예시: <span style="color:	#00FFFF">sort_indices</span>는 첫번째이자 유일한 입력이 배열이어야 함 ) 

### Invoking functions 
- Compute 함수는 <span style="color:	#00FFFF">arrow::compute::CallFunction()</span>을 사용하여 이름으로 호출할 수 있음 

```java
std::shared_ptr<arrow::Array> numbers_array = ...;
std::shared_ptr<arrow::Scalar> increment = ...;
arrow::Datum incremented_datum;

ARROW_ASSIGN_OR_RAISE(incremented_datum,
arrow::compute::CallFunction("add", {numbers_array, increment}));
std::shared_ptr<Array> incremented_array = std::move(incremented_datum).make_array();

// std::shared_ptr<Array>에서 Datum으로의 암시적 변환을 허용 
```

- <span style="color:	#00FFFF">arrow::compute::Add()</span> 처럼 구체적인 API로 직접 사용도 가능 

```java
std::shared_ptr<arrow::Array> numbers_array = ...;
std::shared_ptr<arrow::Scalar> increment = ...;
arrow::Datum incremented_datum;

ARROW_ASSIGN_OR_RAISE(incremented_datum,
arrow::compute::Add(numbers_array, increment));
std::shared_ptr<Array> incremented_array = std::move(incremented_datum).make_array();
```

- 일부 함수는 함수의 정확한 의미를 결정하는 옵션 구조를 허용 or 요구함 

```java
ScalarAggregateOptions scalar_aggregate_options;
scalar_aggregate_options.skip_nulls = false;

std::shared_ptr<arrow::Array> array = ...;
arrow::Datum min_max;

ARROW_ASSIGN_OR_RAISE(min_max,
arrow::compute::CallFunction("min_max", {array},
&scalar_aggregate_options));

// Unpack struct scalar result (a two-field {"min", "max"} scalar)
std::shared_ptr<arrow::Scalar> min_value, max_value;
min_value = min_max.scalar_as<arrow::StructScalar>().value[0];
max_value = min_max.scalar_as<arrow::StructScalar>().value[1];
```

- 하지만, Grouped Aggregations는 CallFunction을 통해 호출할 수 없음



## Implicit casts 
- 커널이 인수 유형과 정확하게 일치하지 않는 경우, 함수는 실행 전에 인수를 변환해야 함       
  (예시: 사전 인코딩된 배열의 비교는 커널에서 직접 지원되지 않지만, Implicit casts(암시적 캐스트)를 만들어 디코딩된 배열과 비교할 수 있음)
- 각 함수는 암시적 캐스트 동작을 적절하게 정의할 수 있음           
  (예시: 비교 및 산술 커널에는 동일한 형식의 인수가 필요하며, 인수를 두 input의 모든 값을 수용할 수 있는 숫자 형식으로 promote하여 다른 숫자 형식에 대한 실행을 지원함)

### Common numeric type 
- Input numeric type set의 common numeric type (공통 숫자 유형)은 모든 입력 값을 수용할 수 있는 가장 작은 숫자 유형 
- 입력이 부동 소수점 유형인 경우, 공통 숫자 유형은 입력 중에서 가장 넓은 부동 소수점 유형 
- 그렇지 않으면 공통 숫자 유형은 정수
- input이 sign 된 경우, 공통 숫자 유형도 signed

|Input types|Common numeric type|Notes|
|:---:|:---:|:---:|
|int32,int32|int32| - |
|int16,int32|int32|최대 너비 32, LHS -> int32로 승격| 
|uint16,int32|int32|input 하나는 signed, unsigned된 것은 재정의|
|uint32,int32|int64|uint32의 범위를 수용하도록 확장| 
|uint16,uint32|uint32|모든 input이 unsigned, unsigned 유지|
|int16,uint32|int64| - |
|uint64,int16|int64|int64는 모든 uint64 값을 수용할 수 없음|
|float32,int32|float32|RHS -> float32로 승격|
|float32,float64|float64| - |
|float32,int64|float32|int64가 더 넓지만, float32로 승격| 



## Available functions 

### Type category 
- 지원되는 유형이 완전히 listing 되지 않도록 여러 general type category를 사용 
  + Numeric         
  : 정수 유형 (Int8 등) 및 부동 소수점 유형 (Float32, Float64, 때때로 Float16)        
    일부 함수는 Decimal128 및 Decimal256 입력도 허용 
  + Temporal         
  : 날짜 유형 (Date32, Date64), 시간 유형 (Time32, Time64), Timestamp, Duration, Interval 
  + Binary-like            
  : Binary, LargeBinary, FixedSizeBinary 
  + String-like        
  : String, LargeString 
  + Nested        
  : List-likes (FixedSizeList 포함), Struct, Union 및 Map과 같은 관련 유형 

- 함수의 구체적인 input 유형 지원 여부가 확실하지 않을 경우 시도해볼것 
- 지원되지 않는 입력 유형은 <span style="color:#FF8C00">Type Error Status</span>를 반환 

### Aggregations 
- 스칼라 집계는 (청크) 배열 또는 스칼라 값에서 작동하고 입력을 단일 출력 값으로 줄임 

### Grouped Aggregations ("group by")
- 그룹화된 집계는 직접 호출할 수 없지만, SQL-style "group by" 작업의 일부로 사용됨 
- 스칼라 집계와 마찬가지로, 그룹화된 집계는 여러 입력 값을 단일 출력 값으로 줄임 
- 그러나 그룹화된 집계는 입력의 모든 값을 집계하는 대신, 일부 "key" 열 집합에서 입력 값을 분할한 다음 각 그룹을 개별적으로 집계하여 그룹당 하나의 출력 값을 내보냄 
- 지원되는 집계 함수
  + 모든 함수 이름에는 접두사 <span style="color:#FF8C00">hash_</span>가 붙는데, 이는 위의 해당 스칼라와 구분되며 내부적으로 구현되는 방식을 반영

### Element-wise ("scalar") functions 
- 모든 element-wise function(요소별 함수)은 배열과 스칼라를 모두 입력으로 받아들임 

> **unary 함수의 의미** 
- 스칼라 입력은 스칼라 출력을 생성 
- 배열 입력은 배열 출력을 생성 

> **binary 함수의 의미** 
- (scalar, scalar) 입력은 스칼라 출력을 생성 
- (array, array) 입력은 배열 출력을 생성 (두 입력의 길이는 동일해야 함)
- (scalar, array), (array,scalar)는 배열 출력을 생성        
  스칼라 입력은 동일한 값이 N번 반복되는 다른 입력과 동일한 길이 N의 배열인 것처럼 처리됨 

***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/docs/cpp/compute.html#element-wise-scalar-functions>