---

title: "[Apache Arrow] 04.Projects Powered by Arrow" 

excerpt: Apache Arrow를 사용하는 시스템 

categories: 
    - Apache Arrow
tags:
    - [Apache Arrow]

toc: true
toc_sticky: true

date: 2022-01-20
last_modified_At: 2022-01-20

---

## Apache Parquet
- 데이터 처리 프레임워크 
- 데이터 모델이나 프로그래밍 언어 선택에 관계 없이 Hadoop ecosystem의 모든 프로젝트에서 사용할 수 있는 columnar storage format 
- C++ 및 Java 구현은 Arrow 데이터 구조에 대한 벡터화된 읽기 및 쓰기를 제공함 

## Apache Spark 
- 대규모 데이터 처리를 위한 빠르고 일반적인 엔진 
- Apache Arrow를 사용하여 Spark DataFrame과 pandas DataFrame 간의 변환 성능 향상 
- PySpark에서 벡터화된 사용자 정의 함수(`pands udf`) 세트 활성화 

## ArcPy
- 공간 분석, 데이터 관리 및 변환 작업을 수행하고 자동화하기 위해 ArcGIS 제품군 내에서 작업하기 위한 Esri의 포괄적이고 강력한 API 
- 입력 및 출력으로 Arrow table 지원 

## AWS Data Wrangler 
- AWS 데이터 관련 서비스와 DataFrame을 연결하는 AWS로 pandas 라이브러리의 기능 확장 

## Bodo 
- 주류 기업을 위한 고성능 컴퓨팅 아키텍처를 대중화하여 Python 분석 워크로드를 효율적으로 확장할 수 있는 범용 Python 분석 엔진 
- Arrow를 사용하여 Parquet 파일에 대한 I/O와 데이터 작업에 대한 내부 지원 

## Cylon 
- 기존 Big Data 및 AI/ML 프레임워크와 원활하게 통합될 수 있는 오픈 소스 고성능 분산 데이터 처리 라이브러리 
- Arrow 메모리 형식을 사용하고 C++, Java 및 Python 언어 연결을 지원 

## Dask 
- 동적 작업 그래프의 병렬 및 분산 실행을 위한 Python 라이브러리 
- Parquet 파일에 액세스 하기 위해 pyarrow 사용 지원 

## Data Preview 
- 텍스트 및 binary data 파일을 보기 위한 Visual Studio Code Extension 
- Arrow 데이터 파일 및 스키마를 로드, 변환 및 저장하기 위해 Arrow JS API를 사용 

## Falcon 
- 대화형 데이터 탐색 도구 
- Arrow JavaScript 모듈을 사용하여 Arrow 파일을 로드 

## FASTDATA.io 
- Arrow를 입력 및 출력으로 지원하고 내부적으로 사용하여 성능을 극대화 
- 기존 Apache Spark API와 함께 사용 가능 

## Fletcher 
- Apache Arrow in-memory 형식을 사용하는 도구 및 프레임워크와 FPGA 가속기를 통합할 수 있는 프레임워크 
- Arrow Schemas 세트에서 Fletcher는 가속 시 커널이 사용하기 쉬운 인터페이스를 통해 시스템 대역폭에서 RecordBatch를 읽고 쓸 수 있도록 고도로 최적화된 하드웨어 구조 생성 

## GeoMesa 
- 분산 컴퓨팅 시스템에서 대규모 지리 공간 쿼리 및 분석을 가능하게 하는 도구 모음 
- 브라우저 내 시각화 및 추가 분석에 사용할 수 있는 Arrow IPC 형식의 쿼리 결과 지원 

## GOAI 
- GPU 도구 및 공급업체 전반에 걸친 Arrow 기반 분석을 위한 Open GPU-Accelerated Analytics Initiative 

## graphique 
- Arrow table과 parquet data sets를 위한 GraphQL 서비스 

## Graphistry 
- 보안, 사기 방지 및 관련 조사를 위해 사용하는 강력한 시각적 조사 플랫폼 
- NodeJS GPU 백엔드 및 클라이언트 라이브러리에서 Arrow 사용 

## HASH 
- 브라우저 내 IDE를 사용하여 시뮬레이션을 구축, 실행 및 학습하기 위한 오픈 코어 플랫폼 
- Apache Arrow를 사용하여 Rus, JavaScript 및 Python에 걸쳐 작성된 시뮬레이션 로직 간에 copy 없는 데이터 전송 가능 

## InAccel 
- FPGA를 활용하는 기계 학습 가속 프레임워크 
- Apache Arrow가 지원하는 데이터 프레임을 지원하여 구현된 ML 알고리즘에 대한 입력 역할 

## InfluxDB IOx
- Rust로 작성된 오픈 소스 time series 데이터베이스 
- 인메모리 형식으로 Apache Arrow, 지속성 형식으로 Apache Parquet 및 RPC용 Apache Arrow Flight를 사용함 

## libgdf 
- CUDA 기반 분석 기능의 AC 라이브러리 및 구조화된 데이터에 대한 GPU IPC 지원 
- Arrow IPC 형식을 사용하고, 분석 기능에서 Arrow 메모리 레이아웃을 대상으로 함 

## MATLAB 
- 엔지니어와 과학자를 위한 numerical 컴퓨팅 환경 
- Apache Arrow를 사용하여 Parquet 및 Feather 파일 읽기 및 쓰기 지원 

## OmniSci 
- GPU와 CPU 모두에서 실행되도록 설계된 in-memory 컬럼형 SQL 엔진 
- CUDA IPC 핸들을 통한 데이터 수집 및 교환을 위해 Arrow 지원 

## pandas 
- 파이썬 프로그래머를 위한 데이터 분석 툴킷 
- pyarrow를 사용하여 Parquet 파일 읽기 및 쓰기 지원 

## Petastorm 
- Apache Parquet 형식의 데이터 세트에서 직접 딥 러닝 모델의 단일 머신 또는 분산 교육 및 평가를 가능하게 함 
- Tensorflow, Pytorch 및 PySpark와 같은 주요 Python 기반 기계 학습 프레임워크를 지원 