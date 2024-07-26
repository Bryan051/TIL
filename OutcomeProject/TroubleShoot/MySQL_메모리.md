## 메모리 사용량 문제
### 버퍼 풀 사이즈를 4g, 4g 로 설정.
- docker 의 총 메모리가 약 8기가여서 너무 무식하게 조절한 것 같다.
- 실제로 내 트랜잭션시 얼마정도 사용하는지 확인해보자.

```
  SHOW ENGINE INNODB STATUS;

  ----------------------
  BUFFER POOL AND MEMORY
  ----------------------
  Total large memory allocated 0
  Dictionary memory allocated 710981
  Buffer pool size   262122
  Free buffers       4096
  Database pages     251219
  Old database pages 92653  
```
- 총 버퍼 풀 크기는 262122 페이지이며, 이 중 4096 페이지가 자유 상태.
  - 대략 2GB 메모리
- 수정된 페이지는 없으며, 보류 중인 읽기/쓰기 작업도 없다.

- innodb_buffer_pool_size를 2GB로 줄인다.
- innodb_buffer_pool_instances를 2로 줄여 메모리 사용을 최적화.
-	innodb_log_file_size를 줄여 로그 파일의 메모리 사용을 줄이도록 한다.
```
  innodb_buffer_pool_size = 2G // 기존 4G
  innodb_buffer_pool_instances = 2 // 기존 4G
  innodb_log_file_size = 512M // 기존 1G
```
