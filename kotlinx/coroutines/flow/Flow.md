```kotlin
/**

*/
public interface Flow<out T> {
		/**
		*/
    public suspend fun collect(collector: FlowCollector<T>)
}
```

```kotlin
/**
		주어진 일시중단 가능한 block을 기반으로 cold flow를 생성한다.
		flow가 cold라는 것은 block이 매번 호출된다는 의미이다. 터미널 연산자가 resulting flow에 적용될 때 마다

		*resulting flow란? flow { ... } 빌더를 사용해 생성된 최종 Flow 객체를 의미한다.

		flow 빌더의 emit은 기본적으로 취소할 수 있고, emit 호출 마다 ensureActive가 호출되어 코루틴이 활성 상태인지 확인한다.

		flow의 컨텍스트를 보존하기 위해서 emit은 엄격하게 block의 디스패처에서만 호출되어야한다.
		withContext를 통해 다른 컨텍스트로 전환하면 IllegalStateException이 발생한다.
		flow의 실행 컨텍스트를 변경하고 싶다면 -> flowOn 연산자를 사용해라.
 */
public fun <T> flow(@BuilderInference block: suspend FlowCollector<T>.() -> Unit): Flow<T> = SafeFlow(block)
```