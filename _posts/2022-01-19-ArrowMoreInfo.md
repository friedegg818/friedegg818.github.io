---

title: "[Apache Arrow] More Info" 

excerpt: Overview 내용 포함하여 Search를 통해 얻은 정보 정리 

categories: 
    - Apache Arrow
tags:
    - [Apache Arrow]

toc: true
toc_sticky: true

date: 2022-01-19
last_modified_At: 2022-01-19

---

## 개발 배경 
- 데이터 구조 직렬화 / 역직렬화에 엄청난 비효율이 존재하고, 그 과정에서 불필요한 copy가 생기면서 메모리와 CPU 리소스의 낭비 발생 

    ```
        * 직렬화: 데이터를 저장할 수 있는 형식으로 변환하는 프로세스 
    ```

- 모든 엔진이 사용할 수 있고 플랫폼 간에 원활하고 효율적으로 데이터를 공유할 수 있는 표준 in-memory 표현을 목표로 개발 

## Arrow 란? 

![Arrow](/assets/img/basicConcept.png)

- 개발 플랫폼 + 소프트웨어 라이브러리 (재사용 가능) 
- 파일 시스템 / 다양한 저장 형식 / 데이터베이스에 access 하기 위한 도구 → 빠른 데이터 access 및 이동 



- Columnar data 구조의 이점을 in-memory computing과 결합하여 복잡한 데이터 및 동적 스키마의 유연성 제공 
- 오픈소스 및 표준화된 방식으로 수행 

### Columnar data format
- 프로그래밍 언어 독립적인 컬럼 기반 메모리 포맷 
- 테이블 구조 데이터와 계층형 데이터 정의 
- CPU나 GPU 등 현대적 하드웨어에서 효율적인 분석 작업 목표

> **컬럼 기반 메모리 포맷** 
> - 모든 컴퓨터 언어가 이해할 수 있는 표준 컬럼 메모리 형식 
> - 통계적인 집계나 대용량 데이터 처리에 적합한 메모리 구조 
> - C, C++, Java, Python 등의 대부분의 언어를 지원하여 Client SDK (Software Development Kit)를 이용해 손쉬운 개발 가능 
> - Columnar 유관 여러 플랫폼이나 서비스에서 거의 호환됨 
>   → Arrow format만 지원하면 데이터 변환을 할 필요가 없음 
>   → 대규모 데이터 분석 작업에서는 변환 문제만 해결되어도 많은 시간 절약 가능 
>   → **데이터 분석 속도 향상**
