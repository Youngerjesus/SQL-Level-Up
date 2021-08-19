# 인덱스 기본

***

## 목차

- 인덱스 구조 및 탐색

- 인덱스 기본 사용법

- 인덱스 확장기능 사용법

***

## 1. 인덱스 구조 및 사용법 

본 절은 인덱스 구조와 탐색 원리에 대해서 설명한다. 

아주 기본적인 내용이라고 생각하지만 많은 개발자들이 인덱스의 기본 구조와 원리를 정확하게 이해하지 못하는 경우가 많다. 

본 절에서 가장 강조하고 싶은 부분은 인덱스의 탐색 과정은 수직적 탐색 이후에 수평적 탐색을 이용한다는 것이다.

이런 인덱스의 탐색 과정을 이해하고 나면 인덱스 구조가 명확해지고 인덱스 튜닝 방법에 대해서도 정확하게 이해할 수 있다. 

### 1.1 미리 보는 인덱스 튜닝

인덱스 튜닝은 다음 장인 3장에서 자세히 보겠지만 이번 장에서 그 핵심 원리를 빠르게 한번 설명하려고 한다. 

이유는 인덱스 구조를 바라보는 시각을 SQL 튜닝에 맞추기 위해서다. 

우리가 데이터베이스에서 데이터를 찾을 땐 크게 두 가지 방법이 있다. 아직까지도 이 방법을 벗어나지 못하고 있다.  

- 테이블 전체를 스캔한다. 

- 인덱스를 이용해 검색한다. 

#### 인덱스 튜닝의 핵심 원리 

인덱스를 사용하는 이유는 큰 테이블에서 소량의 데이터를 찾기 위해서 사용한다. 

온라인 트랜잭션 처리(Online Transaction Processing) 시스템에서는 소량 데이터를 주로 검색하므로 인덱스 튜닝이 무엇보다 중요하다. 

세부적인 인덱스 튜닝 방법은 여러개가 있겠지만 여기서는 가장 큰 핵심 요소를 설명하려고 한다. 

첫 번째는 인덱스 스캔 과정에서 발생하는 비효율을 줄이는 것이다. 즉 이 말은 불 필요한 인덱스 스캔을 줄이는 '인덱스 스캔 효율화 튜닝' 이다. 

예를 들면 인덱스가 (이름 + 시력) 으로 이뤄진 경우와 (시력 + 이름) 으로 이뤄진 경우에 홍길동의 시력을 찾고 싶다고 해보자. 

전자는 인덱스를 통해서 일부만 검색하면 되지만 후자의 인덱스의 경우에는 모든 사람들을 다 찾아봐야한다.  

두 번째는 테이블 엑세스 횟수를 줄이는 것이다.  

인덱스 스캔 이후에 인덱스에 있는 칼럼으로 쿼리를 커버할 수 없다면 즉 커버링 인덱스를 사용하는 것이 아니라면 테이블에 엑세스 해야한다.

이런 테이블 엑세스는 랜덩 I/O 기반의 엑세스기 때문에 반드시 줄이는 것이 성능에 좋다. 

예를 들어 데이터베이스에서 시력이 1.0 ~ 1.5 사이에 있는 홍길동을 모두 찾고 싶다고 가정해보자.

인덱스가 (이름 + 시력) 으로 되어있다면 테이블에 엑세스 하지 않아도 되니까 가장 좋지만 인덱스가 이름만으로 또는 시력만으로 되어있다고 해보자. 

여기서 시력이 1.0 ~ 1.5 인 사람은 총 50명 정도 된다. 그리고 이름이 홍길동인 사람은 총 5명 있다. 

이런 상황에서 테이블 엑세스 횟수를 비교해보면 이름만으로 인덱스가 걸린 사람의 경우가 훨씬 적다. 

정리하자면 인덱스는 인덱스 스캔을 효율적으로 탐색할 수 있어야 하고 실제로 테이블에 엑세스 하는 횟수가 적도록 해야한다. 

아직 인덱스 구조도 설명하지 않았는데 이렇게 인덱스 튜닝의 핵심 요소부터 간단하게 설명한 이유는 핵심을 먼저 전달하기 위해서다. 

SQL 튜닝을 딱 한마디로 정리하자면 랜덤 I/O 와의 전쟁이다. 이것들을 줄이도록 노력해야한다. 

#### SQL 튜닝은 랜덤 I/O 와의 전쟁

데이터베이스의 성능이 느린 이유는 랜덤 I/O 때문이다. 읽어야 할 데이터 량이 많고 그 과정에서 디스크 I/O 가 발생하기 때문에 느리다. 

이런 성능을 위해 DBMS 는 랜덤 I/O 를 줄이기 위해 많이 개선되었다. 

IoT, 클러스터, 파티션, Batch I/O 등 모두 랜덤 I/O 를 줄이는게 핵심이다.

SQL 조인 같은 경우 가장 많이 사용하는 Nested Loop Join 도 대량의 데이터를 조인할 때 생기는 랜덤 I/O 를 줄이기 위해서 Hash Join, Sort Merge Join 이 개발되었다. 

### 1.2 인덱스 구조

인덱스는 대용량 테이블에서 필요한 데이터에만 효율적으로 접근학디 위해서 개발되었다. 

국어 사전 같은 경우에 ㄱ,ㄴ,ㄷ,ㄹ,ㅁ ... 순으로 정렬되어 있어서 필요한 단어를 찾기위해 모두 엑세스 할 필요가 없는 것 처럼 인덱스의 원리도 이와 유사하다. 

데이터베이스는 인덱스가 없다면 처음부터 끝까지 모든 데이터를 읽어야한다. 

하지만 인덱스가 있다면 읽어야 하는 시작지점을 찾을 수 있고 쭉 읽다가 끝나는 지점을 알 수 있다. 

DBMS 는 일반적으로 인덱스를 B*Tree 구조를 이용한다.

B*Tree 는 가장 꼭대기인 Root 각 노드의 주소를 가지고 있는 Branch 실제 테이블의 주소를 가지고 있는 Leaf 로 이뤄져있다.

Root 와 Branch 노드의 키 값에는 하위 블록에 대한 주소를 가지고 있다. 

Leaf 블록의 각 레코드는 각 키 값으로 정렬되어있고 테이블 레코드의 주소를 가리키는 ROWID 값이 들어가 있다. 

ROWID 는 데이터 블록의 주소 와 로우 번호로 이뤄져있다. 이를 통해 실제 레코드가 있는 데이터 블록을 가지고 올 수 있다. 

### 1.3 인덱스의 수직적 탐색 

인덱스의 자료구조를 알았으니 실제 탐색 방법에 대해서 알아보자. 

인덱스의 탐색 과정은 수직적 탐색 과정과 수평적 탐색 과정으로 이뤄져있다.

인덱스의 수직적 탐색 과정은 데이터를 찾을 시작 지점을 찾는 것이다. 

인덱스의 루트 블록부터 시작해서 조건에 맞는 브런치 블록을 거쳐서 처음 찾을 리프 블록을 찾는 것이다.

인덱스를 효율적으로 사용하지 못하는 경우에는 이런 시작 지점이 여러개가 될 수 있는 쿼리라서 인덱스의 일부만 탐색하는게 아니라 전체를 봐야하는 경우라서 효율적이지 못할 수 있다.

### 1.4 인덱스의 수평적 탐색

수직적 타핑을 통해 스캔의 시작지점을 찾았다면 찾고자 하는 데이터를 끝까지 찾기 위해 이제는 수평적 탐색을 시작한다. 

리프 블록의 경우의 키 값들은 서로 정렬되어있고 Double Linked List 로 연결되어있다. 그래서 서로의 앞 뒤 블록으로 이동하면서 찾는게 가능하다. 

### 1.5 결합 인덱스 구조와 탐색 

칼럼을 두 개 이상 사용해서 인덱스를 만드는 것도 가능하다.

단순히 이름 이나 성별 가지고 인덱스를 만드는 경우도 있을 수 있겠지만 (실제로는 성별 가지고 인덱스를 만들지 않는다. 분별도가 너무 낮기 때문에)

이름과 성별을 합쳐서 인덱스를 만들 수도 있다. 

여기서 좀 알려주고 싶은건 인덱스가 (이름 + 성별) 로 이뤄진 경우와 (성별 + 이름) 으로 이뤄진 경우가 어떤 차이가 있는지에 대해서 좀 설명하겠다. 

여러 개의 칼럼을 이용해서 인덱스를 만들었다면 가장 왼쪽의 순서부터 정렬이 되어있다고 생각하면 된다. 

전자의 경우에는 리프 블록의 레코드에 이렇게 정렬되어 있을 것이다. 강가현 남자, 강가현 여자, 강나현 남자, 강나현 여자 ...

하지만 후자의 경우에는 이렇게 되어 있을 것이다. 남자 강가현, 남자 강나현, ... 여자 강가현, 여자 강나현 ... 

이런 구조를 알고 인덱스 튜닝을 생각해보면 된다. 인덱스 튜닝의 가장 큰 핵심 요소중 하나가 인덱스 스캔의 범위를 최소화 해라 였다. 

내가 만약에 홍길동인 사람을 모두 가지고 오고 싶다면 인덱스는 (이름 + 성별) 로 되어 있는 경우가 스캔의 범위가 더 적다. 

하지만 홍길동인 남자만 가지고 오고 싶다 등 특정한 성별과 이름을 모두 조합해서 하나의 경우만 가지고 오고 싶다면 인덱스가 (이름 + 성별) 이던 (성별 + 이름) 이던 탐색하는 스캔의 범위는 같을 것이다.

***

## 2. 인덱스의 기본 사용법

인덱스를 사용하는 기본 방법은 Index Range Scan 하는 방법을 익히는 것이다.

Index Range Scan 이 무엇인지 알고 어떻게 하면 Range Scan 이 안되는지를 이해하면 Index Range Scan 하는 방법을 자연스럽게 익히게 될 것이다.

### 2.1 인덱스를 사용한다는 것 

사전에서 글로벌로 시작하는 단어를 찾아본다고 가정해보자. 

쉽게 찾을 것이다. ㄱ 부터 시작하는 단어부터 해서 쭉쭉 찾을 수 있다. 

이러한 이유는 글자들이 ㄱ,ㄴ,ㄷ,ㄹ, ... 순으로 정렬되어 있고 한군데 모여있기 때문이다. 그래서 글로벌로 시작하는 글자를 찾을 수 있다.

인덱스로 설명하자면 수직적 스캔이 가능하기 떄문이다. 이로인해 글자가 시작하는 지점을 찾을 수 있다. 

하지만 글로벌 이라는 글자를 포함하는 단어를 찾는다고 생각해보자. 또 4 ~ 6 번째 글자가 글로벌인 단어를 찾는다고 생각해보자. 

이번에는 찾기 쉽지 않다. 모든 사전을 다 뒤져봐야 찾을 수 있을지도 모른다.

이렇게 시작 지점을 찾을 수 없는 경우에는 인덱스를 모두 뒤져보는 (Index Full Scan) 을 하던지 테이블 전체 스캔 (Table Full Scan) 을 하던지 할 것이다.

이처럼 인덱스를 통해 시작 지점을 찾을 수 있는 방식은 인덱스의 칼럼이 가공되거나, 중간값인 경우에는 사용할 수 없다. 

정리해서 인덱스를 정상적으로 사용할 수 있다는 말은 인덱스를 통해 시작 지점인 리프 블록을 찾아서 수평적으로 탐색하다가 내가 원하는 탐색이 아니면 멈출 수 있다는 것이다. 

### 2.2 인덱스를 Range Scan 할 수 없는 이유

인덱스 칼럼을 가공하면 인덱스를 정상적으로 사용할 수 없다. 

이는 기본중에 기본이다. 하지만 이를 제대로 설명할 수 있는 사람은 없다. 

인덱스 칼럼을 가공하면 인덱스를 사용할 수 없는 이유로는 `인덱스 스캔의 시작 지점` 을 제대로 찾을 수 없기 때문이다. 

예를 들어서 초등학교 내에 2007년 1월에 태어난 학생을 찾는다고 해보자.

우선 2007년 1월 1일에 태어난 학생을 찾고 쭉 탐색하면서 (정렬되어 있으니까.) 2007년 2월 1일인 학생이 나오면 스캔을 멈추면 된다.

이렇게 인덱스 Range Scan 은 시작 지점과 종료 지점이 있다. 

하지만 만약에 년도와 상관없이 5월 달에 태어난 학생들을 찾아보자 라는 쿼리를 한다고 해보자. 

시작 지점을 찾을 수 있을까? 멈출 지점을 찾을 수 있을까? 없다. 

그러므로 Index Full Scan 을 하던지 Table Full Scan 을 하던지 할 것이다. 

즉 정리해서 인덱스 Range Scan 을 할 수 없는 경우는 다음과 같다.

- 칼럼을 가공하는 경우 

- LIKE 를 이용해 %패턴% 문자열 검색을 하는 경우

- WHERE 정에 OR 연산을 하는 경우

- WHERE 절에 IN 연산을 하는 경우 

### 2.3 더 중요한 인덱스 사용 조건

인덱스를 사용하는 데 있어 더 중요한 선행 조건이 있디.

인덱스가 (소속팀 + 사원명 + 연령) 으로 걸려 있다고 해보자. 

다음과 같은 SQL 문은 인덱스를 사용할까?

```sql
SELECT 사원번호, 소속팀, 인원, 입사일자, 전화번호
FROM 사원
WHERE 사원명 = '홍길동'
```

인덱스가 소속팀 + 사원명 + 연령으로 구성되어 있다고 했다. 즉 이는 소속팀으로 먼저 정렬되고 그 다음 사원명으로 정렬되고 그 다음 연령으로 정렬된다. 

이 말은 가장 첫번째 기준인 소속팀으로 먼저 정렬되어 있지 사원명은 같은 소속팀 내에서만 정렬되어 있다는 뜻이다. 사원명으로 정렬되어 있지 않다. 

결합 칼럼 인덱스의 경우 인덱스 Range Scan 을 하기 위한 가장 큰 첫번째 원칙은 제일 왼쪽에 있는 칼럼을 기준으로 검색을 해야 가능하다는 뜻이다. 

그렇다면 아래와 같은 SQL 문은 인덱스를 사용할 수 있을까? 

```sql
# 인덱스는 기준연도 + 과세구분코드 + 보고회차 + 실명확인번호로 이뤄져있다.

SELECT *
FROM TXA1234
WHERE 기준연도 = :str_1234
AND substr(과세구분코드, 1, 4) = :txtr_dcd
AND 보고회차 = :rpt_tmrd
AND 실명확인번호 = :rnm_cnfm_no
``` 

여기서 과세구분코드는 인덱스 칼럼에 포함되지만 가공되어 있다. 인덱스 Range Scan 을 탈 수 있을까? 

가능하다. 왜냐하면 선행조건인 기준연도가 정상적으로 가공되지 않았고 사용되기 때문이다.

다시 말하지만 인덱스를 Range Scan 하기 위해서는 인덱스 선두 칼럼이 가공되지 않은 상태로 조건절에 있어야 한다. 

#### 인덱스 잘 타니까 튜닝 끝?

인덱스를 탄다는 말은 Index Range Scan 이 된다는 뜻이다.

인덱스가 성공적으로 Range Scan 하면 끝난 것일까? 

인덱스 튜닝의 핵심은 인덱스 스캔의 범위가 줄어야 한다는 뜻이다. 즉 내가 원하는 데이터를 찾기 위해서 얼마나 효율적인 스캔을 했는지가 중요하다.

위의 예제에서 기준연도를 사용하니까 Index Range Scan 은 정상적으로 작동한다. 

하지만 그 다음 정렬 요소인 과세구분코드가 가공되어 있으므로 모든 과세구분코드를 탐색할 것이다. 스캔의 범위를 줄여주지 않는다. 

이 같은 경우는 스캔의 범위를 줄여주지 않는다. 

정리하자면 인덱스 Range Scan 이 일어나더라도 스캔의 범위가 줄어들지 않는다면 이는 튜닝이 더 필요할 수 있다.

### 2.4 인덱스를 이용한 소트 연산 생략

인덱스는 키 값을 기반을 정렬되어 있다. 그래서 Range Scan 이 가능하기도 하다. 

그러므로 인덱스를 이용한 결과의 조회는 부수적으로 정렬되어 있는 효과가 있다. 

즉 인덱스를 사용한 Range Scan 에서는 ORDER_BY 와 같은 연산이 기본적으로 적용되어 있다. 

### 2.5 ORDER_BTY 절에서 칼럼 가공 

조건절이 아닌 ORDER_BY 절이나 SELECT-LIST 에서 칼럼을 가공한다면 정렬 연산을 생략하는게 아니라 추가적으로 할 수도 있다. 

인덱스가 장비번호 + 변경일자 + 변경순번 으로 구성되어있다고 가정해보자. 

다음과 같은 SQL 문은 ORDER_BY 연산을 생략할 수 있다. 

```sql
SELECT * 
FROM 상태변경이력
WHERE 장비번호 = 'C'
ORDER BY 변경일자, 변경순번
``` 

하지만 다음과 같은 SQL 문은 ORDER_BY 연산을 생략할 수 없다.

```sql
SELECT * 
FROM 상태변경이력
WHERE 장비번호 = 'C'
ORDER BY 변경일자 || 변경순번
``` 

ORDER BY 절 칼럼을 인덱스 그대로 사용하는게 아니기 떄문에 추가적인 연산이 필요하기 때문이다. 

### 2.6 SELECT LIST 에서 칼럼 가공

인덱스가 장비번호 + 변경일자 + 변경순번으로 이뤄진 경우에 아래와 같은 연산들은 ORDER_BY 연산을 따로 수행하지 않는다. 

```sql
SELECT MIN (변경순번)
FROM 상태변경이력
WHERE 장비번호 = 'C'
AND 변경일자 = '20180318'
```

왜냐히면 인덱스의 선행 조건들 장비번호 와 변경일자를 통해 수직적 스캔을 하면 변경 순번이 가장 작은 값이 나올 거기 때문이다. 

반대로 변경순번의 최대 값을 구하는 경우에도 수직적 탐색의 방향을 반대쪽으로 잡는다면 변경 순번이 가장 큰 값이 나오게 할 수 있기 때문에 따른 정렬 연산이 필요 없다. 

좀 더 예제를 보자면 다음과 같은 SQL 문은 정렬 연산을 생략할 수 없다. 

```sql
SELECT NVL(MAX (TO_NUMBER(변경 순번)), 0)
FROM 상태변경이력
WHERE 장비번호 = 'C'
AND 변경일자 = '20180316'
``` 

변경 순번의 값을 변형해서 비교해야 하기 때문에 모든 값을 가져와서 정렬을 해야할 수 밖에 없다. 

하지만 다음 SQL 문은 정렬 연산을 생략할 수 있다. 

```sql
SELECT NVL(TO_NUMBER( MAX (변경 순번)), 0)
FROM 상태변경이력
WHERE 장비번호 = 'C'
AND 변경일자 = '20180316'
```

### 2.7 자동 형변환

고객 테이블에 생년월일이 선두 칼럼인 경우가 있다고 생각해보자. 

아래 SQL 문은 인덱스 Range Scan 을 하지 않는다. 테이블 풀 스캔을 했다. 

```sql
SELECT *
FROM 고객
WHERE 생년월일 = 19821225
```

왜냐하면 옵티마이저가 19821225 가 숫자라서 문자 타입인 생년월일을 숫자형으로 칼럼을 자동으로 형변환 했기 때문에 즉 가공했기 때문에 인덱스가 사용되지 않았다. 

숫자형과 문자형이 만나면 숫자형이 이기므로 문자형을 숫자형으로 바꾼다. 

이처럼 날짜형과 문자형이 만나면 날짜형이 이기므로 문자형을 날짜형으로 바꾼다. 

이처럼 형변환을 잘 이해해야 한다.

연산자가 LIKE 일 경우에는 또 다른데 LIKE 자체가 문자열 비교이므로 문자열 기준으로 숫자형 칼럼이 변환된다. 

즉 인덱스 칼럼이 숫자인데 LIKE 연산으로 걸어놨다면 자동으로 문자열로 변환된다는 점을 알고 있어야 한다. 

이처럼 데이터베이스 내 함수들 중에서 자동으로 형변환을 해주는 함수들이 있을 수 있으니까 주의하자. 

***

## 3. 인덱스 확장기능 사용법

지금까지는 Index Range Scan 을 위주로 기본 사용법을 알았다면 이제는 인덱스의 다양한 스캔의 활용방법을 알아보자.

인덱스의 스캔 방식은 탐색 방법에 따라 Index Full Scan, Index Unique Scan, Index Skip Scan, Index Fast Full Scan 등이 있다.

### 3.1 Index Full Scan

Index Full Scan 은 인덱스의 수직적 탐색 없이 인덱스의 리프 블록을 모두 탐색하는 것을 말한다. 

Index Full Scan 은 데이터 검색을 위한 최적읜 인덱스가 없을 때, 인덱스싀 선행조건을 사용할 수는 없지만 인덱스 블록을 통한 검색이 가능할 때 차선으로 선택되는 방식이다.

#### Index Full Scan 의 효용성

SQL 에서 인덱스의 선두 칼럼을 사용하지 않는다면 옵티마이저는 먼저 Table Full Scan 을 고려한다. 

하지만 대용량 테이블이고 소량의 데이터만 엑세스 할 것 같다고 생각하면 옵티마이저는 Index Full Scan 도 고려한다. 

그치만 이 방식은 어디까지나 Index Range Scan 을 사용하는 것보다 효과가 없다. 수행 빈도가 낮은 SQL 문이라서 딱히 Index 를 걸 필요가 없다면 상관없겠지만 

수행빈도가 높다면 Index 를 적절하게 걸어서 Range Scan 을 하도록 하는게 좋다.

### 3.2 Index Unique Scan

Index Unique Scan 은 인덱스의 수직적 탐색으로만 데이터를 찾는 스캔이다. 

주로 조건절에서 '=' 을 이용한 검색을 할 때 또는 Unique 칼럼을 찾을 때 사용하는 스캔 방식이다.  

Unique 인덱스라고 해도 범위 조건으로 (between, like) 검색할 때는 Index Range Scan 이 사용되기도 한다. 

### 3.3 Index Skip Scan

인덱스 선두 칼럼을 조건절에 사용하지 않으면 기본적으로 Table Full Scan 이 발생하고 상황이 맞으면 Index Full Scan 을 하기도 한다고 말했다.

하지만 이 보다 조금 더 나은 방식을 제공해주기도 하는데 그게 바로 Index Skip Scan 이다.

Index Skip Scan 은 탐색할 가능성이 있는 블록을 찾는다고 생각하면 된다. 

주로 Index Skip Scan 이 유효한 경우에는 선두 칼럼이 없지만 선두 칼럼의 Distinct Value 개수가 적지만 후행 칼럼의 Distinct Value 개수가 많다면 유용하다. 

### 3.4 Index Fast Full Scan 

Index Fast Full Scan 은 말 그대로 Index Full Scan 보다 빠르다.   

빠른 이유로는 논리적인 인덱스 리프 블록을 가지고 오기 보다는 물리적으로 디스크에 저장된 순서대로 즉 인덱스 세그먼트 전체를 Multiblock I/O 를 기반으로 가지고 오기 때문에 빠르다.

이렇게 빠르기 때문에 좋은 점도 있지만 물리적으로 연결된 순서대로 가지고 오기 때문에 값들이 정렬되어 있지는 않댜.

  