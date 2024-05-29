#String

- String 은 불변. 새로 합칠 시 기존의 String 값은 GC로 처리.
    빈번해 질 경우 메모리 할당과 해제 반복, 성능저하.
---
- String Builder
  - 기존의 데이터에 더하는 방식.
    ```java
    sb.append("abc");
    sb.insert(int offset, String str);
    // 첫 번째와 두 번째 파라미터로 받는 숫자 인덱스에 위치한 문자열을 대체한다
    sb.replace(int index1, int index2, String str);
    

    ```
---
- String Buffer
  - thread-safe 하므로 여러 쓰레드에서 동시에 해당 문자열에 접근한다면 사용을 고려.
---
  1. 문자열 변경이 빈번하지 않는다면 String 사용을 고려
  2. 문자열이 빈번하게 변경되면서 멀티쓰레드 환경이라면 StringBuffer 사용을 고려
  3. 문자열이 빈번하게 변경되면서 멀티쓰레드 환경이 아니라면 StringBuilder 사용을 고려
  
