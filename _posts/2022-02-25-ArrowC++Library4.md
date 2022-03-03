---

title: "[Apache Arrow] 09.Arrow C++ Library-4" 

excerpt: Compute Functions

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

- <span style="color:	#00FFFF">arrow::compute::Add() 처럼 구체적인 API로 직접 사용도 가능 

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


***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/docs/cpp/compute.html#element-wise-scalar-functions>