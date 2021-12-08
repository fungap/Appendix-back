## 1. cookie 를 이용한 조회수 중복방지 
1. api 호출한 ip에 cookie를 던져줌
2. cookie 시간동안 조회수 안올라감<br>
- board_id 값을 붙여서 가져온 ip 에 보낸다. <br><br>
 ![image](https://user-images.githubusercontent.com/88120776/145164858-f495147f-9444-47be-a61c-d43d679f0d4e.png)
 ![image](https://user-images.githubusercontent.com/88120776/145164889-a01708bb-0d64-4d82-b801-a875cb40bc5e.png)
- 포스트맨으로 확인시 cookie 가 잘들어간다. <br><br>
 ![image](https://user-images.githubusercontent.com/88120776/145165397-20c75c8f-dd5a-40aa-93a3-38b462c6c341.png)
<br>

여기서 포스트맨으로 백에서 테스트시에 잘작동하였으나 프론트와 통신할 때는 작동하지 않음<br>
프론드 개발도구에서 노란색 경고창 문구를 확인 <br>
경고창 문구를 검색해보니 SameSite 에 대한 내용이었음 <br>
크롬 80버전 부터는 새로운 쿠키 정책이 적용 되어 Cookie의 SameSite속성의 기본값이 'None'에서 'Lax' 로 변경됨<br>
따라서 쿠키 처리 할대에 SameSite값을 None으로 해줘야 함<br>
프록시서버에서 SameSite 값을 설정하여 조회수 중복 해결<br><br>
![image](https://user-images.githubusercontent.com/88120776/145165877-f9768d71-2a18-4426-8c72-3ae3c2ebb122.png)

## 2. table에 없는 like_state 값 계산 
프론트 api 요청으로 table 에는 없는 like_state 값과 like_count , comment_count 값을 계산해야했습니다.<br>
![image](https://user-images.githubusercontent.com/88120776/145166344-ab29246b-9376-4e1c-82f2-3bfc12faaac2.png)

테이블 설계 에서 likes 테이블은 아래와 같았습니다.<br><br>
![image](https://user-images.githubusercontent.com/88120776/145166722-6bf3a1c1-94c3-4468-b8f2-4238b93330b1.png)<br><br>
특정 유저가 어떤 게시글의 좋아요를 누르면 위의 DB에 저장이 됩니다. 따라서 게시글에서 해당 유저로 로그인이 되었을때 likes table에 해당 colum이 있으면 좋아요를 누른것으로 반별 합니다.<br>
try
- 처음 sequelize문으로 시도 했으나 3개의 조인과 2개의 카운트 가 갖는 복잡도의 난이도 때문에 구현 실패하였습니다.
- 따라서 아래돠 같은 쿼리 case 문을 사용해 like state값을 likes 테이블에서 해당유저의 board_id 값이 있다면 'true' 없으면 'false' 를 반환하게 하였습니다.  
```
 case u.board_id
  when b.board_id then 'true'
  else 'false'
  end as like_state
```
- like state는 뽑아 냈지만 이 후 count 값에 터무니 없이 큰값이 들어가게 되는 문제가 있었습니다..
- 이것은 하나의 select 문에 여러 카운트와 join문을 사용 함으로써 중복으로 count 값이 적용되는 현상이었습니다.
- 따라서 한개의 select 문을 두개로 분리하여 select 하나당 하나의 count문을 사용함으로써 문제를 해결했습니다. 
```
select t1.board_id, t1.board_title, t1.board_image, t1.view_count, t1.like_count, t2.comment_count, t2.like_state from
  (SELECT b.board_id,b.board_title,b.board_image,b.view_count,count(l.board_id) as like_count
   
    FROM boards AS b
    left OUTER JOIN likes AS l
    ON b.board_id = l.board_id
    WHERE b.board_delete_code = 0
    GROUP BY b.board_id
    ORDER BY b.createdAt DESC) as t1
    join
    
  (select b.board_id, count(c.board_id) as comment_count,
  
  case u.board_id
  when b.board_id then 'true'
  else 'false'
  end as like_state
  FROM boards AS b
  LEFT OUTER JOIN comments AS c
  ON (b.board_id = c.board_id AND c.comment_delete_code=0)
  left outer join likes as u
  on b.board_id = u.board_id and u.user_id = ${user_id}
  group by b.board_id
  ORDER BY b.createdAt DESC) as t2
  on t1.board_id = t2.board_id
  ORDER BY t1.like_count DESC
  ```