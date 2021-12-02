[readme로-돌아가기](https://github.com/fungap/fungap-back)

## Chatting


![room별 유저리스트](https://user-images.githubusercontent.com/46738141/144252640-1731eee9-611b-41a1-891a-a722bafb84d7.png)
위 그림 처럼 각 방의 유저리스트라는 배열을 선언하고

프론트에서 채팅방 페이지에 들어오면 join_chat 이벤트를 보내 룸안에 들어가고 유저목록 배열에 넣어주고
채팅방 페이지에서 나가면 user_left 이벤트로 룸에서 나가고 유저목록 배열에서 제거하는 형태로 구현하였다.
![join_chat](https://user-images.githubusercontent.com/46738141/144252484-9dc97e23-21af-4cfd-a7d8-5fdab1a02f15.png)

![user_left](https://user-images.githubusercontent.com/46738141/144252253-11a6a924-a4db-47a1-b0a8-3b95f9e1cc87.png)

![image](https://user-images.githubusercontent.com/46738141/144253287-c0192721-17c9-46dd-bda5-800362c06c3b.png)

언뜻 보기엔 잘 구현이 된 것 처럼 보였지만 다른 페이지로의 이동이 아닌 브라우저를 바로 종료해 버리면
바로 socket disconnect가 일어나 user_left 이벤트를 보내지 못하는 현상이 발생하였고, 그로 인해 유저 목록에서 지워지지 못해 반 영구적으로 남아 있는 현상으로 발전하였다.
![계속해서 남아있는 유령유저](https://user-images.githubusercontent.com/46738141/144254783-66fdb2e2-85d3-4e83-83cd-6076dbf6ac1b.png)

처음에는 disconnect 이벤트에 될거라 생각해 시도해 보았지만
되지 않았고 disconnecting 이라는 이벤트를 찾게 된다.
![image](https://user-images.githubusercontent.com/46738141/144255261-78db6760-d7bd-43fc-8150-b53166097752.png)
이것은 연결이 끊어지는 중, 말 그대로 연결이 끊어지는 이벤트가 발생하면 그 전에 이벤트를 보내는 효자 이벤트였다.
이 것을 이용해 disconnect가 발생하면 user_left랑 비슷하게 모든 방에서 나가게 구현 하였더니
잘 작동하였다
![image](https://user-images.githubusercontent.com/46738141/144264832-adb47e29-bfc3-4186-b1c1-7bcc98f1e2bc.png)

