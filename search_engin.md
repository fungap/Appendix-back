## mySQL

1. Like operator

   ```
   SELECT column1, column2, ...
   FROM table_name
   WHERE columnN LIKE pattern;
   ```

   | LIKE Operator                                                 | Description                                                                    |
   | ------------------------------------------------------------- | ------------------------------------------------------------------------------ |
   | WHERE CustomerName LIKE 'a%'                                  | Finds any values that start with "a"                                           |
   | WHERE CustomerName LIKE '%a'                                  | Finds any values that end with "a"                                             |
   | <span style="color:red">WHERE CustomerName LIKE '%or%'</span> | <span style="color:red">Finds any values that have "or" in any position</span> |
   | WHERE CustomerName LIKE '\_r%'                                | Finds any values that have "r" in the second position                          |
   | WHERE CustomerName LIKE 'a\_%'                                | Finds any values that start with "a" and are at least 2 characters in length   |
   | WHERE CustomerName LIKE 'a\_\_%'                              | Finds any values that start with "a" and are at least 3 characters in length   |
   | WHERE ContactName LIKE 'a%o'                                  | Finds any values that start with "a" and ends with "o"                         |

   ```
   const {keyword} = req.query;
   const keywords = keyword.split(' ')
       const list_keywords = []

       for (let i = 0; i < keywords.length; i++) {
           if (i == 0) {
           list_keywords.push(
               `t1.board_title LIKE '%${keywords[i]}%' OR t1.board_content LIKE '%${keywords[i]}%'`
           )
           } else {
           list_keywords.push(
               `OR t1.board_title LIKE '%${keywords[i]}%' OR t1.board_content LIKE '%${keywords[i]}%'`
           )
           }
       }

   const newlist_keywords = list_keywords.toString().replace(/,/g, "");

   const query = `
   ...
   where ${newlist_keywords}`
   ```

## FULL TEXT INDEX

### 1. 개요

FULLTEXT INDEX는 일반적인 인덱스와는 달리 매우 빠르게 테이블의 모든 텍스트 필드를
검색합니다.

이 인덱스는 검색 엔진과 유사한 방법으로 자연어를 이용하여 데이터를 검색할 수 있도록 모든 데이터의 문자열 단어를 저장합니다.

- InnoDB 또는 MyISAM 에서 FULL TEXT INDEX 사용 가능하다.
- 전체 텍스트 인덱스는 VARCHAR, CHAR 또는 TEXT 열에 대해서만 생성가능하다.
- FULLTEXT 인덱스 정의는 CREATE TABLE 문에서 지정하거나 ALTER TABLE 또는 CREATE INDEX를 사용하여 나중에 추가할 수 있다.
- FULLTEXT 인덱스가 없는 큰 데이터 세트는 기존 FULLTEXT 인덱스가 있는 테이블에 데이터를 로드하는 것보다 테이블에 데이터를 로드하는 것이 훨씬 빠르다. 따라서 데이터를 로드한 후 인덱스를 생성 하여야 한다.

### 2. 설정 (이미 존재하는 Table)

+ INDEX 추가!
    > CREATE FULLTEXT INDEX idx_ft_title_and_body on articles(title, body);
    ALTER TABLE articles ADD FULLTEXT INDEX idx_ft_title_and_body(title, body);

+ token화 글자수 설정!!
    > innodb_ft_min_token_size=2
    ft_min_word_len=2
    //설정변경시 다시 full-text index 재생성이 필요하다.!

+ 저장된 단어들 확인!
    > SET GLOBAL innodb_ft_aux_table = 'test/articles';
    SET GLOBAL innodb_optimize_fulltext_only=ON;
    OPTIMIZE TABLE articles;

    SELECT WORD, DOC_COUNT, DOC_ID, POSITION FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_TABLE;


    ```
    const {keyword} = req.query;
    const keywords = keyword.split(' ')
    .
    .
    .
    const query = `
    ...
    SELECT * FROM boards WHERE MATCH (board_title,board_desc) AGAINST ('${keywords}' IN NATURAL LANGUAGE MODE)...`
    ```

## 검색엔진(ELK stack)

### 전문(full text) 검색 엔진

+ 루씬은 기본적으로 역파일 색인(inverted file index)라는 구조로 데이터를 저장합니다. 루씬을 사용하고 있는 Elasticsearch도 마찬가지로 색인된 모든 데이터를 역파일 색인 구조로 저장하여 가공된 텍스트를 검색한다. 이런 특성을 전문(full text) 검색이라고 한다.

+ JSON 문서 기반 Elasticsearch는 내부적으로는 역파일 색인 구조로 데이터를 저장하고 있으나, 사용자의 관점에서는 JSON 형식으로 데이터를 전달한다. JSON형식은 간결하고 개발자들이 다루기 편한 구조로 되어 있어 색인 할 대상 문서를 가공 하거나 다른 클라이언트 프로그램과 연동하기에 용이하다.

+ 또한 key-value 형식이 아닌 문서 기반으로 되어 있기에 복합적인 정보를 포함하는 형식의 문서를 있는 그대로 저장이 가능하며 사용자가 직관적으로 이해하고 사용할 수 있다. Elasticsearch에서 질의에 사용되는 쿼리문이나 쿼리에 대한 결과도 모두 JSON 형식으로 전달되고 리턴된다.

### RESTFul API

+ 현재 대규모 시스템들은 대부분 마이크로 서비스 아키텍처(MSA)를 기본으로 설계됩니다. 이러한 구조에 빠질 수 없는 것이 REST API와 같은 표준 인터페이스 이며, Elasticsearch는 Rest API를 기본으로 지원하고 모든 데이터 조회, 입력, 삭제를 http 프로토콜을 통해 Rest API로 처리한다.

### 멀티테넌시 (multitenancy)

+ Elasticsearch의 데이터들은 인덱스(Index) 라는 논리적인 집합 단위로 구성되며 서로 다른 저장소에 분산되어 저장된다. 서로 다른 인덱스들을 별도의 커넥션 없이 하나의 질의로 묶어서 검색하고, 검색 결과들을 하나의 출력으로 도출할 수 있는데, Elasticsearch의 이러한 특징을 멀티테넌시 라고 한다.

### 클러스터 구성

![image](https://user-images.githubusercontent.com/90595291/144285870-76934c26-002a-4543-991e-94c29e917008.png)
cpu2 메모리 4GB 스펙의 인스턴스 2개(첫 사용기간 3개월 무료!!) <br>
두개의 서버 모두 elasticsearch 각각의 서버에 logstash, kibana를 따로 사용중이다.
defalt 값으로 elasticsearch와 kibana에 heap memory가 1GB로 설정되어있어 메모리 4GB를 선택 하였다.(heap memory를 더 작게 부여할 수 있었지만 온전히 기능 이용하고 싶었다.)
