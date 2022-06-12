### SQL, 절차적 vs 선언적

#### SQL은 선언적
SQL은 일반적으로 선언적이라 분류된다.
ex) SELECT * From Users WHERE id = ?;
에서, 우리는 원하는 것을 명시하지만, 그를 수행하는 방법에 대해선 명확히 언급하지 않는다. 

#### 튜링 완전
[튜링 완전 - 분기 점프(goto)가 가능하다. 읽기/쓰기가 가능하다.](https://www.cs.odu.edu/~zeil/cs390/latest/Public/turing-complete/index.html)
[분기 점프(goto)가 있는 모든 프로그램은 if문, 루프문을 가지면서 flag를 나타내기 위한 변수를 사용하거나, 중복된 코드를 작성하는 방식으로 완전히 변환될 수 있음이 증명되어있다.](https://en.wikipedia.org/wiki/Control_flow#cite_note-1)
따라서
- 읽기/쓰기가 가능하면서 goto문이 있을 경우
- 읽기/스기가 가능하면서 if문과 loop문이 모두 있을 경우
튜링 완전하다.
다만 실제로 튜링 완전을 위해서는 무한한 저장 공간이 필요하나, 물리적으로 불가능하므로 메모리는 유한하더라도 편의상 튜링 완전하다고 부르는 경우가 많다.


#### SQL의 절차적 확장
하지만 초창기의 선언적 SQL은 튜링 완전하지 못하였다. 쿼리문은 읽기/쓰기는 가능했으나, 분기 점프는 구현하지 못하였다.
일반적으로 사용되는 프로그래밍 언어들은 거의 모두 튜링 완전하다. (HTML은 튜링 완전하지 않으며, 프로그래밍 언어라고 불리지 않는 이유중에 하나도 그것이다).
SQL은 튜링 완전하지 못하고, 범용 프로그래밍 언어는 튜링 완전하므로, 범용 프로그래밍 언어에서는 가능한 것이 SQL에서는 불가능한 등의 문제가 생겼다.

이에 SQL:1999 표준에서는 SQL을 절차적으로 확장하는 시도를 하게 된다.

##### 영구 저장 모듈(Persistent Storage Module, PSM)
SQL:1999 에서 추가된 내용이다.
제어 흐름(분기 점프), 예외 처리, 지역 변수, 매개변수, 커서 등의 내용이 포함된다.
PSM덕분에, SQL에서도 범용 프로그래밍 언어에서 가능하던 대부분의 기능을 구현할 수 있게 되었다.
이러한 절차적 확장이 적용된 SQL들은 튜링 완전하다고 볼 수 있다.

아래는 PSM의 문법 혹은 사용 예시들이다.

변수는 declare문을 사용하며, 유효한 SQL타입을 모두 사용 가능하다.

```SQL
WHILE (boolean experssion) do
  (sequence of statements);
END WHILE
```
```SQL
REPEAT
  (sequence of statements);
UNTIL (boolean expresssion)
END REPEAT
```
while과 repeat문은 일반적인 루프를 위해 사용될 수 있다.
```SQL
DECLARE n INTEGER default 0;
FOR r AS
  SELECT budget FROM department
  WHERE dept_name = 'MUSIC'
DO
  SET n = n - r.budget
END FOR
```
for문은 프로그램의 질의 결과를 가지고 반복문을 돌리는 데 사용할 수 있다.
leave문은 break처럼, iterate문은 continue처럼 사용할 수 있다.
반복문들을 하나의 트랜잭션 안에서 사용하고 싶다면, begin atomic ... end로 묶어주면 된다.
```SQL
if (boolean expression)
  then (statement or compound statement)
 elseif (boolean expression)
  then (statement or compound statement)
else (statement or compound statement)
end if
```

```SQL
교재 185쪽(그림 5.8)
```

```SQL
DECLARE out_of_classroom_seats CONDITION
DECLARE EXIT HANDLER FOR out_of_classroom_seats
BEGIN
  (sequence of statements);
END


DECLARE CONTINUE HANDLER FOR 1051
BEGIN
  (sequence of statements);
END;

DECLARE CONTINUE HANDLER FOR SQLWARNING
BEGIN
  (sequence of statements);
END;
```
DECLARE CONTINUE HANDLER도 가능하다. EXIT HANDLER는 실행 중 오류 발생 시 begin-end문을 빠져나가며, CONTINUE HANLDER는 오류가 발생한 다음 문장을 계속해서 수행한다.
미리 정의된 예외처리들: sqlexception, sqlwarning, not found
https://dev.mysql.com/doc/refman/8.0/en/declare-handler.html

다만, SQL표준으로 등록되어있음에도, 이를 구현하는 것은 선택사항이다. 또한, 표준이 나오기 이전부터 많은 DBMS들이 해당 기능의 필요성을 느껴서 자체적으로 제작을 마쳤었다. 그리고 이 자체적인 기능들은 표준이 없었기에, 기능은 비슷해도 문법은 제각각이었고, 표준이 제정된 이후로도 바뀌지 않았다. 따라서 프로시저나 함수 등을 정의하려먼 각 SQL 별로 상세 문법에 대해서 더 파악해야할 필요성이 있다.


##### 재귀
PostgreSQL에서는 튜링 완전성을 가짐을 증명하기 위해 재귀를 사용한다.
cyclic tag system은 튜링 완전하다고 증명되어있다. 그리고 PostgreSQL에서는 recursion을 사용해 이를 [구현](https://wiki.postgresql.org/index.php?title=Cyclic_Tag_System&oldid=15106)하였다. 따라서 PostgreSQL도 튜링완전하다.

recursion과 관련된 내용에 대한 내용은 다음 주에 더 공부하고 다루겠습니다...


### prepared statement
예전에는 sql injection을 방지할 수 있다는 사실만 알고 있었고, 그래서 사용했었다. 하지만 이와 별개로, 속도적인 장점도 존재하였다.
- prepared statement: 쿼리를 컴파일하고 캐싱하는 작업이 있어, 여러 번의 호출이 있다면 일반 statement에 비해서 훨씬 빠를 수 있다.
- 다만 한 번만 실행되는 쿼리라면 오히려 비효율적인 면도 있다.

[자주 실행되는 느린 쿼리에 대해 prepared statement을 저장한 후의 성능이 개선된 경우](https://orangematter.solarwinds.com/2014/11/19/analyzing-prepared-statement-performance/)

해당 부분은 그 외sql 튜닝 스터디 교재 1장에서 나오는 내용이 더 자세하고 좋아 보여 생락하였습니다..







### 고급 집계 기능
순위를 알려주는 rank, percent_rank 등이 있었다.
한 테이블을 파티션별로 분할헤서 aggregate, window function 등을 적용할 수도 있었다.
window function은 mysql 8버전부터 지원하는 부분들이라서, 미국 캐시워크의 버전으로는 수행해볼 수 없었다.
이 부분도 다움주에 더 공부하고 다루겠습니다...
