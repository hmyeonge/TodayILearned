# 게시글 리스트 만들기

---

## 테스트 데이터 프로시저

```java
DELIMITER $$

CREATE PROCEDURE testDataInsert()
BEGIN
    DECLARE i INT DEFAULT 1;

    WHILE i <= 120 DO
        INSERT INTO board(title, content)
          VALUES(concat('제목',i), concat('내용',i));
        SET i = i + 1;
    END WHILE;
END$$
DELIMITER $$
```

위를 MySQL 워크벤치에 붙여넣고 ctrl + enter

[`BoardController.java`](http://BoardController.java) 코드 추가

```java
@GetMapping("/board/list")
    public String boardList() {

        return "boardlist";
    }
```

return 할 boardlist 를 생성

→ resources - templates - boardlist.html 파일 생성 (철자주의)

`boardlist.html` 파일에 코드 추가

<body> 태그 안에 layout 클래스와 table 을 생성

```java
<body>

    <div class = "layout">

        <table>
            <thead>
                <tr>
                    <th>글번호</th>
                    <th>제목</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>1</td>
                    <td>제목입니다</td>
                </tr>
            </tbody>
        </table>

    </div>

</body>
```

<head> 와 <body> 사이에 <style> 태그 생성

```css
<style>

    .layout{
        width : 500px;
        margin : 0 auto;
        margin-top : 50px;
    }

</style>
```

마지막에 [http://localhost:8080/board/list](http://localhost:8080/board/list) 링크에서 아래를 확인 가능함

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/94d6368e-ae1e-4639-9179-b054aba35ec1/53a42847-dcf3-49a6-bd19-fa4eaa3b1f50/Untitled.png)

# 스스로 정리

---

강의를 따라하던 중 프로젝트의 구조와 각 컴포넌트의 역할을 제대로 알아야 하겠다는 필요성을 느껴 정리해보았다.

## 프로젝트 구조를 바탕으로 컴포넌트의 역할 보기

> Controller는 외부 요청을 받아들이고, Service는 이 요청을 처리하는 비즈니스 로직을 담당하며, Repository는 데이터베이스와의 상호작용을 처리하고, Entity는 이 데이터의 구조를 정의한다

### BoardController

- 사용자의 웹 요청에 따라 적절한 서비스 로직을 호출하고, 사용자에게 보여줄 뷰 (Html 페이지) 를 결정하는 역할

**Autowired**

```java
@Autowired // 스프링이 자동으로 BoardService 타입에 맞는 빈(bean)을 찾아 boardService 참조변수를 할당함
    private BoardService boardService; // BoardService 타입의 객체를 사용하기 위한 참조변수를 선언함

```

- @Autowired 를 사용해 BoardService 객체를 자동으로 주입받는다. → BoardController 가 실제 게시글 처리 로직을 BoardService 를 통해 수행할 수 있게 해준다
    - 게시글 처리 로직 : 데이터베이스에 저장, 목록 조회

**게시글 작성폼 웹 페이지**

```java
@GetMapping("/board/write") 
    public String boardWriteForm(){

        return "boardwrite"; 
    }
```

- 사용자가 웹 브라우저를 통해 /board/write 주소로 접속할 때 boardWriteForm 메소드가 호출되어 boardwrite.html 를 반환한다
    - boardwrite.html : 게시글 작성을 위한 웹 페이지

**게시글 작성 처리**

```java
@PostMapping("/board/writepro") 
    public String boardWritePro(Board board) {
        
        boardService.write(board);

        return "";
    }
```

- 사용자가 게시글 작성 폼을 채우고 제출할 때 호출되는 boardWritePro 메소드 (제출을 눌렀을 때 주소가 /board/writepro 로 변한다)
- 사용자가 입력한 게시글 정보 (Board 객체) 를 받아 BoardService 의 write 메소드를 통해 데이터베이스에 저장
- 💡 매개변수 Board board 는, Board 클래스를 만들어 엔티티 형식 그대로 board 객체를 받아 boardSerivce.write(board) 메소드를 호출한다.

**게시글 목록 페이지**

```java
@GetMapping("/board/list")
    public String boardList() {

        return "boardlist";
    }
```

- 사용자가 /board/list 주소로 접속할 때 호출되는 boardList 메소드
    - GetMapping 어노테이션이 달린 boardList 메소드라고 부른다
- 저장된 게시글 목록을 보여주는 boardlist.html 을 사용자에게 반환

**@GetMapping 과 @PostMapping 의 비교**

- 둘다 Spring Framework의 어노테이션으로, 웹에서 클라이언트의 HTTP 요청을 특정 메소드에 매핑할 떄에 사용된다 (공통점)
- **@GetMapping** : HTTP GET 요청을 처리하는메소드에 사용
    - GET 요청 : 주로 데이터를 조회할 때 ( 웹페이지 요청, 검색 쿼리 전송) 사용된다
    - 예시 : 사용자가 브라우저에서 A 페이지를 열기 위해 URL 을 입력한다 → 서버는 해당 URL 에 매핑된 @GetMapping 어노테이션이 붙은 메소드를 실행해 요청된 A 페이지를 반환한다
- @PostMapping : HTTP POST 요청을 처리하는 메소드에 사용
    - POST 요청 : 주로 폼 데이터 제출이나 파일 업로드에 사용된다
    - 예시 : 사용자가 웹 폼에 데이터를 입력하고 제출 버튼을 클릭하면, 폼 데이터는 POST 요청을 통해 서버로 전송되고, 서버는 해당 데이터를 처리하기 위해 @PostMapping 어노테이션이 붙은 메소드를 실행한다

<aside> 💡 어노테이션을 사용함으로써 Spring MVC 에서는 클라이언트의 다양한 HTTP 요청을 적절한 메소드로 라우팅해 처리한다

</aside>

- 라우팅
    
    클라이언트로부터의 요청 (URL, HTTP 메소드) 을 특정 코드 (컨트롤러의 메소드) 에 매핑하는 과정
    
- Spring MVC
    
    Spring Framework 의 일부로, Java 기반의 웹 애플리케이션을 구축하이 위한 Model-View-Controller (MVC) 아키텍처를 구현한다.
    
    |MVC 아키텍처|애플리케이션을 3가지 주요 구성 요소로 분리하는 디자인 패턴|
    |---|---|
    |Model|데이터와 비즈니스 로직을 담당함|
    |애플리케이션의 데이터 구조를 정의하고, 데이터베이스와의 상호작용을 처리한다||
    |View|사용자 인터페이스를 담당함|
    |사용자에게 정보를 시각적으로 표현하는 역할을 한다||
    |Controller|사용자의 입력과 이벤트를 처리하고, Model 과 View 사이를|
    |연결한다||
    
    💡Spring MVC 에서 BoardController 은 Controller에 해당하고, BoardService와 BoardRepository, Entity의 Board 클래스는 Model 에 해당한다. 그 외 boardwrtie.html 과 같은 파일들이 View 에 해당한다
    

### BoardService

Controller 에서 받아들인 요청을 처리하는 비즈니스 로직을 담당한다

아래는 사용자가 게시판에 글을 쓸 때 , 그 글을 데이터베이스에 저장하는 로직을 담당하는 클래스를 정의한다

```java
@Service
public class BoardService {

    @Autowired
    private BoardRepository boardRepository;

    public void write(Board board) {
        boardRepository.save(board);
    }
}
```

- Autowired 어노테이션을 사용해 BoardRepository 의 인스턴스를 자동으로 주입받아, 바로 사용할 수 있다
- wirte 메소드 : 사용자로부터 입력받은 게시글 정보 (Board 객체) 를 save 메소드를 통해 데이터베이스에 저장한다

```java
 @PostMapping("/board/writepro") 
    public String boardWritePro(Board board) {

        boardService.write(board);

        return "";
    }
```

제출 버튼을 눌렀을 때, boardService.write() 메소드가 board 매개변수를 받아 호출된다. 그리고 [boardRepository.save](http://boardRepository.save)(board) 가 호출된다.

### BoardRepository

- **JpaRepository** (Spring Data JPA 모듈의 일부이고 데이터베이스에 접근하기 위한 표준 메소드들을 제공) 를 **확장하는 인터페이스**
    
    - JpaRepository
        
        스프링이 제공하는 인터페이스로, 기본적인 CRUD 기능과 페이징, 정렬 등을 위한 메소드를 이미 정의함
        
- BoardRepository 인터페이스를 정의함으로써 개발자는 데이터베이스와 상호작용하기 위해 필요한 메소드를 직접 구현하지 않아도 된다. 개발자는 인터페이스만 정의하면 된다. 그러면 데이터베이스와의 실제 상호작용은 스프링이 처리해준다
    
- 스프링은 런타임에 이 인터페이스의 구현체를 생성하고, 그 구현체를 사용해 실제 데이터베이스 작업을 수행한다
    
    - 레포지토리 인터페이스의 구현체로 동작하 프록시 객체를 자동으로 생성한다. 프록시 객체는 인터페이스가 정의한 모든 메소드 중 호출된 메소드에 대해 적절한 데이터베이스 연산을 수행한다.

### Entity 의 Board 클래스

- 데이터베이스의 board 테이블을 나타내는 **엔티티**
- 엔티티란?
    - 데이터베이스의 테이블 구조와 데이터를 객체지향적으로 표현한 자바 클래스
        
    - 테이블의 각 **레코드(테이블의 한 행)**는 엔티티 객체의 인스턴스로 매핑된다
        
        - 레코드 (record)
            
            레코드(record) 는 테이블의 구조에 맞추어 여러 개의 서로 다른 데이터 필드 (테이블의 열에 해당) 를 포함한다. 필드들은 그 레코드가 표현하는 데이터 엔티티의 속성을 나타낸다
            
            게시판의 게시글을 저장하기 위한 board 테이블이 있다고 하자. 테이블의 각 레코드는 한 개의 게시글에 해당한다. 그리고 각 레코드는 게시글의 제목, 내용, 작성자, 작성 시간 등 여러 속성을 열의 형태로 가진다.
            
        
        |Board 클래스|board 테이블|
        |---|---|
        |인스턴스|한 행 (가로줄)|
        
        - 개별 레코드, 객체 | | 필드 | 한 열 (세로)
        - 테이블의 특정 속성 정의 |
    - 데이터베이스의 board 테이블에 있는 한 행은 자바의 Board 클래스 인스턴스 하나로 표현되고, 테이블의 각 열은 Board 클래스의 필드와 매핑된다.
        
    - 이렇게 객체지향 프로그래밍에서 사용하는 객체와 관계형 데이터베이스에서 사용하는 테이블 간의 매핑을 통해, 데이터를 보다 직관적이고 효율적으로 조작할 수 있다.
        
        - 개발자는 데이터베이스를 직접 SQL 쿼리문으로 다룰 필요가 없어진다
- 어노테이션 ( @Entity , @Id , @GeneratedValue ) 을 사용해 자기자신 클래스를 데이터베이스 테이블에 매핑하고 기본 키를 정의한다