## CI build 중 jar 파일을 찾지 못하는 문제

<img src ="https://github.com/Bryan051/TIL/assets/68111122/6ac84f92-976b-40d3-bfeb-de05afbdeafc" width="80%">

- Run docker 이전에 Run ./gradlew build 를 정상적으로 실행시켰는데 왜 jar 파일을 찾지 못하는 걸까?
### 결론
- Docker Hub 리포지토리 이름 확인: tags: ${{ secrets.DOCKERHUB_USERNAME }}/app:latest로 설정하여 해결
  - build 시 내 레포를 정확하게 지정해 주지 않아 생성 된 곳을 못찾았던 것.
### 과정
- 결론을 먼저 시도 했어도 안됐을것
- 이유 1. application properties 가 원격저장소에 그대로 있었다.
  - 있으면 안될내용이라 지우고 내용을 secrets 로 처리해서 생성까지 했다.
  ```
      # application properties
    - name: application.properties
      run: |
        cd src/main
        mkdir resources
        cd resources
        touch application.properties
        echo "${{ secrets.APPLICATION_PROPERTIES }}">> application.properties
      shell: bash
  ```
- "/build/libs/Outcome-0.0.1-SNAPSHOT.jar": not found
  - 이 에러가 도저히 지워지지않아 한 시도들.
    ```
      # build/libs 디렉토리의 파일 목록을 출력합니다.
    - name: Show Gradle Build Result
      run: |
          echo "Gradle build completed!"
          ls -R ./build/libs  
    ```
    ```
      # JAR 파일을 Docker 빌드 컨텍스트로 복사
    - name: Copy JAR to root
      run: cp build/libs/Outcome-0.0.1-SNAPSHOT.jar .
    ```
- 그럼에도 불구하고 같은 에러 유지. build/libs에 jar 파일이 있는것도 확인하고 docker로 복사까지 했는데 왜?
  ```
      빌드 컨텍스트 확인: Docker 빌드 컨텍스트가 올바르게 설정되어 있는지 확인합니다
  ```
- 로 힌트를 얻어 docker 이후 디렉토리가 문제인가? 라는 의문을 품고 위 결론의 방법으로 해결함.
