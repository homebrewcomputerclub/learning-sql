# Learning SQL #4,5,6
# 4. 필터링
SQL 문에는 SQL 문으로 수행되는 행 수를 제한하기 위해 하나 이상의 필터조건을 포함하는 `where` 절이 선택적으로 포함된다. 또한 `select` 문에는 그룹화된 데이터의 필터 조건을 포함하는 `having` 절이 포함된다.(103p)

## 조건 평가
`where` 절은 `and` 또는 `or` 연산자로 구분된 하나 이상의 조건을 포함한다.

`not` 연산자는 `<>` 로 바꿔 사용할 수도 있다. `<>` 를 사용하는 것이 이해하는 게 더 쉽다고 설명하는데, 개인적으로는 `not` 이 더 이해하기 쉽다.

## 조건 작성
조건은 하나 이상의 연산자와 하나 이상의 표현식으로 구성된다. 표현식은 다음 중 하나일 수 있다.
	- 숫자
	- 테이블 또는 뷰의 열
	- `Maple Street` 과 같은 문자열
	- `concat` 과 같은 내장 함수
	- 서브 쿼리
	- ('Boston', 'New york') 등과 같은 표현식 목록

조건 내에서 사용되는 연산자는
	- =, !=, <, >, <>, in, like, between과 같은 비교 연산자
	- +,-,*과 같은 산술 연산자

## 조건 유형
### 동등 조건
`id = 21` 와 같은 형식을 뜻함
### 부등 조건
부등 조건inequality은 표현식이 동일하지 않을 때 사용하는 조건을 뜻한다.
즉 `<>` (`not`) 을 사용하는 조건을 뜻한다.
### 범위 조건
범위조건range condition은 특정 범위 내에 있는지를 확인하는 조건이다. 
`rental_data < '2005-05-25` 
`amount between 10.0 AND 11.99` 
### 멤버십조건
`in` 연산자로 여러 항목이 포함된 목록을 검색할 수 있다.
`in ('G', 'PG')`
반대로 `not in` 조건을 사용할 수도 있다.
### 서브쿼리 사용
`in` 절에 들어갈 조건으로 서브쿼리 즉시 집합을 생성해서 넣을 수도 있다.
`in (select rating FROM film WHERE title LIKE '%PET%')` 
### 일치조건
일치조건matching conditions은 equals condition과는 좀 다름. 동등 조건이 같은 것을 찾는다면 일치 조건은 `like` 연산을 뜻함. 문자열로 하면 전체 문자열 중 조건으로 사용된 문자열이 포함된 데이터가 있는지 검색하는 것.
`like '%kb%'` 라고 하면 테이블 컬럼 문자열 중 kb가 포함된 모든 row를 반환한다.

위에서 사용한 `%` 는 아일드카드라고 함. 문자열 일치를 식별하는 검색 표현식을 쓸 때 사용한다. `%` 는 앞 뒤에 붙일 수 있고 문자열이 몇 글자여도 상관없지만 `_`  언더스코어를 사용한다면 반드시 한 글자여야 한다.
`LIKE '_A_T%S'`  A 앞 뒤에는 언더스코어를 사용했기 때문에 반드시 한 글자여야 하지만, T 뒤에는 `%` 이기 때문에 몇 글자가 와도 상관없다.
언더스코어를 붙여서 여러 번 사용하는 것도 가능하다.

## Null
null은 여러 정의가 가능하다.
- 해당사항 없음
- 아직 알려지지 않은 값
- 정의되지 않은 값
null을 작업할 때 주의해야 할 사항은
- null일 수는 있지만 null과 같을 수는 없다.
- 두 개의 null은 같지 않다.
값이 null인지 확인하려면 `=` 이 아니라 `is (not) null` 연산을 사용해야 한다.

## 4장 연습문제
- 다음 필터 조건에 따라 반환되는 payment ID는 무엇인가?
```
customer_id <> 5 AND (amount > 8 OR date(payment_date_ = '2005-08-23')
```
id가 5가 아니면서 amount가 8보다 크거나 (또는) payment_date가 2005-08-23인 모든 row를 반환한다.
따라서 답은 101, 107.

- 다음 필터 조건에 따라 반환되는 payment ID는 무엇인가?
```
customer_id = 5 AND NOT (amount > 6 OR date(payment_date) = '2005-06-19')
```
c_id가 5면서, amount가 6보다 크거나, 또는 payment_date가 2005-06-19가 **아닌** 모든 row.
답: 많아서 생략.

- payments 테이블에서 금액이 1.98, 7.98 또는 9.98인 모든 행을 검색하는 쿼리를 작성해라.
```sql
SELECT *
FROM payments p
WHERE p.amount in (1.98, 7.98, 9.98);
```

- 성(last_name)의 두 번째 위치에 A가 있고, A 다음에 W가 있는 모든 고객을 찾는 쿼리 작성
```sql
SELECT *
FROM payments p
WHERE p.last_name LIKE '_A%W%';
```


# 5. 다중 테이블 쿼리
조인은 두 개 이상의 테이블의 데이터를 함께 가져오는 매커니즘이다. 
이 장에서는 가장 단순한 inner join만 살펴봄.

## 조인
조인은 여러 테이블의 데이터를 함께 가져오는 것이라고 설명했다. 이때 두 테이블을 잇는 고리가 외래키(foreign key, FK)다. 이 외래 키를 연결 수단으로 사용하여 두 테이블의 열을 하나의 결과셋에 포함하는 것이 바로 조인이다.

## 데카르트 곱
데카르트 곱은 그냥 join 키워드를 사용해서 두 테이블 데이터를 모두 가져오는 것이다.
```sql
SELECT *
FROM customer c JOIN address a;
```
address 테이블에 599명의 고객와 603개의 행만 있는데 결과는 36197개가 나온다.
두 테이블을 어떻게 조인해야 할지 지정하지 않으면 데이터베이스 서버는 데카르트 곱cartesian product인 두 테이블의 모든 순열(599 X 603)을 생성한다. 이런 ㅇ ㅠ형의 조인을 교차 조인cross join이라고 하지만 보다시ㅣ피 실수가 아니라면 사용할 일이 거의 없다.

## 내부조인
크로스 조인이 아니라, 각 고객에 대한 단일 행, 즉, 고객과 주소의 쌍으로 유의미한 행을 도출해내려면 두 테이블이 어떤 관계인지 확인해야 한다.
customer는 고객의 주소 데이터를 보관하는 테이블의 PK인 address_id를 외래 키로 가지고 있다.
이것을 FROM 하위절인  `ON` 절에 명시해야 한다.
```sql
SELECT *
FROM customer c INNER JOIN address a
	ON c.address_id = a.address_id;
# 조인 시 사용하는 컬럼 이름이 동일한 경우 on 대신 using 사용 가능
# USING (address_id);
# 하지만 혼동을 피하기 위해 항상 on을 사용하는 게 낫다.
```
만약 c.address_id에 대응하는 a.address_id가 존재하지 않는다면 조인은 실패하고 결과셋에서 제외된다. address가 null이어도 출력하고 싶다면 inner join이 아닌 outer join을 사용한다.

## ANSI JOIN 문법
위에서 소개한 JOIN 문법은 ANSI SQL 표준인 SQL92 버전에 따른 것임. 하지만 데이터베이스 서버 대부분 SQL92 스펙이 나오기 전부터 사용되었으므로 이전 조인 문법도 이해한다.
```sql
...
FROM customer c, address a
WHERE c.address_id = a.address_id
```
예전에는 `on` 하위절을 사용하지 않고 FROM에 조인 테이블을 콤마로 구분하고 WHERE 절에서 조인 조건을 포함했다. 이처럼 SQL92 구문을 무시해도 상관없지만 ANSI JOIN 문법은 몇 가지 이점을 제공한다.
- 조인 조건과 필터 조건은 이해하기 쉽게 두 개의 각각 다른 절(on 하위절과 where절)로 구분된다.
- 각 테이블 쌍에 대한 조인 조건이 on 절에 포함되어 있으므로 조인 조건의 일부가 실수로 누락될 가능성이 낮다.
- SQL92 조인 문법을 사용하는 쿼리는 표준화되어 있으므로 데이터베이스 서버 간 이식이 가능하다.
SQL92 구문 이점은 조인 및 필터조건을 모두 포함하는 복잡한 쿼리를 좀더 쉽게 식별할 수 있다는 것이다.

## 세 개 이상 테이블 조인
세 개 이상 테이블을 조인할 때도 두 개 테이블 조인할 때와 마찬가지다.
```sql
SELECT *
FROM customer c INNER JOIN address a
	ON c.address_id = a.address_id
	INNER JOIN city ct
	ON a.city_id = ct.id;
```
참고로 FROM 절에 테이블을 나열하는 순서는 중요하지 않다. 쿼리를 가장 잘 실행하는 방법을 결정하는 것은 데이터베이스 서버다.

## 서브쿼리 조인
```sql
SELECT *
FROM customer c
	INNER JOIN
		(SELECT a.address_id, a.address, ct.city
			FROM address a
				INNER JOIN city ct
				ON a.city_id = ct.city_id
			WHERE a.district = 'California') addr
	ON c.address.id = addr.address_id;
```

## 테이블 재사용
여러 테이블 조인 시 같은 테이블을 두 번 이상 조인해야 할 수도 있음.
샘플 DB에서 배우는 `film_actor` 테이블에 있는 영화와 관련이 있다. 두 명의 특정 배우가 출연한 영화 제목을 모두 찾으려면 다음과 같은 쿼리를 `film` 테이블과 `film_actor` 테이블, 그리고 `actor` 테이블을 조인할 수 있다.

```sql
SELECT f.title 
FROM film f  
	INNER JOIN film_actor fa  
	ON f.film_id = fa.film_id  
	INNER JOIN actor a  
	ON fa.actor_id = a.actor_id 
WHERE ((a.first_name='CATE' AND a.last_name ='MCQUEEN')  OR (a.first_name = 'CUBA' AND a.last_name = 'BIRCH'));

+----------------------+
| title                |
+----------------------+
| ATLANTIS CAUSE       |
| BLOOD ARGONAUTS      |
| COMMANDMENTS EXPRESS |
| DYNAMITE TARZAN      |
| EDGE KISSING         |
| FINDING ANACONDA     |
| GREATEST NORTH       |
| JUNGLE CLOSER        |
| LANGUAGE COWBOY      |
| LEAGUE HELLFIGHTERS  |
| LIBERTY MAGNIFICENT  |
| LOST BIRD            |
| MAGNIFICENT CHITTY   |
| MARS ROMAN           |
| NORTHWEST POLISH     |
| ROAD ROXANNE         |
| RUGRATS SHAKESPEARE  |
| SHIP WONDERLAND      |
| SONS INTERVIEW       |
| STRANGER STRANGERS   |
| TENENBAUMS COMMAND   |
| TOOTSIE PILOT        |
| TOWERS HURRICANE     |
| VICTORY ACADEMY      |
| ANNIE IDENTITY       |
| BLOOD ARGONAUTS      |
| CAPER MOTIONS        |
| COMANCHEROS ENEMY    |
| DARN FORRESTER       |
| DOCTOR GRAIL         |
| FACTORY DRAGON       |
| FIDDLER LOST         |
| FLYING HOOK          |
| FRENCH HOLIDAY       |
| GABLES METROPOLIS    |
| HAWK CHILL           |
| INSIDER ARIZONA      |
| JERSEY SASSY         |
| LEGEND JEDI          |
| MASSAGE IMAGE        |
| NAME DETECTIVE       |
| PACIFIC AMISTAD      |
| PATTON INTERVIEW     |
| PERDITION FARGO      |
| POTTER CONNECTICUT   |
| PRIDE ALAMO          |
| SALUTE APOLLO        |
| SEARCHERS WAIT       |
| SNATCH SLIPPER       |
| TOWERS HURRICANE     |
| TROJAN TOMORROW      |
| VIRGIN DAISY         |
| VOLCANO TEXAS        |
| WATERSHIP FRONTIER   |
+----------------------+
54 rows in set (0.00 sec)

```

이 쿼리는 Cate McQueen 또는 Cuba Birch가 출연한 모든 영화를 반환한다.
이때 두 배우가 함께 출연한 영화만 검색하려면, `film` 테이블에서 `film_actor` 테이블에 두 행이 있는 모든 행을 찾아야 한다. 그 중 하나는 케이트 맥퀸과 관련된 것이고, 하나는 쿠바와 관련된 것이다. 따라서 서버가 다른 절에서 참조하는 항목을 알 수 있도록, 각각 다른 별칭을 가진 `film_actor` 및 `actor` 테이블을 두 번 포함해야 한다.

```sql
SELECT f.title 
FROM film f  
	INNER JOIN film_actor fa1  ON f.film_id = fa1.film_id  
	INNER JOIN actor a1  ON fa1.actor_id = a1.actor_id  
	INNER JOIN film_actor fa2   ON f.film_id = fa2.film_id  
	INNER JOIN actor a2   ON fa2.actor_id = a2.actor_id 
WHERE (a1.first_name = 'CATE' AND a1.last_name= 'MCQUEEN')  AND (a2.first_name = 'CUBA' AND a2.last_name = 'BIRCH');
#
+------------------+
| title            |
+------------------+
| BLOOD ARGONAUTS  |
| TOWERS HURRICANE |
+------------------+
2 rows in set (0.00 sec)

```


## self join
자기 자신과도 조인할 수 있다. 이것을 셀프 조인이라고 한다. 일부 테이블에는 자기 참조 외래 키self-referencing foreign key가 포함된다. 즉, 동일한 테이블 내에 기본 키를 가리키는 하나의 열이 포함되어 있다. 
예를 들어 영화 시리즈일 경우 `film` 테이블에 해당 영화의 전편을 나타내는 `prequel_film_id` 컬럼이 포함되어 있을 수도 있다.
셀프 조인을 이용해서 전편 제목과 더불어 전편이 있는 모든 영화 제목을 나열하는 쿼리를 다음처럼 쓸 수 있다.

```sql
SELECT f.title, f_prnt.title prequel
FROM film f
	INNER JOIN film f_prnt
	ON f_prnt.film_id = f.prequel_film_id
WHERE f.prequel_film_id IS NOT NULL;
```
이 쿼리는 `prequel_film_id` 외래 키를 사용해서 `film` 테이블을 셀프 조인한다.

## 5장 학습 점검
### 연습문제1
- 다음과 같은 결과를 얻기 위해 다음 쿼리의 <#> 부분을 채우시오.
```sql
+------------+-----------+------------------------+----------------+
| first_name | last_name | address                | city           |
+------------+-----------+------------------------+----------------+
| PATRICIA   | JOHNSON   | 1121 Loja Avenue       | San Bernardino |
| BETTY      | WHITE     | 770 Bydgoszcz Avenue   | Citrus Heights |
| ALICE      | STEWART   | 1135 Izumisano Parkway | Fontana        |
| ROSA       | REYNOLDS  | 793 Cam Ranh Avenue    | Lancaster      |
| RENEE      | LANE      | 533 al-Ayn Boulevard   | Compton        |
| KRISTIN    | JOHNSTON  | 226 Brest Manor        | Sunnyvale      |
| CASSANDRA  | WALTERS   | 920 Kumbakonam Loop    | Salinas        |
| JACOB      | LANCE     | 1866 al-Qatif Avenue   | El Monte       |
| RENE       | MCALISTER | 1895 Zhezqazghan Drive | Garden Grove   |
+------------+-----------+------------------------+----------------+

SELECT c.first_name, c.last_name, a.address, ct.city
FROM customer c
	INNER JOIN address <#1>
	ON c.address_id = a.address_id
	INNER JOIN city ct
	ON a.city_id = <#2>
WHERE a.district = 'California';
```
<#1>: a
<#2>: ct.city_id

### 연습문제2
- John이라는 이름의 배우가 출연한 모든 영화 제목을 반환하는 쿼리를 작성
```sql
SELECT f.title
FROM film f
	INNER JOIN film_actor fa
		ON f.film_id = fa.film_id
	INNER JOIN actor a
		ON a.actor_id = fa.actor_id
WHERE a.first_name = 'John';

+---------------------------+
| title                     |
+---------------------------+
| ALLEY EVOLUTION           |
| BEVERLY OUTLAW            |
| CANDLES GRAPES            |
| CLEOPATRA DEVIL           |
| COLOR PHILADELPHIA        |
| CONQUERER NUTS            |
| DAUGHTER MADIGAN          |
| GLEAMING JAWBREAKER       |
| GOLDMINE TYCOON           |
| HOME PITY                 |
| INTERVIEW LIAISONS        |
| ISHTAR ROCKETEER          |
| JAPANESE RUN              |
| JERSEY SASSY              |
| LUKE MUMMY                |
| MILLION ACE               |
| MONSTER SPARTACUS         |
| NAME DETECTIVE            |
| NECKLACE OUTBREAK         |
| NEWSIES STORY             |
| PET HAUNTING              |
| PIANIST OUTFIELD          |
| PINOCCHIO SIMON           |
| PITTSBURGH HUNCHBACK      |
| QUILLS BULL               |
| RAGING AIRPLANE           |
| ROXANNE REBEL             |
| SATISFACTION CONFIDENTIAL |
| SONG HEDWIG               |
+---------------------------+
29 rows in set (0.00 sec)

```


### 연습문제 3 (??)
- 같은 도시에 있는 모든 주소를 반환하는 쿼리를 작성한다. 이때 `address` 테이블을 셀프 조인해야 하며, 각 행에는 두 개의 서로 다른 주소가 포함되어야 한다.

```sql
SELECT *
FROM address a
	INNER JOIN city ct
		ON a.city_id = ct.city_id
WHERE 
```


# 집합 연산자
집합 연산자에는 세 가지 종류가 있음.
- union(합집합)
- intersect(교집합)
- except(차집합)
이 중에 MySQL8.0이 지원하는 것은 union뿐임.
union연산은 `union` 키워드로 할 수 있고 `union all` 은 중복을 제거하지 않고 모든 결과를 반환하겠다는 뜻임.

#LearningSQL #SQL-filtering #SQL-Join
