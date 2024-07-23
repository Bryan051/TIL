## Auto_Increment 불일치

![image](https://github.com/user-attachments/assets/afd4135d-51f4-4511-8516-8bdad9acd865)
- ENGINE=InnoDB AUTO_INCREMENT=40026856 과 같이
   1천만 단위로 넣었지만 auto increment 가 몇만개 더 올라가 있는 것을 볼 수 있다.
- 트랜잭션 단위가 너무 컸을 때 중지한 적이 있는데, 그럴 때 올라가는 것은 확인했지만 이번엔 정상적으로 종료되었음에도 발생했다.

- 동시성 문제:   
여러 트랜잭션이 동시에 INSERT 명령어를 실행하면서 AUTO_INCREMENT 값이 예상보다 더 크게 증가할 수 있다.
- 해결:
  - 트랜잭션 단위를 배치 사이즈로 조절 후 프로시저방식을 사용하면 해결이 가능하나 시간이 너무 소요된다.
  - 번거롭지만 더미데이터를 집어넣는 경우 지금은 수작업으로 처리하는게 훨씬 빨랐다.
    ```
      SET @sql = CONCAT('ALTER TABLE ad_view AUTO_INCREMENT = ', 30000001);
      PREPARE stmt FROM @sql;
      EXECUTE stmt;
      DEALLOCATE PREPARE stmt;
    ```




### 추가 (MySQL) JPA Bulk Insert 의 문제
BulkInsert란 Insert 쿼리를 한번에 처리하는 것을 의미한다.   
```
  INSERT IGNORE INTO ad_view (created_at, video_ad_id)
  SELECT
      CURDATE(),
      FLOOR(1 +RAND() * 3000)
  FROM
```
- Insert 합치기를 하려면 JdbcUrl 옵션에 rewriteBatchedStatements=true이 필수로 설정되어 있으면 바로 적용이 가능다.
- 단, Id 생성 전략이 auto_increment일 경우 JPA를 통한 save는 Insert 합치기가 적용되지 않는다.
- 결론은 Auto_increment이면 JPA가 아닌 JdbcTemplate과 같은 네이티브 쿼리를 작성하는 경우에만 insert합치기를 통한 bulk insert가 지원된다.
