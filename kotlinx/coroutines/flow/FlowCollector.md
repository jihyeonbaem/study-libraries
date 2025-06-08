```kotlin
/**
	FlowCollector는 flow의 중간 또는 최종 수집기로 사용된다.
	Flow가 방출하는 값들을 받아 처리하는 객체(T)를 나타낸다.

	이 인터페이스를 직접 구현하지 않고,
	-> 커스텀 연산자를 구현할 때 flow 빌더의 수신 객체로 사용하거나 SAM 변환을 통해 사용해야한다.

	이 인터페이스의 구현체는 스레드-안전 하지 않다.
 */
public fun interface FlowCollector<in T> {

		/**
			업 스트림에서 방출된 값을 수집한다.
			이 메서드는 스레드-안전 하지 않으므로 동시에 호출해서는 안된다.
		 */
    public suspend fun emit(value: T)
}
```