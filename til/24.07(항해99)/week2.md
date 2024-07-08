## 데이터생성

- adview, videoview 데이터 생성 로직까지 구현. 배치들어가면됨.

  ![image](https://github.com/Bryan051/TIL/assets/68111122/5a8145d1-7eb9-4491-8f3d-6cfab10b76ec)

  ![image](https://github.com/Bryan051/TIL/assets/68111122/59ccea6a-87be-4980-aed2-6b4537b49d49)

  user 1번이 video 6번(user2의 동영상, 배치시 내가 내 동영상을 본 기록은 카운트 하지 않을 것이기 때문) 재생,정지 시나리오.

  - video_ad 매핑 테이블. 31번과 32번에 6번 동영상에 대한 6,5번 광고가 붙어있다.
    5분에 1개꼴로 랜덤으로 광고는 저장된다. ad_position 300,600, ~ , vidlength/300
![image](https://github.com/Bryan051/TIL/assets/68111122/53d84945-7ce6-48da-a592-77c7ca48900e)

  - last_played 시간을 0,650, 950으로 동영상 시점을 3 번 저장해봤다.</br>
    video_view 동영상 히스토리 테이블이다.
    ![image](https://github.com/Bryan051/TIL/assets/68111122/baf18a5a-38dc-4caf-a1a3-c651094acba6)

  - 300초, 600초, ~ 시간차 순으로 삽입된 광고지점을 last_played 가 넘으면 ad_view가 생성이된다. </br>
  - 30번은 6번 광고, 31번은 5번 광고, 32번은 1번 광고이다. (300,600,900), last_played 950의 결과.
    ![image](https://github.com/Bryan051/TIL/assets/68111122/129fb46f-8204-4d3a-93c2-ce26445e9a65)

- ad_view에 데이터가 중복으로 들어가는 문제. </br>
  재생시점은 서비스단에서 last_played로 받아와야 한다. 하지만 지금 받아왔을때
  서버에서 그 시점 이전부터 전부 광고를 본것으로 간주하고 카운트해 중복으로 데이터가 쌓인다.</br>
  서비스단에서 본 시간만 넘어온다고 가정하고 더해서 넣어주거나 기존의 시점으로 생각 한 뒤 빼줘도 되지만 </br>
  근본적으로 들어오는 모든 값을 처음부터 다시 재생을 한다는 가정을 한다면 last_played 무의미할것. 고민된다.
  
- userId를 param 으로 가져오지 않고 jwt 토큰 값으로 가져 올 건데 requestDto나 postman에서 id 값을 직접 넣어주고있었다. 수정.
    




 



