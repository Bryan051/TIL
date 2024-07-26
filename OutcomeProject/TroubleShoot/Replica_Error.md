## 에러 해결방법 모음짐
### Duplicate Key 오류가 주된 원인이었다.
### 상세 에러 확인
```
  SELECT * FROM performance_schema.replication_applier_status_by_worker;
```
 1062 | Worker 1 failed executing transaction 'ANONYMOUS' at source log mysql-bin.000049, end_log_pos 7247809;   
 Could not execute Write_rows event on table outcome.ad_rev; Duplicate entry '9641' for key 'ad_rev.PRIMARY',Error_code: 1062;   
 handler error HA_ERR_FOUND_DUPP_KEY; the event's source log mysql-bin.000049, end_log_pos 7247809

 - 광고 정산시 9641번 ad_rev 가 중복으로 들어간 듯 하다. 더 없을까?
   
### 복제 건너뛰기
```
  STOP SLAVE;
  SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1;
  START SLAVE;
```
- Duplicate entry '9641' 부터 10단위로 9651, 9661, 9671, 9681, 9691, 9701, 순으로 들어가고 있었다.
- 정산시 중복으로 들어가는 문제가 있었고 동시성 제어가 필요할 듯 하다.

### 급한 불 끄기
```
  STOP SLAVE;
  RESET SLAVE;
  CHANGE MASTER TO MASTER_HOST='master_host',
  MASTER_USER='replication_user',
  MASTER_PASSWORD='replication_password',
  MASTER_LOG_FILE='mysql-bin.xxxxxx',  -- 마스터 데이터베이스의 최신 binlog 파일 (SHOW MASTER STATUS; 명령어로 확인한 값)
  MASTER_LOG_POS=xxxx;                 -- 최신 binlog 파일의 위치 (SHOW MASTER STATUS; 명령어로 확인한 값)
  START SLAVE;
```
