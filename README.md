# TookitWithFlow
Some development toolkits implement with coroutine.
## Click with throttle first and throttle latest by flow

```kotlin
@FlowPreview
@ExperimentalCoroutinesApi
fun View.throttleFirst(scope: CoroutineScope, timeDelay: Long = 1000, callback: () -> Unit) {
    scope.launch {
        var isSend = true
        callbackFlow {
            setOnClickListener {
                if (isSend) {
                    trySend(Unit)
                }
                isSend = false
            }

            awaitClose { setOnClickListener(null) }
        }.catch { Log.d("TAG", it.message.toString()) }
            .shareIn(scope, SharingStarted.WhileSubscribed(), 0)
            .safeCollect {
                callback.invoke()
                delay(timeDelay)
                isSend = true
            }
    }
}

@FlowPreview
@ExperimentalCoroutinesApi
fun View.throttleLatest(scope: CoroutineScope, timeDelay: Long = 1000, callback: () -> Unit) {
    scope.launch {
        callbackFlow {
            setOnClickListener { trySend(Unit) }

            awaitClose { this@throttleLatest.setOnClickListener(null) }
        }.catch { Log.d("TAG", it.message.toString()) }
            .shareIn(scope, SharingStarted.WhileSubscribed(), 0)
            .mapLatest { delay(timeDelay) }
            .buffer(0)
            .safeCollect { callback.invoke() }
    }
}

suspend fun <T> Flow<T>.safeCollect(block: suspend () -> Unit) =
    runCatching { collect { block.invoke() } }.onFailure { Log.d("TAG", it.message.toString()) }
```
