
# 00. 개요
## Executor란
- Executor 프레임워크는 멀티스레딩 및 병렬 처리를 쉽게 사용할 수 있도록 돕는 기능의 모음
- 이 프레임워크는 작업 실행의 관리 및 스레드 풀 관리를 효율적으로 처리해서 개발자가 직접 스레드를 생성하고 관리하는 복잡함을 줄어줌
```java
public interface Executor {
	void execute(Runnable command);
}
```
## ExecutorService
- ExecutorService는 Executor 인터페이스를 확장한 인터페이스로써 작업 제출과 제어 기능을 추가로 제공한다.
- ExecutorService의 가장 대표적인 구현체는 ThreadPoolExecutor다.

```java
public interface ExecutorService extends Executor, AutoCloseable {
	<T> Future<T> submit(Callable<T> task);
	
	@Override
	default void close(){...}
}
```

## ThreadPoolExecutor
- ThreadPoolExecutor는 ExecutorService의 대표적인 구현체로, 이름 그대로 스레드 풀(Thread Pool)을 이용해 작업을 실행하는 클래스
- BlockingQueue : 작업을 보관
- ThreadPool : 스레드를 관리

# 01. ThreadPoolExecutor

<img width="382" height="280" alt="스크린샷 2025-08-20 오전 11 40 17" src="https://github.com/user-attachments/assets/c18846b7-1144-4f9a-b443-bf0fb7b4b641" />

## 구성
- ExecutorService의 구현체인 `new ThreadPoolExecutor()`를 통해 생성 가능
  - `corePoolSize` : 스레드 풀에서 관리되는 기본 스레드의 수
  - `maximumPoolsize` : 스레드 풀에서 관리되는 최대 스레드 수
  - `keepAliveTime`, `TimeUnit unit` : 기본 스레드 수를 초과해서 만들어진 스레드가 생존할 수 있는 대기 시간
    - 이 시간 동안 처리할 작업이 없다면 초과 스레드는 제거
    - `BlockingQueue workQueue`: 작업읍 보관할 블로킹 큐
- `ExecutorService executorService = new ThreadPoolExecutor(2, 2, 0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>())`

## 흐름

1. `ExecutorService executorService = new ThreadPoolExecutor(2, 2, 0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>())`
<img width="350" height="260" alt="스크린샷 2025-08-20 오전 11 40 57" src="https://github.com/user-attachments/assets/432993a7-c0da-4aee-8353-e3716889dc85" />

2. `Main` 스레드가 executorService.execute(`task1 ~ task4`) 호출
<img width="382" height="280" alt="스크린샷 2025-08-20 오전 11 40 17" src="https://github.com/user-attachments/assets/c18846b7-1144-4f9a-b443-bf0fb7b4b641" />

3. 작업 진행
<img width="523" height="292" alt="image" src="https://github.com/user-attachments/assets/6823fbbe-48a6-4c10-8ca8-fb1763a55aaf" />
<img width="526" height="312" alt="image" src="https://github.com/user-attachments/assets/d490b1d7-5576-41ef-851f-0cd8f4b770b6" />

4. 작업이 완료되면 스레드 풀에 스레드를 반납 (`WATING 상태`)
<img width="524" height="305" alt="스크린샷 2025-08-20 오전 11 42 36" src="https://github.com/user-attachments/assets/d1a43824-d71f-452e-b9b4-8aef100f8b07" />

5. 작업 종료
<img width="508" height="301" alt="스크린샷 2025-08-20 오전 11 43 33" src="https://github.com/user-attachments/assets/042b9669-b3e6-49d7-a32e-ce4c27f1b38a" />



> `close()`는 자바 19부터 지원되는 메서드, 이전 버전은 `shutdown()` 호출 --> 둘의 차이 존재


# 02. Executor 전략
- ThreadPoolExecutor를 사용하면 스레드 풀에 사용되는 숫자와 블로킹 큐등 다양한 속성을 조절할 수 있다.
- 자바는 Executors 클래스를 통해 3가지 기본 전략을 제공한다.
  - `newSingleThreadPool()` : 단일 스레드 풀 전략
  - `newFixedThreadPool(nThreads)` : 고정 스레드 풀 전략
  - `newCachedThreadPool()` : 캐시 스레드 풀 전략

## 단일 스레드 풀 전략
- **`newSingleThreadPool()`**
```java
new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILISECONDS, new LinkedBlockingQueue<Runnable>())
```
- 스레드 풀에 기본 스레드 1개만 사용
- 큐 사이즈에 제한이 없음. (`LinkedBlockingQueue`)
- 주로 간단히 사용하거나, 테스트 용도로 사용


## 고정 풀 전략
- **`newFixedThreadPool(nThreads)`**
```java
new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILISECONDS, new LinkedBlockingQueue<Runnable>())
```

- 스레드 풀에 `nThreads`만큼의 기본 스레드를 생성
	- 초과 스레드는 생성하지 않음
- 큐 사이즈에 제한이 없다.(`LinkedBlockingQueue`)
- 스레드 수가 고정되어 있기 때문에 CPU, 메모리 리소스가 어느정도 예측 가능한 안정적인 방식
- 큐 사이즈도 제한이 없어서 작업을 많이 담아두어도 문제가 없음
- **주의**
  - 이 방식의 큰 장점은 스레드 수가 고정되어서 CPU, 메모리 리소스가 어느정도 예측 가능하다는 점
  - 일반적인 상황에서는 안정적인 서비스 운영 가능
  - 하지만 상황에 따라 가장 큰 단점이 될 수 있음
    - 사례1. 점진적인 사용자 확대
    - 사례2. 갑작스런 요청 증가
  - **서버 자원은 여유가 있는데, 사용자만 점점 느려지는 문제 발생**

## 캐시 풀 전략
- **newCachedThreadPool()**
```java
new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
```
- 기본 스레드를 사용하지 않고, 60초 생존 주기를 가진 초과 스레드만 사용
- 초과 스레드의 수는 제한이 없음
- 큐에 작업을 저장하지 않는다. (`SynchronousQueue`)
  - **중간에 버퍼를 두지 않는 스레드 간 직거래**
  - 대신에 생산자의 요청을 스레드 풀의 소비자 스레드가 직접 받아서 바로 처리
- 모든 요청이 대기하지 않고 스레드가 바로바로 처리함, 따라서 빠른 처리 가능
- **주의**
  - 이 방식은 작업 수에 맞추어 스레드 수가 변하기 때문에, 작업 추리 속도가 빠르고 CPU, 메모리를 매우 유연하게 사용할 수 있다는 장점이 있음
  - 따라서 장잠이 큰 단점이 될 수 있음
    - 사례1. 점진적인 사용자 확대
    - 사례2. 갑작스런 요청 증가
    - **캐시 스레드 풀 전략은 서버 자원을 최대한 사용하지만, 서버가 감당할 수 있는 임계점을 넘는 순간 시스템이 다운 가능**

## 사용자 정의 풀 전략
- **점진적인 사용자 확대**
  - 개발한 서비스가 잘 되어서 사용자가 점점 늘어난 경우
- **갑작스런 요청 증가**
  - 마케팅 팀의 이벤트가 대성공 하면서 갑자기 사용자가 폭증

위 상황을 아래와 같이 세분화 전략을 사용하면 어느 정도 대응이 가능
- **일반** : 일반적인 상황에서는 CPU, 메모리 자원을 예측할 수 있도록 고정 크기의 스레드로 서비슬르 안정적으로 운영
- **긴급** : 사용자의 요청이 갑자기 증가하면 긴급하게 스레드를 추가로 투입해서 작업을 빠르게 처리
- **거절** : 사용자의 요청이 폭증해서 긴급 대응도 어렵다면 사용자의 요청을 거절 --> 시스템 다운되는 최악의 상황 모면

```java
new ThreadPoolExecutor(100, 200, 60, TimeUnit.SECONDS, new ArrayBlockingQueue<>(1000));
```
- 100개의 기본 스레드 사용
- 추가로 긴급 대응 가능한 긴급 스레드 100개를 사용
- 1000개의 작업이 큐에 대기 가능

# 03. Executor 예외 정책
- 소비자가 처리할 수 없을 정도로 생산 요청이 가득 찬 경우에 대한 정책

## 예외 정책 4가지
`ThreadPoolExecutor`는 작업을 거절하는 다양한 정책을 제공

1. **AbortPolicy** 
    - 새로운 작업을 제출 시 `RejectedExecutionException` 발생 **(기본 정책)**
2. **DiscardPolicy**
    - 새로운 작업을 조용히 버림
3. **CallerRunsPolicy**
	- 새로운 작업을 제출한 스레드가 대신해서 직접 작업을 실행
4. **사용자 정의(`RejectedExecutionHandler)**
	- 개발자가 직접 정의한 거절 정책 사용

> 해당 정책은 `shutdown()` 호출 후 거절된 작업에도 같은 정책이 적용됨


## RejectedExecutionHandler 인터페이스

```java
public interface RejectedExecutionHandler {
	void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```
- `ThreadPoolExecutor`가 거절 하는 상황 발생 시 해당 클래스를 상속받은 구현체에 정의된 정책을 사용한다.
- `ThreadPoolExecutor`의 마지막 인자에 해당 인터페이스의 구현체를 넣어 사용


## AbortPolicy
```java
ExecutorService es = new ThreadPoolExecutor(1,1,0, TimeUnit.SECONDS,  
        new SynchronousQueue<>(), new ThreadPoolExecutor.AbortPolicy());
```
- 기본적으로 설정되어 있는 정책
- **작업이 거절되면 `RejectedExecutionException` 을 던짐** 
- `AbortPolicy`는 `RejectedExecutionHandler`의 구현체
```java
public static class AbortPolicy implements RejectedExecutionHandler {
	public void rejectedExectuion(Runnable r, ThreadPoolExecutor e) {
		throw new RejectedExecutionException("Task "+ r.toStrig() + " rejected from " + e.toString());
	}
}
```


## DiscardPolicy
```java
ExecutorService es = new ThreadPoolExecutor(1,1,0, TimeUnit.SECONDS,  
        new SynchronousQueue<>(), new ThreadPoolExecutor.DiscardPolicy());
```
- **거절된 작업을 무시하고 아무런 예외를 발생 시키지 않음**
- `DiscardPolicy`는 `RejectedExecutionHandler`의 구현체
```java
public static class DiscardPolicy implements RejectedExecutionHandler {
	public void rejectedExectuion(Runnable r, ThreadPoolExecutor e) {
		// empty <-- 비어있음
	}
}
```


## CallerRunsPolicy
```java
ExecutorService es = new ThreadPoolExecutor(1,1,0, TimeUnit.SECONDS,  
        new SynchronousQueue<>(), new ThreadPoolExecutor.CallerRunsPolicy());
```
- 호출한 스레드가 작업을 수행 (생산자가 소비자 역할을 대신함)
	- 이로 인해 새로운 작업을 제출하는 스레드의 속도가 느려질 수 있음
	- 덕분에 생산 속도 조절이 가능 (**소비자 스레드 바로 생성 억제 가능**)
- `CallerRunsPolicy`는 `RejectedExecutionHandler`의 구현체
```java
public static class CallerRunsPolicy implements RejectedExecutionHandler {
	public void rejectedExectuion(Runnable r, ThreadPoolExecutor e) {
		if(!e.isShutdown()){ // shutDown() 호출 시 호출한 스레드가 실행 되지 않도록 if()에서 호출
			r.run(); // 호출한 스레드가 직접 실행
		}
	}
}
```

- `ThreadPoolExecutor`를 `shutdown()`하면 이후 요청 하는 작업이 거절되는데 이때도 같은 정책이 적용됨
- 그런데 `CallerRunsPolicy` 정책은 `shutDown()` 이후에도 작업을 수행함
- 따라서 `shutdown()` 조건을  체크해서 이 경우에는 작업을 수행하지 않도록 해야함


## 사용자 정의
```java
ExecutorService es = new ThreadPoolExecutor(1,1,0, TimeUnit.SECONDS,  
        new SynchronousQueue<>(), new ThreadPoolExecutor.MyRejectedExecutionHandler());
```
- `RejectedExecutionHandler` 인터페이스를 구현하여 자신만의 거절 처리 전략 정의
- 특정 요구사항에 맞는 작업 거절 방식을 설정 할 수 있음
```java
public static class MyRejectedExecutionHandler implements RejectedExecutionHandler {
	static AtmoicInteger count = new AtomicInteger(0);
	public void rejectedExectuion(Runnable r, ThreadPoolExecutor e) {
		int i = count.incrementAndGet();
		log("[경고] 거절된 누적 작업 수: " + i);
	}
}
```







