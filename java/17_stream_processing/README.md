# 스트림 요소 처리

## 목차

- [x] [스트림이란?](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.1_stream.md)
- [x] [내부 반복자](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.2_internal_iterator.md)
- [x] [중간 처리와 최종 처리](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.3_processing_steps.md)
- [x] [리소스로부터 스트림 얻기](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.4_stream_source.md)
- [x] [요소 걸러내기(필터링)](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.5_filtering.md)
- [x] [요소 변환(매핑)](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.6_mapping.md)
- [x] [요소 정렬](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.7_sorting.md)
- [x] [요소를 하나씩 처리(루핑)](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.8_looping.md)
- [x] [요소 조건 만족 여부(매칭)](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.9_matching.md)
- [x] [요소 기본 집계](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.10_basic_element_aggregation.md)
- [x] [요소 커스텀 집계](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.11_custom_element_aggregation.md)
- [x] [요소 수집](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.12_element_collection.md)
- [x] [요소 병렬 처리](https://github.com/2025-cs-study/long-octopus-leg/blob/main/java/17_stream_processing/17.13_parallel_element_processing.md)

## 면접 예상 질문
- Java의 스트림(Stream)이 무엇인지 설명해주세요.
- Collection과 Stream의 차이점을 설명해주세요.
- 내부 반복자와 외부 반복자의 차이점을 설명해주세요.
- I/O Stream이 무엇인지 설명해주세요.
- 필터 스트림이 무엇인지 설명해주세요.
- 버퍼링과 Buffered 필터 스트림에 대해서 설명해주세요.
- 스트림 파이프라인(pipelines)이 무엇인지 설명해주세요.
- 문자 스트림이 무엇인지 설명해주세요.
- 스트림을 생성하는 방법에 대해 아는대로 설명해주세요.
- Files.lines() 메소드를 사용하여 파일로부터 스트림을 얻는 과정을 설명해주세요.
- 스트림의 필터링 메소드에 대해서 설명해주세요.
- 스트림에서 map()과 flatMap()의 차이점을 설명해주세요.
- 기본 타입 스트림(IntStream, LongStream, DoubleStream)에서 요소를 변환하는 방법에 대해서 설명해주세요.
- 스트림 요소를 정렬하는 방법에 대해서 설명해주세요.
- Comparable, Comparator의 차이점에 대해 설명해주세요.
- 스트림에서 루핑이란 무엇인지 설명하고, peek()와 forEach() 메소드의 차이점에 대해 설명해주세요.
- 함수형 인터페이스란 무엇인지 설명해주세요.
- Consumer 인터페이스는 어떤 역할을 하는지 설명해주세요.
- 스트림에서 지연 연산(lazy evaluation)이란 무엇이며, 어떤 장점이 있는지 설명해주세요.
- peek() 메소드를 사용하여 데이터를 수정하는 것이 권장되지 않는 이유에 대해 설명해주세요.
- 스트림에서 매칭이란 무엇인지 설명하고, 어떤 메소드가 있는지 설명해주세요.
- Predicate 인터페이스는 어떤 역할을 하는지 설명해주세요.
- Optional 클래스에 대해 설명해주세요.
- OptionalDouble, OptionalInt, OptionalLong과 같은 기본형 특화 Optional 클래스들이 존재하는 이유에 대해 설명해주세요.
- reduce() 메소드에 identity 매개값의 유무에 따른 차이점에 대해 설명해주세요.
- BinaryOperator 인터페이스의 역할에 대해 설명해주세요.
- collect() 메소드의 용도와 역할에 대해 설명해주세요.
- Collector와 Collectors에 대해 설명해주세요.
- Collectors.groupingBy()에서 두 번째 매개변수의 역할은 무엇인지 설명해주세요.
- 동시성과 병렬성에 대해 설명해주세요.
- 병렬 스트림의 내부 동작 방식(포크조인 프레임워크)을 설명해주세요.
- 병렬 처리 성능에 영향을 미치는 요인은 어떤게 있는지 설명해주세요.
- 병렬 스트림 사용시 데이터의 순서(ordering)가 유지되는지 설명해주세요.