## Deploy with GithubAction to EC2
- CI에 이어 .yml 파일에 GithubAction의 workflow 부터 작성.
```
    - name: Build docker image
      run: docker build -t iri051/app:latest .
    - name: Publish image to docker hub
      run: docker push iri051/app:latest

    - name: Ec2 docker-compose up
      uses: appleboy/ssh-action@master
      with:
        host: ${{secrets.HOST}}
        username: ubuntu
        key: ${{secrets.KEYPAIR}}
        script: |
          docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
          docker compose down
          docker rmi ${{ secrets.DOCKER_REPO }}:latest
          docker pull ${{ secrets.DOCKER_REPO }}:latest
          docker compose up -d
          docker logout
```
- docker image 부터 빌드 하고 compose-up 으로 container를 띄운다.
- GithubAction - Secrets 안에 개인정보 등을 저장 후 ${{secret.???}} 으로 불러와 사용한다.
  - Host : EC2 퍼블릭 주소. 비용 때문에 껏다켰다 할거라 계속 바뀔 것이다. </br>
            elastic 으로 유지 가능 하지만 비용때문에 껏다 킬 것인데 얘도 비용이 든다. 값싼 노동력을 활용하자.
  - KEYPAIR : pem 키 저장된 곳에서 cat 명령어로 내용물을 싹 긁은 후 넣어줬다.
  - 나머지 도커 관련된 이름, 비밀번호, 저장소 이름까지 함께 관리했다.

## EC2 인스턴스 관리
- micro로 감당이 안되는 듯 하다. medium 2cpu 4Gib 로 설정했다. 과금은 무서운것이다.
- ec2 콘솔에서 docker를 설치 해 두어야 한다.
  - run: 부분의 docker 명령어를 사용이 안된다. 저 docker build는 image를 build 하는 것이다.
- 인바운드 규칙에 port 80과 8080을 추가하자.
  - 콘솔에서 돌아가는걸 확인했는데 웹으로 실행이 안되면 당황스럽다.
