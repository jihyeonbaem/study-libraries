## (수신채널 반복기) ChannelIterator<out E>

- ReceiveChannel에 대한 Iterator이다.
- 이 인터페이스의 인스턴스는 스레드-안전 하지 않으며, 여러개의 코루틴에서 동시에 사용하면 안된다.

### suspend operator fun hasNext(): Boolean

- next를 호출해 가져올 요소를 준비
- 이전 hasNext 호출로 가져온 요소가 아직 next로 소비되지 않았다면 → `true` 반환
- 채널에 사용가능한 요소가 있다면 true 반환 + 해당 요소를 채널에서 제거 (이 요소는 next 호출로 반환됨)
- 채널이 cause없이 isClosedForReceive == true 로 닫히면 → `false` 반환
- 채널이 cause와 함께 닫히면 → 닫힌 예외 cause를 던짐
- 채널이 닫히지 않았지만 요소가 없다면 → 채널에 요소가 전송되거나 채널이 닫힐 때까지 일시중단됨

---

- 이 일시중단 함수는 취소가능하다
    - 일시중단 중일 때, 이 함수를 호출한 코루틴의 Job 취소되면 즉시 Cancellation과 함께 재개됨
- 즉시 취소 보장

  → hasNext()가 작업 중 채널에서 요소를 가져왔더라도, 일시중단 중 취소되면 CancellationException이 던져짐

- 즉시 취소 보장의 결과로

  → 채널에서 가져온 일부 값이 손실될 수 있음.

  전달되지 않은 요소를 처리하는 방법은 → Channel의 unDeliveredElement 참고

- 일시중단되지 않을 때, 즉, 다음 요소가 즉시 사용가능할 때 → 취소를 확인하지 않음

  ⇒ 필요시 ensureActive, CoroutineScope.isActive를 사용해서 취소를 확인해야함


### operator fun next(): E

- 이전의 `hasNext`호출로 채널에서 제거된 요소를 가져오거나, hasNext가 호출되지 않았다면

  → IllegalStateException을 던진다.

- 이 메서드는 `hasNext`와 함께 사용해야한다.

    ```kotlin
    while (iterator.hasNext()) {
    		val element = iterator.next()
    		// ... 요소 처리하기 ...
    }
    ```

- 채널 반복을 더 자연스럽게 하는 방법은 for 루프를 사용하는 것이다.

    ```kotlin
    for (element in channel) {
    		// ... 요소 처리하기 ...
    }
    ```

- hasNext가 true를 반환한 경우 절대 예외를 던지지 않는다.
- hasNext가 채널이 닫힌 원인으로 예외를 던졌다면 → 동일한 예외를 다시 던진다.
- hasNext가 원인 없이 채널이 닫혀 false를 반환했다면 → ClosedReceiveChannelException을 던진다.