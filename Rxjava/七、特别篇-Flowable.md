
## RxJava Flowable 深度解析文档

### 目录

1.  **引言：为什么需要 Flowable？**
    *   Observable 的局限性
    *   背压（Backpressure）概念引入
    *   Flowable 的诞生与定位
2.  **Flowable 基础使用**
    *   创建 Flowable (Creating Flowables)
    *   订阅 Flowable (Subscribing to Flowables)
    *   `Subscriber` 与 `Subscription`
    *   `request(n)` 的核心作用
3.  **背压（Backpressure）详解**
    *   背压问题的本质：生产者与消费者速度失衡
    *   `request(n)` 协议：消费者驱动的拉取模式
    *   背压策略（Backpressure Strategies）
        *   `MISSING`
        *   `ERROR`
        *   `BUFFER`
        *   `DROP`
        *   `LATEST`
4.  **Flowable 操作符与背压**
    *   标准操作符（map, filter 等）对背压的传递
    *   `onBackpressureBuffer()`
    *   `onBackpressureDrop()`
    *   `onBackpressureLatest()`
    *   `observeOn()` 与背压
    *   `Flowable.generate()` 与 `Flowable.create()`
5.  **Flowable 与其他类型的转换**
    *   `Flowable` 转 `Observable` (`toObservable()`)
    *   `Observable` 转 `Flowable` (`toFlowable()`)
6.  **内部机制与源码解析（伪代码）**
    *   订阅流程 (`subscribe` 调用链)
    *   `request(n)` 的向上传递
    *   操作符如何处理背压（以 `map` 和 `observeOn` 为例）
    *   缓冲区的实现机制（简述）
7.  **最佳实践与总结**
    *   何时选择 `Flowable` 而不是 `Observable`？
    *   常见陷阱与注意事项
    *   性能考量
    *   总结

---

### 1. 引言：为什么需要 Flowable？

*   **Observable 的局限性：**
    在 RxJava 1.x 和 RxJava 2.x 的早期，`Observable` 是主要的响应式类型。它非常适合处理事件流，如 UI 点击、有限的数据序列等。然而，当上游（生产者）发射数据的速度远超下游（消费者）处理速度时，`Observable` 缺乏内置的机制来协调这种速度差异。数据会无限制地涌向下游，最终可能导致内存溢出（`OutOfMemoryError`）或应用程序崩溃。这就是所谓的 **背压问题（Backpressure Problem）**。虽然 RxJava 1.x 尝试通过一些操作符（如 `onBackpressureBuffer`, `onBackpressureDrop`）来缓解，但这更像是“补丁”而非根本解决方案。

*   **背压（Backpressure）概念引入：**
    背压是一种流量控制机制。它允许消费者（`Subscriber`）向上游生产者（`Flowable` 或其上游操作符）发出信号，告知它当前能够处理多少数据项。生产者根据这个信号来控制其数据发射的速率，从而避免压垮消费者。

*   **Flowable 的诞生与定位：**
    为了从根本上解决背压问题，RxJava 2.x 引入了 `Flowable`。`Flowable` 是一个遵循 **Reactive Streams 规范** 的响应式基类。该规范定义了一套标准的接口（`Publisher`, `Subscriber`, `Subscription`, `Processor`）和交互规则，其中就包含了背压机制。
    *   **定位：** `Flowable` 设计用于处理可能发射 0 到 N 个（N 可能非常大甚至无限）数据项的流，并且内置支持背压。它适用于那些数据量不可控或可能很大的场景，例如：文件读写、数据库查询结果、网络流、传感器数据等。
    *   **与 `Observable` 的区别：**
        *   `Flowable`：支持背压，用于处理大量/可能无限的数据流。订阅者是 `Subscriber<T>`。
        *   `Observable`：不支持背压，适用于数据量可控、不会导致 OOM 的场景（如 UI 事件、少量数据）。订阅者是 `Observer<T>`。

### 2. Flowable 基础使用

*   **创建 Flowable:**
    与 `Observable` 类似，可以通过多种方式创建 `Flowable`：

    ```java
    // 1. just: 发射固定数量的项
    Flowable<Integer> flowableJust = Flowable.just(1, 2, 3, 4, 5);

    // 2. fromIterable: 从集合发射
    List<String> list = Arrays.asList("A", "B", "C");
    Flowable<String> flowableFromIterable = Flowable.fromIterable(list);

    // 3. range: 发射一个范围内的整数
    Flowable<Integer> flowableRange = Flowable.range(10, 5); // 10, 11, 12, 13, 14

    // 4. interval: 定期发射递增的 Long 值 (默认在 computation 调度器, 支持背压)
    Flowable<Long> flowableInterval = Flowable.interval(1, TimeUnit.SECONDS);

    // 5. create: 用于自定义发射逻辑，需要处理背压
    Flowable<String> flowableCreate = Flowable.create(emitter -> {
        // ... 发射逻辑 ...
        // 需要检查 emitter.isCancelled()
        // 需要响应 emitter.requested()
        emitter.onNext("Data 1");
        emitter.onNext("Data 2");
        emitter.onComplete();
    }, BackpressureStrategy.BUFFER); // 指定背压策略

    // 6. generate: 同步地、逐个地生成数据，自带状态管理和背压支持
    Flowable<Integer> flowableGenerate = Flowable.generate(
        () -> 0, // 初始状态
        (state, emitter) -> {
            emitter.onNext(state); // 发射当前状态
            if (state == 10) {
                emitter.onComplete();
            }
            return state + 1; // 更新状态
        });
    ```

*   **订阅 Flowable:**
    订阅 `Flowable` 需要使用 `org.reactivestreams.Subscriber` 接口。

    ```java
    flowableJust.subscribe(new Subscriber<Integer>() {
        private Subscription subscription;
        private int count = 0;
        private final int BATCH_SIZE = 2; // 每次请求处理多少个

        @Override
        public void onSubscribe(Subscription s) {
            System.out.println("onSubscribe: Subscribed!");
            this.subscription = s;
            // *** 关键：必须调用 request() 来启动数据流 ***
            System.out.println("Requesting initial batch: " + BATCH_SIZE);
            subscription.request(BATCH_SIZE); // 请求第一批数据
        }

        @Override
        public void onNext(Integer integer) {
            System.out.println("onNext: Received " + integer);
            count++;
            // 处理完一批后，再请求下一批
            if (count % BATCH_SIZE == 0) {
                 System.out.println("Processed batch, requesting next: " + BATCH_SIZE);
                 subscription.request(BATCH_SIZE);
            }
            // 模拟处理耗时
            try { Thread.sleep(100); } catch (InterruptedException e) { e.printStackTrace(); }
        }

        @Override
        public void onError(Throwable t) {
            System.err.println("onError: " + t.getMessage());
        }

        @Override
        public void onComplete() {
            System.out.println("onComplete: Stream finished.");
        }
    });
    ```

*   **`Subscriber` 与 `Subscription`:**
    *   `Subscriber<T>`: `Flowable` 的观察者。接口包含四个方法：
        *   `onSubscribe(Subscription s)`: 订阅发生时调用，传递 `Subscription` 对象。**这是建立连接和请求数据的起点。**
        *   `onNext(T t)`: 接收到一个数据项时调用。
        *   `onError(Throwable t)`: 发生错误时调用，流终止。
        *   `onComplete()`: 数据流正常完成时调用，流终止。
    *   `Subscription`: 代表了生产者和消费者之间的连接。接口包含两个方法：
        *   `request(long n)`: 消费者调用此方法通知生产者可以发送 `n` 个数据项。**这是背压的核心。** 如果 `n <= 0`，会抛出 `IllegalArgumentException`。`Long.MAX_VALUE` 表示请求无限数据（即关闭背压）。
        *   `cancel()`: 消费者调用此方法通知生产者停止发送数据，并清理资源。

*   **`request(n)` 的核心作用：**
    `request(n)` 是消费者控制数据流速的关键。没有 `request()` 调用，即使 `Flowable` 有数据，`onNext` 也不会被调用。消费者通过 `request(n)` 显式地向上游“拉取”数据，决定自己何时以及能处理多少数据。

### 3. 背压（Backpressure）详解

*   **背压问题的本质：**
    当生产者（Upstream/Source）产生数据的速率 > 消费者（Downstream/Subscriber）处理数据的速率时，中间若没有协调机制，数据就会堆积。对于 `Observable`，这通常发生在内部缓冲区（如 `observeOn` 操作符引入的）或者直接导致下游 `Observer` 的 `onNext` 被频繁调用，最终耗尽内存或 CPU 资源。

*   **`request(n)` 协议：消费者驱动的拉取模式：**
    `Flowable` 采用 Reactive Streams 规范定义的协议来解决这个问题：
    1.  **订阅时 (`onSubscribe`)**: 消费者获得 `Subscription` 对象。
    2.  **请求数据**: 消费者通过 `subscription.request(n)` 告知上游它准备好接收 `n` 个数据项。
    3.  **生产数据**: 上游收到请求后，开始发射数据，但最多发射 `n` 个 `onNext` 事件。
    4.  **再次请求**: 消费者处理完部分或全部数据后，可以再次调用 `request(m)` 请求更多数据。
    5.  **循环**: 这个“请求-生产”的循环持续进行，直到流完成 (`onComplete`)、出错 (`onError`) 或被取消 (`cancel`)。
    这种机制确保了数据流速由最慢的消费者决定，防止了数据淹没下游。

*   **背压策略（Backpressure Strategies）：**
    当你使用 `Flowable.create()` 方法将非背压感知的代码（例如回调API）包装成 `Flowable` 时，你需要告诉 RxJava 如何处理潜在的背压问题。这时就需要指定一个 `BackpressureStrategy`：

    *   `MISSING`: 不应用任何策略。如果下游没有及时 `request()`，生产者继续发射可能导致 `MissingBackpressureException`。适用于你知道下游总能跟上或者源头本身就有节流的情况。
    *   `ERROR`: 如果下游跟不上（即生产者想发射数据但下游的请求计数为0），立即抛出 `MissingBackpressureException`。这是最严格的策略，能快速发现背压问题。
    *   `BUFFER`: 将生产者发射的数据缓存起来。如果下游处理慢，数据会暂存在内部缓冲区中。
        *   **优点：** 不丢失数据。
        *   **缺点：** 如果下游一直很慢，缓冲区可能无限增长，最终导致 `OutOfMemoryError`。需要谨慎使用，最好配合 `onBackpressureBuffer()` 操作符设置容量限制或溢出策略。
    *   `DROP`: 如果下游跟不上（请求计数为0），生产者尝试发射的新数据会被直接丢弃。下游只会收到它请求范围内的、且在它请求时生产者恰好发射的数据。
    *   `LATEST`: 类似 `DROP`，但会保留最新的一个数据项。当下游再次 `request()` 时，如果之前有被“暂存”的最新数据项，会先发射它，然后再处理后续生产的数据。

    **示例 (`Flowable.create`):**

    ```java
    Flowable<Integer> source = Flowable.create(emitter -> {
        for (int i = 0; i < 1000; i++) {
            if (emitter.isCancelled()) return;

            // 关键: 检查下游是否还能接收 (仅作示意，实际 create 不好直接判断)
            // 在 BUFFER/DROP/LATEST 策略下，发射器内部会处理这个逻辑
            // 在 ERROR 策略下，如果此时 requested() == 0，调用 onNext 会抛异常
            // 在 MISSING 策略下，调用 onNext 可能在下游操作符（如 observeOn）处抛异常

            emitter.onNext(i);
            System.out.println("Emitted: " + i + ", Requested: " + emitter.requested()); // requested() 显示下游当前还能接收多少
            // 模拟快速发射
            try { Thread.sleep(1); } catch (InterruptedException e) {}
        }
        emitter.onComplete();
    }, BackpressureStrategy.DROP); // 使用 DROP 策略

    source
        .observeOn(Schedulers.computation()) // 切换到不同线程，模拟耗时处理
        .subscribe(new Subscriber<Integer>() {
            Subscription sub;
            @Override
            public void onSubscribe(Subscription s) {
                sub = s;
                System.out.println("Requesting 10");
                sub.request(10); // 初始请求10个
            }
            @Override
            public void onNext(Integer integer) {
                System.out.println("Processing: " + integer);
                // 模拟耗时处理
                try { Thread.sleep(100); } catch (InterruptedException e) {}
                // 处理完一个，请求下一个 (非常慢的消费者)
                // sub.request(1); // 或者等处理完一批再请求
            }
            @Override public void onError(Throwable t) { t.printStackTrace(); }
            @Override public void onComplete() { System.out.println("Done"); }
        });
    ```
    在这个例子中，因为下游处理慢且初始只请求了10个，并且使用了 `DROP` 策略，大量由生产者发射的数据（10 到 999）会被丢弃。

### 4. Flowable 操作符与背压

*   **标准操作符（map, filter 等）：**
    大多数标准的转换和过滤操作符（如 `map`, `filter`, `flatMap` 等）本身是支持背压的。它们会尊重下游的 `request()` 调用，并将其传播到上游。它们通常不会自己引入背压问题，除非操作本身非常耗时。

*   **`onBackpressureBuffer()`:**
    对一个 `Flowable` 应用缓冲策略，即使它上游可能不支持背压或者你想改变其行为。可以指定缓冲区大小、溢出策略（抛异常、丢弃最老、丢弃最新）。
    ```java
    Flowable.interval(1, TimeUnit.MILLISECONDS) // 极快的生产者
            .onBackpressureBuffer(100, // 缓冲区大小 100
                    () -> System.out.println("Buffer overflow!"), // 溢出时回调
                    BackpressureOverflowStrategy.DROP_LATEST) // 溢出时丢弃最新的
            .observeOn(Schedulers.io())
            .subscribe(/* ... slow consumer ... */);
    ```

*   **`onBackpressureDrop()`:**
    应用丢弃策略。当下游无法处理时，丢弃上游发射的新数据。可以提供一个 `onDrop` 回调来处理被丢弃的数据。
    ```java
    sourceFlowable
            .onBackpressureDrop(droppedItem -> System.out.println("Dropped: " + droppedItem))
            .subscribe(/* ... slow consumer ... */);
    ```

*   **`onBackpressureLatest()`:**
    应用保留最新策略。当下游无法处理时，丢弃旧数据，只保留最新的一个，当下游再次请求时优先发射这个最新的数据。
    ```java
    sourceFlowable
            .onBackpressureLatest()
            .subscribe(/* ... slow consumer ... */);
    ```

*   **`observeOn()` 与背压:**
    `observeOn()` 用于切换下游操作符执行的调度器（线程）。它内部包含一个缓冲区（默认大小为 `Flowable.bufferSize()`，通常是 128）。它从上游 `request()` 数据填充缓冲区，然后当下游 `request()` 时，从缓冲区取出数据发射给下游。这个缓冲区使得 `observeOn` 能够解耦上下游的执行，但也可能成为背压点。如果下游处理慢，`observeOn` 的缓冲区可能会满，此时它会停止向上游 `request()`，从而将背压传递上去。

*   **`Flowable.generate()` 与 `Flowable.create()`:**
    *   `generate()`: 设计用于同步地、状态驱动地生成数据。它每次只允许 `emitter.onNext()` 调用一次，并且自动处理了 `request()` 计数，是实现自定义同步源且支持背压的推荐方式。
    *   `create()`: 更灵活，允许异步发射，但需要开发者手动管理发射逻辑，检查 `isCancelled()`, 并且根据 `BackpressureStrategy` 隐式或显式地处理 `requested()` 计数。更容易出错。

### 5. Flowable 与其他类型的转换

*   **`Flowable` 转 `Observable` (`toObservable()`):**
    将 `Flowable` 转换为 `Observable`。这个过程会 **丢失背压信息**。如果原始 `Flowable` 可能产生大量数据，转换后的 `Observable` 可能会导致下游 `Observer` 出现 `MissingBackpressureException` 或 OOM。仅在确定数据量可控或下游能快速处理时使用。

*   **`Observable` 转 `Flowable` (`toFlowable()`):**
    将 `Observable` 转换为 `Flowable`。因为 `Observable` 本身不携带背压信息，所以在转换时 **必须** 提供一个 `BackpressureStrategy` 来告诉 `Flowable` 如何处理潜在的数据过载。
    ```java
    Observable<Integer> observable = Observable.range(1, 1_000_000);

    // 必须指定策略，例如 BUFFER
    Flowable<Integer> flowable = observable.toFlowable(BackpressureStrategy.BUFFER);

    flowable.subscribe(/* ... Subscriber with request(n) ... */);
    ```

### 6. 内部机制与源码解析（伪代码）

理解 `Flowable` 内部如何工作有助于更好地使用它。以下使用伪代码简化说明：

*   **订阅流程 (`subscribe` 调用链):**
    ```pseudocode
    // User calls:
    sourceFlowable.map(data -> transform(data)).subscribe(mySubscriber);

    // Internal Steps (Simplified):
    // 1. subscribe(mySubscriber) is called on the MapFlowable instance.
    MapFlowable.subscribe(downstreamSubscriber):
        // Create a MapSubscriber that wraps the downstreamSubscriber
        MapSubscriber mapSub = new MapSubscriber(downstreamSubscriber, mappingFunction);
        // Propagate the subscription upstream
        sourceFlowable.subscribe(mapSub); // mapSub is now the subscriber for the source

    // 2. subscribe(mapSub) is called on the sourceFlowable instance (e.g., RangeFlowable).
    RangeFlowable.subscribe(actualSubscriber): // actualSubscriber is mapSub here
        // Create a Subscription specific to Range (e.g., RangeSubscription)
        RangeSubscription rangeSub = new RangeSubscription(actualSubscriber, start, count);
        // Call onSubscribe on the actual subscriber, passing the subscription
        actualSubscriber.onSubscribe(rangeSub); // mapSub.onSubscribe(rangeSub) is called

    // 3. mapSub.onSubscribe(rangeSub) is called.
    MapSubscriber.onSubscribe(upstreamSubscription):
        this.upstreamSub = upstreamSubscription;
        // Pass the subscription down to the original user subscriber
        this.downstreamSubscriber.onSubscribe(this); // mySubscriber.onSubscribe(mapSub) is called
        // Note: MapSubscriber itself implements Subscription for the downstream

    // 4. mySubscriber.onSubscribe(mapSub) is called.
    MySubscriber.onSubscribe(subscription): // subscription is mapSub here
        this.subscription = subscription;
        // User now has the subscription and can call request()
        this.subscription.request(initialN);
    ```
    核心思想：每个操作符都会创建一个包装下游 `Subscriber` 的新 `Subscriber`，并将订阅请求向上传递。同时，这个包装 `Subscriber` 通常也实现了 `Subscription` 接口，用于接收下游的 `request()` 和 `cancel()` 调用。

*   **`request(n)` 的向上传递:**
    ```pseudocode
    // User calls:
    mySubscriber.subscription.request(n); // subscription is the MapSubscriber instance

    // Internal Steps:
    // 1. request(n) is called on MapSubscriber.
    MapSubscriber.request(n):
        // Map operator usually doesn't buffer, it requests the same amount from upstream.
        // (It might track outstanding requests if needed).
        this.upstreamSub.request(n); // Request propagates upwards

    // 2. request(n) is called on RangeSubscription.
    RangeSubscription.request(n):
        // Atomically add 'n' to the internal requested count.
        requestedCount.addAndGet(n);
        // Trigger the emission loop/logic if not already running.
        drain(); // Start emitting if possible and needed.

    // Emission Logic (in drain() or similar):
    RangeSubscription.drain():
        while (true):
            // How many items are requested by downstream?
            r = requestedCount.get();
            // How many have we emitted so far?
            emitted = emittedCount;
            // How many can we emit now?
            canEmit = r - emitted;

            if (canEmit == 0):
                // Downstream hasn't requested more, wait.
                break;

            // Loop to emit 'canEmit' items (or fewer if source completes/errors)
            for i = 0 to canEmit:
                if (isCancelled()): return;
                currentValue = calculateNextValue();
                if (sourceIsComplete()):
                    actualSubscriber.onComplete();
                    return;

                actualSubscriber.onNext(currentValue); // Emit one item
                emittedCount++;

            // If we emitted everything requested so far, check again in case more was requested concurrently.
            // Otherwise, loop continues if canEmit was less than requested 'r'.
        // If loop finished because source completed or error occurred, handle that.
    ```
    核心思想：`request(n)` 调用沿着 `Subscription` 链向上传播。源头（或有缓冲的操作符）会维护一个请求计数器。当收到 `request(n)` 时，增加计数器。内部的发射逻辑（通常在一个循环或调度任务中）会检查这个计数器，并且只发射不超过请求数量的数据项。每发射一个，计数器（或已发射计数）相应减少/增加。

*   **操作符如何处理背压（`map` 和 `observeOn`）:**
    *   **`map`:** 通常是“透明”的。它收到下游 `request(n)`，就向上游 `request(n)`。收到上游 `onNext(T)`，转换后立刻调用下游 `onNext(R)`。它不缓冲，直接传递背压信号。
    *   **`observeOn`:** 这是关键的背压处理点。
        ```pseudocode
        ObserveOnSubscriber.onSubscribe(upstreamSub):
            this.upstreamSub = upstreamSub;
            downstream.onSubscribe(this); // Pass itself (as Subscription) downstream
            // Initially request ObserveOn's internal buffer size from upstream
            upstreamSub.request(bufferSize);

        ObserveOnSubscriber.request(n):
            // Add 'n' to the downstream requested count
            downstreamRequested.addAndGet(n);
            // Trigger the drain loop on the specified scheduler
            scheduleDrain();

        ObserveOnSubscriber.onNext(item):
            // Received item from upstream
            if (!internalQueue.offer(item)): // Try adding to internal buffer
                // Buffer full! Handle based on strategy (e.g., signal error, block, drop)
                // Since observeOn uses a SpscArrayQueue, it typically signals error or drops if full.
                // Importantly, it stops requesting from upstream if buffer is full.
                handleBufferFull();
            else:
                // Item added to queue, trigger drain loop if needed
                scheduleDrain();

        ObserveOnSubscriber.drain(): // Runs on the specified scheduler (e.g., IO thread)
            while (true):
                ds_requested = downstreamRequested.get(); // How many downstream wants
                emitted = 0;
                while (emitted < ds_requested):
                    if (isCancelled()): queue.clear(); return;
                    item = internalQueue.poll(); // Get item from buffer
                    if (item == null):
                        // Buffer is empty, break inner loop
                        break;

                    downstream.onNext(item); // Emit to downstream
                    emitted++;

                // Reduce downstream requested count
                if (emitted > 0):
                    downstreamRequested.addAndGet(-emitted);
                    // Request more from upstream to refill buffer
                    upstreamSub.request(emitted); // Request the number just consumed

                // Check termination conditions, check if more work needed, etc.
                if (noMoreWork()): break;
            // End drain loop
        ```
        核心思想：`observeOn` 有一个内部队列（缓冲区）。它从上游预取数据（通常是缓冲区大小）放入队列。当下游 `request(n)` 时，它从队列中取出最多 `n` 个元素，在指定的调度器上发射给下游。发射了多少，就再向上游 `request()` 多少，试图保持缓冲区填充。如果队列满了，它会停止向上游请求，从而实现背压。

*   **缓冲区的实现机制（简述）:**
    RxJava 内部大量使用高效的、通常是无锁的队列实现（如 JCTools 提供的 `SpscArrayQueue` - Single Producer Single Consumer Array Queue）来作为缓冲区。这些队列针对特定的并发场景进行了优化，以减少锁竞争和提高吞吐量。

### 7. 最佳实践与总结

*   **何时选择 `Flowable` 而不是 `Observable`？**
    *   **使用 `Flowable` 的场景：**
        *   处理的数据项可能超过几千个。
        *   数据流可能无限（如定时器、传感器）。
        *   数据源速度不可控或远快于处理速度（文件IO、网络流、数据库结果集）。
        *   需要与遵循 Reactive Streams 规范的库（如 Reactor, Akka Streams）互操作。
        *   你不确定数据量或速率，倾向于安全（避免 OOM）时。
    *   **使用 `Observable` 的场景：**
        *   处理 UI 事件（点击、输入），数量有限且通常不快速。
        *   数据量明确且不大（少于 1000 项通常是安全的经验法则，但非绝对）。
        *   代码中不需要处理背压，可以简化心智模型。
        *   性能极其敏感且确定不会有背压问题的短生命周期流（`Observable` 开销略低于 `Flowable`，但差异通常不大）。

*   **常见陷阱与注意事项：**
    *   **忘记在 `Subscriber` 的 `onSubscribe` 中调用 `request(n)`：** 这是最常见的错误，会导致流不启动。
    *   **在 `Flowable.create()` 中未正确处理背压：** 如果使用了 `MISSING` 或 `ERROR` 策略，需要确保发射逻辑受控。使用 `BUFFER`, `DROP`, `LATEST` 时要理解其行为和潜在的 OOM 或数据丢失风险。优先考虑 `Flowable.generate()` 用于同步源。
    *   **无限 `request(Long.MAX_VALUE)`：** 虽然可行，但这实际上关闭了背压，使其行为类似 `Observable`。只在确定下游能处理所有数据时使用。
    *   **`onBackpressureBuffer()` 的无限缓冲：** 默认的 `onBackpressureBuffer()` 如果不指定大小，可能导致 OOM。务必考虑设置容量限制和溢出策略。
    *   **阻塞操作符中的背压：** `blockingSubscribe()`, `blockingFirst()`, `blockingLast()` 等阻塞方法内部会处理 `request()`，但它们会阻塞调用线程，需谨慎使用。

*   **性能考量：**
    *   `Flowable` 由于需要处理 `request()` 计数和相关的原子操作，其基线开销比 `Observable` 略高。但在大多数场景下，这点差异可以忽略不计。
    *   不当的背压策略（如无限制的缓冲）可能导致严重的性能问题（内存消耗）。
    *   频繁的小批量 `request()` (e.g., `request(1)` in `onNext`) 比一次请求一大批 (`request(128)`) 开销更大，但能提供更精细的流速控制。需要权衡。

*   **总结：**
    `Flowable` 是 RxJava 2+ 中处理数据流（特别是大量或高速流）的基石，其核心是**背压支持**。通过 `Subscriber`、`Subscription` 和 `request(n)` 协议，它实现了消费者驱动的流量控制，有效防止了因生产者过快导致下游崩溃的问题。理解背压策略、相关操作符（`onBackpressureXxx`, `observeOn`, `create`, `generate`）以及其内部工作机制对于编写健壮、高效的响应式应用程序至关重要。在不确定时，优先选择 `Flowable` 通常是更安全的选择。

---
