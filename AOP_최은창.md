
# 00.개요
## AOP란?
- AOP(Aspect Oriented Programming)란 관점 지향 프로그래밍의 약자이며, 객체 지향 프로그래밍을 보완하는 기술이며 메소드나 객체의 기능을 **핵심 관심사**와 **공통 관심사**로 나누어 프로그래밍 하는 것이다. 

## AOP 등장 배경

1. 기존의 OOP의 한계
![](https://velog.velcdn.com/images/cod0216/post/638440da-937f-4cc7-98cd-934da8d4947c/image.png)

  - 객체지향(OOP)은 기능을 클래스로 나누고, 각각의 책임(SRP, 단일 책임의 원책)에 따라 코드를 작성함
  - 로그, 트랜잭션 등 중복되는 로직은 서비스에서 반복 되는데 이는 코드의 중복을 야기 시키고 핵심 비즈니스 로직 외의 코드들도 많아지게 됨
  - 이렇게 되면 변경사항이 생길 시 여러 곳을 수정해야 함 --> 유지보수가 어려워짐

2. 관심사의 분리(SoC(Separation of Concerns)의 필요)
![](https://velog.velcdn.com/images/cod0216/post/167f0749-4df2-4c5a-88da-01a88260ceb4/image.png)
- 그래서 다음과 같은 고민을 하게 됨
- "공통 기능을 따로 빼서 관리할 수 없을까?"
- "핵심 로직과 부가기능을 자연스럽게 분리하고 싶다."
- "로그를 남기거나, 인증을 걸거나, 트랜잭션을 거는 걸 따로 모듈처럼 만들 수 없을까?"
- 이런 필요성이 커지면서 관점 지향 프로그래밍(AOP)이 등장하게 됨

## AOP의 등장

AOP는 다음과 같은 구조를 제안한다.

| 역할     | 내용                                      |
| ------ | --------------------------------------- |
| 핵심 관심사 | 주문, 결제 등 비즈니스에 필요한 코드                   |
| 공통 관심사 | 로깅, 트랜잭션, 보안 등 부가 기능                    |

# 01. Spring AOP 적용

![](https://velog.velcdn.com/images/cod0216/post/ae77cb69-95a0-46f0-941e-8317a8b1ecae/image.png)
```java
@Aspect // AOP임을 명시하는 어노테이션
public class 트랜잭셔널관점{

	트랜잭션메니저

	@Around("예약 서비스들에 대해서") //Advice
	트랜잭션_적용(대상 메서드 정보 ){ // Join Point
		try{
			메서드.실행()
			트랙잭션메니지.커밋
		} catch{
			트랜잭션메니저.롤백()
		}
	}
}
```

- **위 AOP가 바로 `@Transactional`이다**
```java
public class 예약서비스{

	@Transactional
	public void 예약하기(){
		...비즈니즈로직
	}
}
```


# 02.Spring AOP의 원리

## 런타임시 위빙
- 위빙 이란 옷감을 짜다 직조하다 라는 뜻
	- 부가 기능과 핵심 기능을 연결하는 작업
- 런타임 이란 스프링이 실행되는 시점을 의미
- 즉 스프링이 실행되는 시점에 우리가 정의한 **Advice와 핵심 비즈니스 로직을 연결하는 작업**

## 기존 Spring 원리
![](https://velog.velcdn.com/images/cod0216/post/cf147176-d321-41ae-b4cc-ff1d1decaee2/image.png)
- 스프링에서 `예약 서비스`객체 생성 이후 **스프링 컨테이너에 빈으로 등록**한다.

### AOP 적용시 Spring 원리

![](https://velog.velcdn.com/images/cod0216/post/66b10212-c84c-41d2-b0be-673751399c97/image.png)
- 스프링에서 `예약 서비스`객체 생성 이후 **빈 후처리기에 객체를 전달**한다.
-  빈 후처리기에서는 Advisor를 생성한다.
- 이후 빈에 적용 가능한 Advisor를 매칭한다.
- 매칭된 Advisor의 Pointcut 조건에 해당하는 메서드가 있는지 확인한다.
- 조건에 해당하면 프록시 객체를 생성하여, 해당 빈을 프록시로 감싼 후 컨테이너에 등록한다.

### 빈 후처리기(BeanPostProcessor)의 프록시 생성방법
![](https://velog.velcdn.com/images/cod0216/post/264d0d64-e533-425f-a2cc-d3c9a700b350/image.png)
- 빈 후처리기는 ProxyFactory를 사용하여 프록시를 동적으로 생성한다.
- ProxyFactory는 CGLIB를 통해 대상 클래스(`예약 서비스`)를 상속받는 프록시를 생성한다.
  - CGLIB는 대상 크래스가 인터페이스를 구현하지 않았거나 CGLIB 설정 시 사용된다.
- 프록시를 생성할 때 매칭되는 Advisor들을 내부에 등록하고, 메서드 실행 시 Advisor의 Advice를 실행할 수 있도록 구성된다.
- 이렇게 생성된 프록시는 스프링 컨테이너에 등록된다.

# 03. AOP로 전부 적용하면 어떨까?
```java
@Aspect
@Component
public class 예약하기AOP {

	@Before("excution(* com.example.*예약하기(...)))
	public void 예약하기(String content) {
		예약하기_로직()
		System.out.println("DB에 저장됨 : " + content);
	}
}

public class 예약 서비스 {
	public void 예약하기() {
		System.out.println("예약하기 로직이 실행됨");
	}
}
```

- 위 코드처럼 서비스 로직을 AOP에 적용한다면 유지보수에 어려움을 겪게 됨
- 디버깅이 힘들다는 단점이 있음
- 예상하기 힘든 이팩트들이 생길 수 있음
- 유지보수하기에 좋지 않는 코드들이 발생함

**결과적으로 AOP는 횡단 관심사 문제에만 적용해야 한다.** AOP는 OOP를 도와주는 관계이지 대체해서는 안된다.
