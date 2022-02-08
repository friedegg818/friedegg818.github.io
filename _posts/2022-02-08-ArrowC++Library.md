---

title: "[Apache Arrow] 07.Arrow C++ Library" 

excerpt: C++ Library 관련 사항 

categories: 
    - Apache Arrow
tags:
    - [Apache Arrow, Supported Environments, C++ Library]

toc: true
toc_sticky: true

date: 2022-02-08
last_modified_At: 2022-02-08

---

## 구성 및 목적 

### Physical layer
- **Memory management abstraction**: heap 할당, 파일의 메모리 매핑 또는 정적 메모리 영역과 같은 다양한 수단을 통해 할당될 수 있는 메모리에 대한 균일한 API 제공 
- 특히 **Buffer abstraction**는 물리적 데이터의 연속적인 영역을 나타냄 

### 1차원 layer 
- **Data types**: '물리적' 데이터의 '논리적' 해석을 제어. Arrow 내의 많은 작업은 컴파일 타임이나 런타임, 데이터 유형에 의해 매개변수화 됨 
- **Arrays**: 하나 이상의 버퍼를 데이터 형식으로 조합하여 연속된 논리적 시퀀스 값으로 볼 수 있음 
- **Chunked arrays**: 배열의 일반화. 여러 개의 동일한 유형의 배열로 구성

### 2차원 layer 
- **Schema**: 각각의 고유한 이름과 유형 및 선택적 메타데이터가 있는 여러 데이터 조각의 논리적 모음 
- **Table**: 스키마에 따른 청크 배열 모음. Arrow에서 가장 유능한 dataset-providing abstraction
- **Record batches**: 스키마로 설명되는 연속 배열 모음. 테이블의 점진적인 구성이나 직렬화를 허용

### Compute layer
- **Datums**: 유연한 데이터 세트 참조. (ex.배열이나 테이블 참조 등을 보관할 수 있음)
- **Kernels**: 함수에 대한 입출력 매개변수를 나타내는 주어진 데이터 세트의 루프에서 실행되는 특수 계산 함수 

### IO layer 
- **Stream**: 다양한 종류의 외부 데이터 (ex.압축 or 메모리 매핑) 를 통해 비정형 순차나 검색 가능한 액세스를 허용 

### IPC (Inter-Process Communication) layer 
- **Messaging format**으로 최소의 copy를 사용하여 프로세스 간 Arrow 데이터를 교환할 수 있음 

### File format layer 
- 다양한 파일 형식에서 Arrow 데이터를 읽고 쓰는 것이 가능 
- **Parquet**, **CSV**, **Orc** 또는 **Arrow-specific Feather format** 

### Devices layer 
- 기본적인 **CUDA Intergration**이 제공되어 GPU 할당 메모리로 지원하는 Arrow 데이터 설명 가능 

### Filesystem layer 
- Filesystem abstraction을 사용하여 로컬 파일 시스템이나 S3 버킷과 같은 다양한 storage backend에서 데이터를 읽고 쓸 수 있음 

<hr>

## 규칙 
- Arrow C++ API는 몇 가지 간단한 지침을 따름 
- 많은 규칙 및 예외가 존재 

### Language version 
- C++ 11과 호환됨 
- 몇 가지 backports는 <span style="color:orange">std::string_view</span> class와 같은 새로운 기능에 사용됨 

### Namespacing 
- 매크로를 제외한 모든 Arrow API는 arrow 네임스페이스 내부에 네임스페이스가 배치 및 중첩됨 

### Safe pointers 
- Arrow 객체는 safe pointer를 사용하여 전달 및 저장됨 
- 대부분은 <span style="color:orange">std::shared_ptr</span>, 때때로 <span style="color:orange">std::unique_ptr</span>

### 불변성 (Immutability)
- 많은 Arrow 객체는 변경할 수 없어서 일단 생성되면 논리적 속성을 더 이상 변경 불가 
- 번거롭고 오류가 발생하기 쉬운 동기화 없이, 멀티스레드 시나리오에서 활용할 수 있음 
- 물론 이에 대한 명백한 예외도 존재 (ex.IO 개체 or 변경 가능한 데이터 버퍼)

### Error reporting 
- 대부분의 API는 <span style="color:orange">arrow::Status</span> 인스턴스를 반환하여 성공,실패한 결과를 나타냄 
- Arrow는 자체 예외를 발생시키진 않지만 Third-party 예외, 특히 <span style="color:orange">std::bad_allow</span>를 통해 전파될 수 있음 

- API가 오류 코드나 성공적인 값을 반환할 수 있는 경우, 보통 템플릿 클래스 <span style="color:orange">arrow::Result</span>를 반환 
- 그러나 일부 API는 (더 이상 사용되지 않기는 함) <span style="color:orange">arrow::Status</span>를 반환하고 결과 값을 out-pointer 매개변수로 전달 



> **작업 결과 확인 예시** 

```java
  const int64_t buffer_size = 4096;
    
  auto maybe_buffer = 
  arrow::AllocateBufer(buffer_size, &buffer);
  if (!maybe_buffer.ok()) {
       // ...handle error
  } else { 
      std::shared_ptr<arrow::Buffer> buffer = 
      *maybe_buffer;
      // ... use allocated buffer
  }
```     



  > **The caller function 자체가 <span style="color:orange">arrow::Result</span> 또는 <span style="color:orange">arrow::Status</span>를 반환하고 실패한 결과를 전달하는 경우 사용할 수 있는 매크로** 
  
  1. ARROW_RETURN_NOT_OK: <span style="color:orange">arrow::Status</span> 매개변수를 사용하고 성공하지 못하면 반환 
  2. ARROW_ASSIGN_OR_RAISE: <span style="color:orange">arrow::Result</span> 매개변수를 사용하고 성공하면 `lvalue`에 결과를 할당. 오류가 발생하면 해당하는 <span style="color:orange">arrow::Status</span>를 반환 
  


> **예시** 

```java
   arrow::Status DoSomething() {
     const int64_t buffer_size = 4096;
     std::shared_ptr<arrow::Buffer> buffer;
     ARROW_ASSIGN_OR_RAISE(buffer, arrow:: AllocateBuffer(buffer_size));
     // ... allocation successful, do something with buffer below

     // success 반환  
     return Status::OK();
   }
```

<hr>

## 프로젝트에서 Arrow C++ 사용하기 
- 시스템에 Arrow C++ 라이브러리가 이미 있다고 가정했을 경우 

### CMake 

#### 기본 사용법 

- Minimal <span style="color:orange">CMakeLists.txt</span> 파일은 <span style="color:orange">my_example.cc</span> source 파일을 Arrow C++ 공유 라이브러리와 연결된 실행 파일로 컴파일 

```java
   project (MyExample)

   find_package(Arrow REQUIRED)

   add_executable(my_example my example.cc)
   target_link_libraries(my_example PRIVATE arrow_shared)
```


#### 사용 가능한 변수 및 대상 
- <span style="color:orange">find_package(Arrow REQUIRED)</span> 지시문은 CMake가 시스템에서 설치된 Arrow C++를 찾도록 요청함 

> 요청이 반환될 때, 몇 가지 CMake 변수가 설정됨 

 - <span style="color:orange">${Arrow_FOUND}</span> : Arrow C++ 라이브러리르 찾은 경우 true 반환 
 - <span style="color:orange">${ARROW_VERSION}</span> : Arrow 버전 문자열 
 - <span style="color:orange">${ARROW_FULL_SO_VERSION}</span> : Arrow DLL 버전 문자열 

> 그 외에 연결할 수 있는 some target (변수 X, 일반 문자열)

 - <span style="color:orange">arrow_shared</span> : Arrow 공유 라이브러리에 대한 링크 
 - <span style="color:orange">arrow_static</span> : Arrow 정적 라이브러리에 대한 링크 

```java
   !! 보통 Arrow 공유 라이브러리르 사용하는 것이 좋음 
   !! CMake는 대소문자를 구분하므로 위에 나열된 이름과 변수의 철자는 정확히 동일해야 함
```

### pkg-config 

#### 기본 사용법 

- 다음 command line에서 적절한 빌드 flag를 얻을 수 있음

```java
   pkg-config --cflgas --libarrow
```

- Arrow C++ 정적 라이브러리를 연결하려면, <span style="color:orange">--static</span> 옵션 추가 

```java
   pkg-config --cflags --libs --static arrow 
```

- Minimal Makefile은 <span style="color:orange">my_example.cc</span> source 파일을 Arrow C++ 공유 라이브러리와 연결된 실행 파일로 컴파일 

```java
   my_example: my_example:cc 
     $(CXX) -o $@ $(CXXFLAGS) $< $$(pkg-config --cflags --libs arrow)
```

> 다양한 빌드 시스템이 pkg-config를 지원 
- GNU Autotools 
- CMake (<span style="color:orange">find_package(Arrow)</span>를 대신 사용해야 함)
- Meson 등 

#### 사용 가능한 패키지 
- Arrow C++는 각 모듈에 대해 pkg-config 패키지를 제공 

> **패키지 종류** 
> - arrow-csv
> - arrow-cuda
> - arrow-dataset
> - arrow-filesystem
> - arrow-flight-testing
> - arrow-flight
> - arrow-json
> - arrow-orc
> - arrow-python-flight
> - arrow-python
> - arrow-tensorflow
> - arrow-testing
> - arrow
> - gandiva
> - parquet
> - plasma


### Linking 참고 사항 
- 일부 Arrow 구성 요소에는 프로젝트에서 사용할 수 있는 dependencies(종속성)가 존재 
- 프로젝트가 Arrow와 동일한 방식으로 (정적 or 동적) 이러한 dependencies의 동일한 버전을 연결하도록 주의해야 함 
- 그렇지 않으면 ODR 위반이 발생하고, 프로그램이 충돌하거나 데이터가 자동으로 손상될 수 있음 

```
  ** ODR (One Definition Rule) 
    : C++ 프로그래밍 언어 의 중요한 규칙
      객체 및 비 인라인 함수, 템플릿 및 유형은 둘 이상의 정의를 가질 수 없음
```

- 특히 Arrow Flight 및 그의 dependencies Protocol Buffers (Protobuf)와 gRPC가 문제를 일으킬 가능성이 높음 

> 따라서 Arrow Flight를 사용할 떄에는 다음 사항을 참고

- Arrow Flihgt가 정적 연결일 경우, Protobuf와 gRPC도 반드시 정적으로 연결되어야 하며 동적일 경우도 마찬가지 
- 일부 플랫폼은 Arrow Flight에게 최신이 아닌 Protobuf나 gRPC 버전을 제공할 수 있음 
- Arrow Flight는 이러한 dependencies를 번들 제공하므로 두 가지 버전의 Protobuf와 gRPC가 연결될 수 있음 → 혼용되지 않도록 주의해야 함 

> 권장 방법

- 각 dependency의 소스 및 정적/동적 연결 여부를 제어할 수 있는 소스에서 작성된 Arrow 버전에 의존하는 것이 가장 쉽고, 
- 또는 일관된 버전의 Arrow 및 dependencies를 관리할 Conda나 vcpkg같은 패키지 매니저의 Arrow를 사용하는 것이 좋음 


***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/docs/cpp/overview.html>