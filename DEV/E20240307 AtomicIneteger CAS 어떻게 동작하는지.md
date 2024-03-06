## E20240307 AtomicIneteger CAS 어떻게 동작하는지.md
``` java
/**
 * Atomically adds the given value to the current value of a field
 * or array element within the given object {@code o}
 * at the given {@code offset}.
 *
 * @param o object/array to update the field/element in
 * @param offset field/element offset
 * @param delta the value to add
 * @return the previous value
 * @since 1.8
 */
@IntrinsicCandidate
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, offset, v, v + delta));
    return v;
}
```
1. AtomicInteger.getAndIncreament 무한루프여도 괜찮나? timeout 이 없던데?
2. 근데 버전(v)을 계속 업데이트하니까 문제 안되겠다. 낙관적락이네 자원은 메모리를 쓰는.
3. value 를 가져올 때 Volatile 를 쓰는구나. 
4. 가시성만 보장되는 Volatile를 언제, 왜 사용하는지 궁금했었는데 다른 스레드들에서 조작할 때 캐시가 아닌 메인 메모리에서 가져와 최신 값을 명확하게 하려고?
``` java
    /**
     * Atomically updates Java variable to {@code x} if it is currently
     * holding {@code expected}.
     *
     * <p>This operation has memory semantics of a {@code volatile} read
     * and write.  Corresponds to C11 atomic_compare_exchange_strong.
     *
     * @return {@code true} if successful
     */
    @IntrinsicCandidate
    public final native boolean compareAndSetInt(Object o, long offset,
                                                 int expected,
                                                 int x);
```
4. weakCompareAndSetInt 을 따라가면 결국 native
