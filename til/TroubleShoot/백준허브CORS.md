
## CORS란 무엇일까
- 먼저 SOP(Same Origin Policy) 정책이란 단어 그대로 동일한 출처에 대한 정책을 말한다.
  - 그리고 이 SOP 정책은 '동일한 출처에서만 리소스를 공유할 수 있다.'라는 법률을 가지고 있다. </br>
    즉, 동일 출처(Same-Origin) 서버에 있는 리소스는 자유로이 가져올수 있지만,</br>
    다른 출처(Cross-Origin) 서버에 있는 이미지나 유튜브 영상 같은 리소스는 상호작용이 불가능하다는 말이다.
- 교차 출처 리소스 공유(Cross-Origin Resource Sharing, CORS)는 단어 그대로 다른 출처의 리소스 공유에 대한 허용/비허용 정책이다.
  - 아무리 보안이 중요하지만, 개발을 하다 보면 기능상 어쩔 수 없이 다른 출처 간의 상호작용을 해야 하는 케이스도 있으며, 또한 실무적으로 다른 회사의 서버 API를 이용해야 하는 상황도 존재한다.
  - 따라서 이와 같은 예외 사항을 두기 위해 CORS 정책을 허용하는 리소스에 한해 다른 출처라도 받아들인다는 것이다.

## cors 내 에러
fetch 요청 헤더 오리진에 * 들어간 것을 확인
![image](https://github.com/Bryan051/TIL/assets/68111122/6469ce92-f746-4a8e-b9af-fbbbdf2840ab)


## 해결
![image](https://github.com/Bryan051/TIL/assets/68111122/25dadf7b-d144-451c-83c2-79b9de1c94af)

- CORS 정책 상 get (credential) 시 Access-Control-Allow-Origin 에 * 을 사용할 수 없는데</br>
  위 익스텐션으로 * 추가해서 해결했다. (default to * 라고 하는데 왠지 모르겠지만 적용이 안됐다.)
- 다만 보안 문제가 있기에 사용안할땐 끄는게 맞다.
- 토큰 문제일 수 도 있어 익스텐션 끄고 토큰 재발급을 받아봐야겠다.


