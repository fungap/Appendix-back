## typescript

  <br>
  기술적 첼린지 : typescript <br>
  목표 : 모든 javascript 파일을 typescript 파일로 변환 <br>
  왜? : 자바스크립트의 특징인 동적 타이핑에 의한 오류를 typescript를 사용 하여 제거 할 수 있다. <br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;실제로 변환해보면서 정적타이핑을 몸으로 체감하고 명시적인 타입지정으로 코드짠 사람의 의도를 더욱명확하게 전달할수 있고 코<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;드의 가독성을 높이기 위함<br>
  
  ### undifined <br>
  
  undifinde에 대한 처리를 그동안 엄청 소홀해 왔다는 것을 깨닫는다. typescript는 이러한 타입의 처리를 놓치지 않게 하며 여태 놓쳐왔던 undifined와 null값을 예외적으로 처리 할수 있게 되어서
  코드의 오류를 줄여줌을 체감했다.
  아래 코드는 user_mbti 와 count가 있을수 있고 없을수 있다. <br>
  없게 된다면 undifined가 들어가므로 그에대한 처리를 해줘야 한다. 처리를 안해주었을경우 오류가 뜨게 된다.
  ![image](https://user-images.githubusercontent.com/88120776/144174811-61092e29-5992-4d1b-b87c-df4a0fdd653a.png)


  ### 우리는 이 값이 확실이 있는것임을 확신한다.
  ### sequelize 와 typescript
  
