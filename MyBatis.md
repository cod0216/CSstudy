# 00. 개요

## MyBatis란?
- MyBatis는 Java Object와 SQL문 사이의 자동 Mapping 기능을 지원하는 ORM(Object Relational Mapping) FrameWork
	- MyBatis는 SQL을 별도의 파일로 분리해서 관리
	- Object - SQL 사이의 parameter Mapping 작업을 자동으로 해 줌.
	- MyBatis는 Hibernate나 JPA(Java Persistence API)처럼 새로운 DB 프로그래밍 패더라다임을 부담이 없이 개발자가 익숙한 SQL을 그대로 이용하면서 JDBC 코드 작성의 불편함을 제거해 주고 도매인 객체나 VO 객체를 중심으로 개발이 가능

# 01. 특징
##  MyBatis 특징
- 쉬운 접근성과 코드의 **간결함**
	- 가장 간단한 Persistence framework
	- XML 형태로 서술된 JDBC 코드라 생각해도 될 만큼 J**DBC의 모든 기능을 MyBatis가 제공**
	- 복잡한 JDBC 코드를 걷어내며 깔끔한 소스코드를 유지.
	- 수동적인 parameter 설정과 Query 결과에 대한 Mapping 구문을 제거
- SQL문과 프로그래밍 코드의 분리로 **이식성**이 좋음
	- SQL에 변경이 있을 때마다 자바 코드를 수정하거나 컴파일 하지 않아도 됨
	- **SQL을 별도의 파일로 분리해서 관리**
- **다양한 프로그래밍 언어로 구현이 가능**
	- Java, C#, .NET, Ruby

# 02. 흐름
## 게시판 리스트 조회

![image](https://github.com/user-attachments/assets/3cfc3f75-d6ac-4cea-8375-5226c74675b9)


1. **Service계층 (비즈니스 로직)**
```java
@Service
public class BoardService {
	private final BoardDao boardDao;
	public List<Board> listBoards(int page, int size){
		int offset = (page -1) * size;
		List<Board> boards = boardDao.selectBoards(offset, size);
	}
}
```
2. **DAO 인터페이스 (MyBatis 매퍼)**
	- 해당 인터페이스는 MyBatis-Spring이 프록시 구현체를 자동으로 생성함
	- SQL은 XML 매퍼 파일 혹은 애노테이션에 정의
```java
@Mapper
publis interface BoardDao {
	List<Board> selectBoards(@Param("offset") int offset, 
							 @Param("size") int size);
}


// resiouces/mappers/BoardMapper.xml
<select id "selectBoards" resultType="Entity 경로 혹은 DTO 경로">
	SELET id, title, content, author_id, create_at
	FROM board
	ORDER BY create_at DESC
	LIMIT #{size} OFFSET #{offset}
</select>
```   
3. **MyBatis(O/R Mapper)**
	- `SqlSessionFactoryBean`이 `DataSource` 설정으로부터 `SqlSessionFactory`를 생성
	- **`@Mapper`스캔을 통해 `BoardDao` 프록시가 등록**
	- `BoardDao.selectBoards(...)` 호출 시 내부적으로는 
		1. `SqlSession`을 열어 `Connection`을 가져오고
		2. 매핑된 SQL(selectBoards)을 찾은 뒤 파라미터 바인딩
		3. JDBC API(`PreparedStatement`)로 SQL 실행
		4. ResultSet -> Board 객체 리스트 매핑
4. **JDBC Interface & DataSource**
	- Spring에 등록된 `HikariDataSource`에서 미리 준비된 커넥션을 꺼냄
	- **`Connection.prepareStatement()` -> `PreparedStatement` 생성**
```java
ps.setInt(1, size);
ps.setInt(2, offset);
ps.executeQuery();
```
4. **JDBC Driver**
	- 드라이버가 실제 네이트브 프로토콜(TCP 패킷)로 SQL을 DB에 전송
	- 이후 DB가 응답한 결과를 드라이버가 `ResultSet`형태로 변환
5. **Persistence Layer**
	- 트랜잭션, 인덱스 스캔, I/O 등 내부 처리


## 03. MyBatis와 JPA의 차이

### 장단점
|        | MyBatis                                                                                                              | JPA                                                                                                                                                                                                        |
| ------ | -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **장점** | - **SQL 쿼리를 직접 작성할 수 있음**<br>    SQL의 강력한 기능을 그대로 활용할 수 있음<br>- **학습 곡선이 낮음**                                        | - **DB 테이블 간 매핑이 자동으로 처리됨**<br>    SQL 쿼리를 직접 작성하지 않아도 됨<br>- **DB 독립성을 제공함**<br>    JPA는 특정 DB에 종속되지 않으며 다양한 DB를 지원함<br>- **객체 지향 프로그래밍의 장점을 최대한 활용 가능**<br>    객체와 테이블 간의 매핑을 통해 객체 지향적이 설계를 유지할 수 있기 때문 |
| **단점** | - **데이터 베이스 종속적**<br>    특정 DB에 맞춘 SQL쿼리를 작성해야 함<br>- **반복적인 CRUD 코드를 많이 작성해야 하는 번거로움이 존재**<br>    쿼리를 직접 작성해야 하기 때문 | - **러닝 커브가 높음**<br>    다양한 기능과 설정을 이해하고 활용하는데 시간이 소요됨<br>- **복잡한 쿼리를 작성하는데 어려움이 있음**<br>    SQL 쿼리를 자동으로 생성하기 때문에, 특정한 쿼리 최적화가 필요 시 직접 SQL을 작성해야됨                                                          |

### id를 통한 단일 엔티티 조회
- **MyBatis**
```java
public interface UserMapper{
	@Select("SELECT * FROM users WHERE id = #{id}")
	User findById(int id)
}
```
- **JPA**
```java
public interface UserRepository extends JpaRepository<User, Integer> {
	Optional<User> findById(Integer id);
}
```
### 엔티티 컬렉션 조회
- **MyBatis**
```java
public interface UserMapper{
	@Select("Select * FROM users")
	List<User> findAll();
}
```
- **JPA**
```java
public interface UserRepository extends JpaRepository<User, Integer> {
	List<User> findAll()
}
```
### 엔티티 추가
- **MyBatis**
```java
public interface UserMapper {
	@Integer("INSERT INTO users (name, email) VALUES (#{name}, #{email})")
	@Options(userGeneratedKey = true, keyProperty ="id")
	int insert(User user);
}
```
- **JPA**
```java
public interface userRepository extends JpaRepository<User, Integer>{
	User save(User user)
}
```
### 엔티티 업데이트
- **MyBatis**
```java
public interface UserMapper {
	@Update("UPDATE users SET name = #{name}, email = #{email} WHERE id = #{id}")
	int update(User user);
}
```
- **JPA**
```java
public interface UserRepository extends JpaRepository<User, Integer>{
	Save save(User user);
}
```
### id를 이용해서 엔티티 삭제
- **MyBatis**
```java
public interface UserMapper {
	@Delete("DELETE FROM users WHERE id = #{id}")
	int deleteById(int id);
}
```
- **JPA**
```java
public interface UserRepository extends JpaRepositody<user, Integer>{
	void deleteById(Integer id);
}
```

