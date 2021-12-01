Elasticsearch

## mySQL
1. Like operator
```
SELECT column1, column2, ...
FROM table_name
WHERE columnN LIKE pattern;
```
|LIKE Operator|Description|
|------|---|
|WHERE CustomerName LIKE 'a%'|Finds any values that start with "a"|
|WHERE CustomerName LIKE '%a'|Finds any values that end with "a"|
|<span style="color:red">WHERE CustomerName LIKE '%or%'</span>|<span style="color:red">Finds any values that have "or" in any position</span>|
|WHERE CustomerName LIKE '_r%'|Finds any values that have "r" in the second position|
|WHERE CustomerName LIKE 'a_%'|Finds any values that start with "a" and are at least 2 characters in length|
|WHERE CustomerName LIKE 'a__%'|Finds any values that start with "a" and are at least 3 characters in length|
|WHERE ContactName LIKE 'a%o'|Finds any values that start with "a" and ends with "o"|

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
```

```
## FULL TEXT INDEX

FULLTEXT INDEX는 일반적인 인덱스와는 달리 매우 빠르게 테이블의 모든 텍스트 필드를 
검색합니다.

이 인덱스는 검색 엔진과 유사한 방법으로 자연어를 이용하여 데이터를 검색할 수 있도록 모든
 데이터의 문자열 단어를 저장합니다.

- InnoDB 또는 MyISAM 에서 FULL TEXT INDEX 사용 가능하다.
- 전체 텍스트 인덱스는 VARCHAR, CHAR 또는 TEXT 열에 대해서만 생성가능하다.
- FULLTEXT 인덱스 정의는 CREATE TABLE 문에서 지정하거나 ALTER TABLE 또는 CREATE INDEX를 사용하여 나중에 추가할 수 있다.
- FULLTEXT 인덱스가 없는 큰 데이터 세트는 기존 FULLTEXT 인덱스가 있는 테이블에 데이터를 로드하는 것보다 테이블에 데이터를 로드하는 것이 훨씬 빠르다. 따라서 데이터를 로드한 후 인덱스를 생성 하여야 한다.
 
token화 글자수 설정!!
>innodb_ft_min_token_size=2
ft_min_word_len=2


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