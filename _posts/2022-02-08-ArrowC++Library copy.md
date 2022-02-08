---

title: "[Apache Arrow] 07. Arrow C++ Library" 

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

# Arrow C++ Library 

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

***

### <span style="color:#00CCCC">References</span>
- <https://arrow.apache.org/docs/cpp/overview.html>