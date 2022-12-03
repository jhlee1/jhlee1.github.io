---
title: "JDBC"
date: 2019-02-25T01:05:17+09:00
archives: "2019"
tags: ["JDBC", "edwith"]
author: Joohan Lee
---

## JDBC

### 1. 개념

- Java를 이용한 Database 접속과 SQL 실행, 실행 결과로 얻어진 Data의 핸들링을 제공하는 방법과 절차에 관한 규약
- Java 프로그램 내에서 SQL문을 실행하기 위한 Java API
- SQL과 Programming 언어의 통합 접근 중 한 형태
- Java는 표준 인터페이스인 JDBC API를 제공
- Database vendor 또는 기타 Third Party에서는 JDBC 인터페이스를 구현한 Driver를 제공

### 2. JDBC 드라이버 설치

- Maven에 Dependency 추가

```
<depencency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.45</version>
</dependency>
```



### 3. JDBC 사용법

1. Import

   - import java.sql.*;

2. Driver를 로드

   - Class.forName("com.mysql.jdbc.Driver"); // 어떤 vendor를 이용하느냐에 따라 String 값이 달라짐

3. Connection 객체 생성

   - String dburl = "jdbc:mysql://localhost/dbName";

   - Connection conn = DriverManager.getConnection(dburl, ID, PWD);

   - ex)

     ```
     public static Connection getConnection() throws Exception {
     	String url = "jdbc:oracle:thin:@117.16.46.111:1521:xe";
     	String user = "smu";
     	String password = "smu";
     	Connection conn = null;

     	Class.forName("oracle.jdbc.driver.OracleDriver");
     	conn = DriverManager.getConnection(url, user, password);

     	return conn;
     }
     ```

4. Statement 객체를 생성 및 실행

   - Statement stmt = con.createStatement();
   - ResultSet rs = stmt.executeQuery("select no from user");
   - smtm.execute("query"); // any SQL
   - smtm.execute("query");  // SELECT
   - smtm.executeUpdate("query");  // INSERT, UPDATE, DELETE

5. SQL의 결과가 있다면 ResultSet 객체를 생성

   - ResultSet rs = stmt.executeQuery("select no from user");
     while (rs.next()) {
     	System.out.println(rs.getInt("no"));

     }

6. 모든 객체를 닫는다.

   - rs.close();
   - stmt.close();
   - conn.close();

### 4. JDBC 클래스의 생성관계

- DriverManager를 이용해서 Connection 인스턴스를 얻는다
- Connection을 통해서 Statement를 얻는다
- Statement를 이용해 ResultSet을 얻는다
- DriverManager -> Connection -> Statement -> ResultSet
- 닫을 땐 열은 순서의 반대로 닫는다.

### 5. Example

```
public List<GuestBookVO> getGuestBookList(){
		List<GuestBookVO> list = new ArrayList<>();
		GuestBookVO vo = null;
		Connection conn = null;
		PreparedStatement ps = null;
		ResultSet rs = null;
		try{
			conn = DBUtil.getConnection();
			String sql = "select * from guestbook";
			ps = conn.prepareStatement(sql);
			rs = ps.executeQuery();
			while(rs.next()){
				vo = new GuestBookVO();
				vo.setNo(rs.getInt(1));
				vo.setId(rs.getString(2));
				vo.setTitle(rs.getString(3));
				vo.setConetnt(rs.getString(4));
				vo.setRegDate(rs.getString(5));
				list.add(vo);
			}
		}catch(Exception e){
			e.printStackTrace();
		}finally {
			DBUtil.close(conn, ps, rs);
		}		
		return list;		
	}
```

```
public int addGuestBook(GuestBookVO vo){
		int result = 0;
		Connection conn = null;
		PreparedStatement ps = null;
		try{
			conn = DBUtil.getConnection();
			String sql = "insert into guestbook values("
					+ "guestbook_seq.nextval,?,?,?,sysdate)";
			ps = conn.prepareStatement(sql);
			ps.setString(1, vo.getId());
			ps.setString(2, vo.getTitle());
			ps.setString(3, vo.getConetnt());
			result = ps.executeUpdate();
		}catch(Exception e){
			e.printStackTrace();
		}finally {
			DBUtil.close(conn, ps);
		}

		return result;
	}
```

```
public static void close(Connection conn, PreparedStatement ps){
		if (ps != null) {
			try {
				ps.close();
			} catch (SQLException e) {e.printStackTrace(); }
		}
		if (conn != null) {
			try {
				conn.close();
			} catch (SQLException e) {e.printStackTrace();}
		}
	}
```

원본 강의: https://www.edwith.org/boostcourse-web
