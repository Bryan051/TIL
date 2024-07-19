## 현 프로젝트에서 N + 1

- N+1 문제는 현 프로젝트에서 batch 동작을 진행할 때 문제가 될 것으로 보였다.</br>
  페이징으로 끊어서 가져와 문제가 생기지 않을것이라 생각했지만 본질적인 N+1 이 해결되지 않은 상태였다.</br>
  현재 작동방식은,
  1.	먼저 N개의 Video 엔티티를 불러온다.
  2.	각 Video에 대해 추가적인 쿼리를 실행하여 관련된 VideoView 데이터를 가져온다.

- 이렇게 되면 N개의 Video를 불러오는 1개의 쿼리와, 각 Video에 대해 VideoView를 조회하는 N개의 쿼리가 실행되기 때문에</br>
  결과적으로 총 N+1개의 쿼리가 실행되므로 성능에 문제가 발생할 수 있다.

- 페이징과 N+1 문제의 관계
  - 페이징 적용:   
    - 페이징을 사용하여 예를 들어 10개의 Video를 한 번에 가져온다면, 처음에는 10개의 Video를 가져오는 1개의 쿼리가 실행된다.
    -	이후 각 Video에 대해 관련된 VideoView를 가져오는 10개의 쿼리가 실행된다.
    -	결국, 한 페이지 당 총 11개의 쿼리가 실행된다.
  -	N+1 문제:   
    - 페이징으로 인해 N의 크기가 줄어들지만, 페이지 수만큼의 쿼리가 계속해서 실행된다.
    - 현재 1000개의 Video가 있고 페이지 크기가 10이라면, 총 100개의 페이지가 필요하다.
    -  각 페이지 당 11개의 쿼리가 실행되므로, 총 1100개의 쿼리가 실행된다.   

- 따라서, 페이징을 적용해도 N+1 문제는 크기만 줄어들 뿐 본질적으로 해결되지 않는다.
- 각 페이지에 대해 추가적인 N개의 쿼리가 계속해서 실행되기 때문에, 여전히 성능에 영향을 미칠 수 있다.   

## 리팩토링 1
- JpaPagingItemReader를 사용하여 Video 객체를 페이지 단위로 읽어온다. </br>
  LEFT JOIN FETCH를 사용하여 Video와 관련된 VideoView 데이터를 함께 가져온다.
  ```
        @Bean
    public JpaPagingItemReader<Video> videoReader() {
        return new JpaPagingItemReaderBuilder<Video>()
                .name("videoReader")
                .entityManagerFactory(streamingEntityManagerFactory.getObject())
                // 기존 left join x , 일별 정산이니 날짜 맞춰서 가져옴.
                .queryString("SELECT DISTINCT v FROM Video v LEFT JOIN FETCH v.videoViews vv WHERE vv.createdAt = :date")
                .pageSize(100) // 적절한 페이지 사이즈로 설정
                .build();
    }

  
  ```
- Processor에서 Video 객체와 관련된 VideoView 목록을 모두 가져와 필터링하고 통계, 정산.
  - 페이지 사이즈를 적절히 설정해 메모리 사용량과 성능 사이의 균형을 맞춘다. 페이지 사이즈는 데이터 양과 서버의 메모리 용량에 따라 조절.
---
- 리팩토링 이후 쿼리스트링에서 가져오는 하나의 video에 대한 videoview 의 갯수가 2만정도 예상이 되는데,   
    firstResult/maxResults specified with collection fetch; applying in memory... 이후로 멈추게 되었다.   
    ~~videoview를 페이징해서 처리하는 방식이 필요할 것 같다.~~
  
## 리팩토링 2
- 애플리케이션 레벨에서 데이터를 페이지별로 로드하고 집계하려 하면 videoview 의 갯수가 너무 많다.
- 단일 책임 원칙을 유지한채 (reader 에서 video만) processor 에서 count 와 sum 으로 해결하려 한다.
- COUNT와 SUM은 데이터베이스에서 집계 함수로 처리되므로, 데이터베이스가 해당 쿼리를 실행하여 단일 결과를 반환하도록 한다.
  ```java
     @Query("SELECT COUNT(v) FROM VideoView v WHERE v.userId <> :#{#video.userId} AND v.vidId = :video AND FUNCTION('DATE', v.createdAt) = :date")
    int countVideoViewsExcludingUserAndDate(@Param("video") Video video, @Param("date") LocalDate date);

    @Query("SELECT SUM(v.duration) FROM VideoView v WHERE v.userId <> :#{#video.userId} AND v.vidId = :video AND FUNCTION('DATE', v.createdAt) = :date")
    Long sumVideoViewDurationsExcludingUserAndDate(@Param("video") Video video, @Param("date") LocalDate date);

  ```

## 결과
- 배치(통계) 시간 - video_view 3천만개: 13분 </br>
  <img width="496" alt="image" src="https://github.com/user-attachments/assets/f34e8073-106b-49fa-82eb-1b9a6be9e02b">

### 페이지 사이즈
- 페이지 사이즈 늘리는 장점
  - 페이지 사이즈를 늘리게 되면 쿼리의 실행 횟수가 감소된다. video 1천개 - 페이지 10 * 실행횟수 100 / 페이지 100 * 실행횟수 10
  - 네트워크 트래픽이 감소하고 전반적인 성능이 향상된다.
- 단점
  - 메모리 사용량 증가에 따른 GC 부담 증가 및, 응답시간 증가.
  - 현재 1개의 비디오에 1만개의 videoview 가 대략적으로 있는 상황. (일데이터 1천만개, video 1천개)
    - 더 늘어날 수 있으니 현재 사이즈 유지.
### 페이지 사이즈와 청크 사이즈는 일치해야된다. 데이터 정합성.







