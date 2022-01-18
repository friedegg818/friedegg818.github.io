---

title: "[Apache Arrow] Overview" 

categories: 
    - Apache Arrow
tags:
    - [Apache Arrow]

toc: true
toc_sticky: true

date: 2022-01-12
last_modified_At: 2022-01-13

---


# Apache Arrow 

**In-memory 분석을 위한 Cross-language 개발 플랫폼**

- Apache Arrow는 대용량 dataset을 처리하고 전달하는 고성능 application 구축을 위한 소프트웨어 개발 플랫폼
- 분석 알고리즘의 성능 향상 및 시스템이나 프로그래밍 언어 간의 효율적인 데이터 이동을 위해 설계됨

### 중요 Component : In-memory columnar format 
- 최신 하드웨어에서 처리하는데 매우 효율적
- 광범위한 사용 사례에 적합한 범용 테이블 형식 데이터 표현
- 표준화 되고 언어에 구애받지 않는 사양 
- Random access와 Streaming / scan 기반 Workload 지원 
- 분석 database system, data frame library 등의 요구사항을 지원하도록 설계된 풍부한 data type system을 보유 (중첩 및 사용자 정의 data type 포함)

#### 특징 
> **1. Columnar is Fast**

![columnar](/assets/img/columnarformat.png)

- Apache Arrow format을 사용하면 연산 루틴과 실행 엔진이 커다란 데이터 chunk를 스캔, 반복할 때의 효율성을 극대화할 수 있음 
- 특히, 연속된 Columnar layout은 최신 프로세서에 포함된 최신 SIMD 연산을 사용하여 벡터화 가능 

<br>

> **2. Standardization Saves** 

![standard](/assets/img/standardization.png)

**Problem**
- 표준 Columnar data format이 없으면 모든 데이터베이스 및 언어는 자체 내부 data format을 구현해야 함 → 엄청난 낭비 
- 한 시스템에서 다른 시스템으로 데이터를 이동하려면 높은 비용의 직렬화 및 역직렬화가 필요하고, 각 data format에 대한 공통 알고리즘을 다시 작성해야하는 경우가 많음 

**Solution**

- Arrow의 In-memory columnar data foramt은 이러한 문제에 즉시 적용 가능한 솔루션 
- Arrow를 사용하거나 지원하는 시스템은 조금의 비용, 혹은 비용을 전혀 들이지 않고 시스템 간 데이터를 전송할 수 있음 → 분석 워크로드의 직렬화 오버헤드 양이 급격히 감소
- 이러한 Saving 효과 외에도 표준화된 메모리 형식은 여러 언어 간의 알고리즘 라이브러리 재사용을 용이하게 함 

<br>

> **3. Arrow Libraries**

- Arrow 프로젝트에는 다양한 언어로 Arrow columnar format의 데이터로 작업할 수 있는 Library가 포함되어 있음 
  + `C++, C#, Go, Java, JavaScript, Julia, Rust Library` → format에 대한 정확도를 보장하기 위해 서로에 대해 통합 테스트를 거침 
  + `C, MATLAB, Pythoh, R, Ruby용 Arrow Library` → `C++ Library` 기반
 
- 이러한 공식 Library를 사용하면 Arrow columnar format 자체를 구현하지 않아도 Third-party 프로젝트에서 Arrow 데이터로 작업할 수 있음 
-  또한 원격 Storage system에서 데이터를 in & out 하고 네트워크 인터페이스를 통해 Arrow format data를 이동하는 것과 관련된 시스템 문제를 지원하는 많은 소프트웨어 구성 요소가 포함됨 → 이러한 구성 요소 중 일부는 Columnar format이 전혀 사용되지 않는 시나리오에서도 사용할 수 있음 
- Arrow Dataset에 대한 분석 작업 또는 쿼리를 수행하기 위한 알고리즘 라이브러리도 존재
