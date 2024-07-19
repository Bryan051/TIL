![image](https://github.com/user-attachments/assets/e8c30912-8893-45b9-a57b-5b00a6868721)

- History 성 기록 테이블들을 별도로 분리했다.
- FK는 데이터간의 관계를 유지하고 데이터 무결성을 보장하는 용도로 사용한다.</br>
  하지만 사용 해 보니 데이터 일관성의 유지가 양날의 검인 듯 하다. 생각보다 제약사항이 많아 더미데이터를 넣을 때 꽤나 고생했다.
### msa 로 리펙토링을 생각중인데, 엔티티를 분리저장하게되면 결국 fk를 다 끊어내야되는데 어떻게 관리를 해야 할 까?
- FK가 없으면 데이터베이스는 참조 무결성을 보장할 수 없다. 이는 한 테이블의 값이 다른 테이블의 값을 참조할 때, 그 값이 실제로 존재하는지 확인할 방법이 없다.
  - 대신 api 통신을 통해 데이터를 조회하면 해결할 수 있다.
- FK가 없으면 부모 엔티티와 자식 엔티티 간의 데이터 일관성을 유지하기 어렵다.
  - Video 서비스가 사용자 정보를 필요로 할 때 User 서비스의 데이터를 복제하여 사용하는 방법이 있다.
  - 이벤트 브로커: Kafka, RabbitMQ와 같은 메시지 브로커를 사용하여 이벤트를 게시하고 구독하여 데이터 변경을 동기화할 수 있다.
- 위와 같은 사항들을 참고하여 분리에 신경써야 할 듯 하다.