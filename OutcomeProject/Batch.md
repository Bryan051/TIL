## 배치 구조 Refactoring

- reader(paging), processor, writer(chunk) 구조에서</br>
현재 코드는 video 를 읽어오고 processor 에서 videoview 를 조회해 count 하는 방식이다.

- 문득 든 생각이 paging 으로 많은 데이터를 끊어서 가져오는게 reader 의 의의라면, videoview 를 대상으로 읽는게 나은 방법이 아닐까? 하는 의문에 대한 고찰.

1. join fetch 를 해도 페이징 처리를 해서 넘겨오게 되면 비디오 v 에 해당하는 videoview가 전부 다 넘어오는지
2. 다 넘어온다 해도 페이지 사이즈에 따라 혹은 chunk 사이즈에 따라 videoview 가 많을 때 누락이 돼 통계에 영향을 주진 않을지
3. count, sum 메서드가 VideoViewReadRepository에서 들고오는데 reader에서의 역할이 겹치는게 맞는지
4. 겹친다면 이 방식이 구조의 의의에 맞는 것인지

- 페이징 처리를 사용하면, 쿼리에서 지정한 페이지 크기만큼의 결과만 한 번에 가져온다.</br>
  즉, Video에 대해 여러 개의 VideoView가 있는 경우, 해당 VideoView 데이터가 많으면 여러 페이지에 걸쳐 분할되어 반환될 수 있다.</br>
  따라서 하나의 페이지에서 특정 Video에 대한 모든 VideoView를 가져오는 보장은 없다.

- 페이징을 통해 데이터를 가져올 때, Video와 VideoView 간의 관계를 모두 포함하지 못할 수 있다.</br>
  이로 인해 Video에 대한 모든 VideoView가 한 번에 처리되지 않을 수 있으며, 이는 통계 계산에 영향을 미칠 수 있다.</br>
  특히, VideoView의 수가 많아 페이지 크기를 초과하는 경우, 일부 데이터가 누락될 가능성이 있다.

- 현재 설정에서는 reader는 Video 엔티티를 읽어오고, processor가 VideoView 데이터를 조회하여 통계를 계산하는 방식이다.</br>
  reader의 역할이 Video 엔티티를 읽어오는 것에 한정되기 때문에, VideoView 데이터를 조회하는 것은 processor에서 처리한다.</br>
  따라서 reader와 processor의 역할이 겹친다고 볼 수는 없다.

## 결론
- 현재 구조는 배치 처리의 단계별 책임 분리를 잘 따르고 있다고 생각. reader는 Video 엔티티를 읽어오고, processor는 통계를 계산하며, writer는 결과를 저장한다.</br>
  이는 각 단계가 단일 책임 원칙(Single Responsibility Principle)을 따르고 있기 때문에, 구조의 의의에 부합한다.
## 개선점
- reader 에서 video 를 다 들고 오고 해당 video 에 대한 videoview 를 찾는 방식이기 때문에 N+1 문제가 발생할 것으로 보인다.
- joinfetch 로 reader로 한번에 다 들고 올 수는 없다(paging). 더 나은 방식이 있을까?
  - reader 에서 id 만 가져오고("SELECT v.id FROM Video v"), processor 에서 join fetch를 한다면?
- 지금은 또 다른 히스토리 테이블을 쌓는 방식이어서 괜찮지만 update 를 해야되는, 누적합을 고려할 테이블이 있으면 </br>
  reader로 읽어 올 때 writer 에서 계산 한 값을 또 계산하는 경우가 발생할 가능성이 있다.
