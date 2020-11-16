# Week 1

## I. Set up Atlas

- https://cloud.mongodb.com/ 
  - 500MB짜리 Cluster를 무료로 제공해줌
  - 회원가입하고 Freetier로 프로젝트 생성
- 접속방법은 해당 프로젝트 리스트 화면에서 connect 버튼 누르면 자세히 알려줌
- **사용 전 아래 두가지는 필수**
  - Security Setting에서 IP whitelist에 내 IP 추가하거나 모든 IP에 대해 접속허용 설정
  - Database user 생성

## II. 예제 파일 DB에 import

- mongoimport를 이용해서 csv파일을 DB에 저장
  - 최신 버전에서는 uri에 username, password, database name을 다 적어줘야함
  - movies_initial.csv파일을 movies_initial이란 collection에 넣어줌

```bash
$ mongoimport --type csv --headerline --collection movies_initial --uri "mongodb+srv://[DB_USER]:[DB_USER_PASSWORD]@springbootexample-nysvl.mongodb.net/[DATABASE_NAME]?ssl=true&authSource=admin" --file movies_initial.csv
```

## III. Compass와 Atlas 연결

- MongoDB용 GUI 툴
- 연결 방법
  - Atlas에서 해당 프로젝트 리스트 화면에서 connect 버튼 누르고 connect to compass누르면 uri를 복사하고 compass로 넘어오면 자동 붙여넣기처리할 수 있다
  - username하고 password만 수정해주면됨

## IV. Aggregation Framework

- Pipeline 형식으로 이루어져있음
- 중간에 여러개의 Stage를 통해서 data를 가공해서 최종 결과 값을 얻어낼 수 있음

![image-20191008185142106](/Users/joohanlee/Library/Application Support/typora-user-images/image-20191008185142106.png)

- $group
  - https://docs.mongodb.com/manual/reference/operator/aggregation/group/index.html?jmp=coursera-intro-mongodb#accumulator-operator

```
pipeline = [
	{
		'$group': {
			'_id': {"language" : "$language"},
			'count': {'$sum': 1}
		}
	}
]
```

