## (하나만 선택하기) Select

```kotlin
@OptIn(ExperimentalContracts::class)
public suspend inline fun <R> select(crossinline builder: SelectBuilder<R>.() -> Unit): R {
    contract {
        callsInPlace(builder, InvocationKind.EXACTLY_ONCE)
    }
    return SelectImplementation<R>(coroutineContext).run {
        builder(this)
        // TAIL-CALL OPTIMIZATION: the only
        // suspend call is at the last position.
        doSelect()
    }
}
```

- select 함수는 지정된 여러 일시중단 함수의 작업의 결과를 동시에 감시한다.

  절들은 builder 스코프에서 정의된다.

- 호출자는 절들 중 하나가 선택되거나 실패할 때까지 일시중단된다.

---

- 최대 하나의 절이 원자적으로 선택되고, 선택된 block이 실행된다.

  → 선택된 절의 결과가 select의 결과가 된다.

- 만약 어떤 절이 실패하면

  → select 호출은 해당 예외를 발생시키고, 어떠한 절도 선택되지 않는다.


---

- select 함수는 첫 번째 절에 편향되어 있다.

  → 여러 절이 동시에 선택될 수 있을 때

  ⇒ 첫번째 절이 우선권을 가진다. (편항되지 않은 무작위 선택을 하려면 selectUnbiased를 사용해라)


---

- select 표현식에는 default 절 이 없다.

  → 각 비동기 작업은 비동기가 아닌 버전을 제공한다. (예시: onReceive → tryReceive())

  ⇒ 이를 when 문과 함께 사용해 여러 선택지중 선택하거나, 즉시 선택할 수 없는경우 else로 기본동작을 수행할 수 있다.


### 지원되는 select 메서드

| Receiver | Suspending function | select clause |
| --- | --- | --- |
| Job | join | onJoin |
| Deferred | await | onAwait |
| SendChannel | send | onSend |
| ReceiveChannel | receive | onReceive |
| ReceiveChannel | receiveCatching | onReceiveCatching |
| none | delay | onTimeout |

- select 일시중단 함수는 취소가능하다.

  → select 함수가 대기중일 때, 현재 코루틴이 취소되면 CancellationException과 함께 재개된다.

- 즉시 취소 보장

  → select 함수가 결과를 반환할 준비가 됐더라도, 일시중단중 취소되면 → CancellationException을 발생

- 일시중단 되지않은 상태에서는 취소를 확인하지 않는다.

  → yield나 isActive를 사용해 주기적으로 취소를 확인해라