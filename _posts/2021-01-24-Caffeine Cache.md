---
title: "Caffeine Cache 使用"
last_modified_at: 2021-01-24T15:05:02-05:00
categories:
  - Blog
tags:
  - annotation
  - meta annotation
# link: https://foresx.github.io/blog
header:
  overlay_image: /assets/images/banner.jpg
  overlay_filter: 0.5

---


1. get()方法才会触发 loadFunction

2. refreshAfterWrite 可以吞掉loadFunction里面的异常,会立即返回之前的结果而不是刷新的结果.

3. 作为 backup 的时候, 可以把 expire 的时间设置长点.

理解上把 expireAfterWrite 这里是会把缓存给驱逐出去,再次调用的时候,如果使用 getIfPresent 就会返回 null, 如果调用的是 get 方法就会重新 load.
而 refreshAfterWrite 不会把之前的缓存给抛弃掉, 单纯的进行更新. 如果load 方法中有异常则不会更新,并且会返回之前正确的结果.

```java
@Slf4j
public class CaffeineCacheTest {

  private int globeCount;

  private final FakeTicker ticker = new FakeTicker(); // Guava的测试库

  private final LoadingCache<String, String> expireCache = Caffeine.newBuilder()
      .expireAfterWrite(1, TimeUnit.HOURS)
      .refreshAfterWrite(5, TimeUnit.MINUTES)
      .ticker(ticker::read)
      .build(this::mockException);

  private final LoadingCache<String, String> expireAndRefreshCache = Caffeine.newBuilder()
      .expireAfterWrite(1, TimeUnit.HOURS)
      .refreshAfterWrite(5, TimeUnit.MINUTES)
      .ticker(ticker::read)
      .build(this::mockExceptionWithTimeStamp);


  @BeforeEach
  void before() {
    globeCount = 0;
  }

  @Test
    // 结论, get 方法才会通过 load 去加载 method
  void testCaffeineCacheExpire() {
    String str = "Hello world!";
    // 预热
    // 并不会真正触发方法
    expireCache.getIfPresent(str);
    String result = expireCache.get(str);

    assert result.equals(str);
    ticker.advance(2, TimeUnit.HOURS);
    // 这一次失效了
    String secondResult = expireCache.getIfPresent(str);
    assert null == secondResult;

    String thirdResult = expireCache.get(str);
    assert str.equals(thirdResult);
  }

  @Test
    // refresh 抛出异常但是不影响使用
  void testCaffeineCacheRefresh() {
    String str = "Hello world!";

    String first = expireAndRefreshCache.get(str);
    String cachedString = expireAndRefreshCache.get(str);
    ticker.advance(10, TimeUnit.MINUTES);
    String second = expireAndRefreshCache.get(str);
    ticker.advance(10, TimeUnit.MINUTES);
    String third = expireAndRefreshCache.get(str);
    ticker.advance(10, TimeUnit.MINUTES);
    // 这一次没有进行调用缓存,直接返回当前值
    String fourth = expireAndRefreshCache.get(str);
    ticker.advance(10, TimeUnit.MINUTES);
    String fifth = expireAndRefreshCache.get(str);

    // 判断这个时候还没刷新,拿到的是缓存的数据
    Assertions.assertThat(first).isEqualTo(cachedString);
    // 判断第三次进行了刷新
    Assertions.assertThat(second).isNotEqualTo(third);
    // 这个时候会拿到的是第三次的结果,第四次第五次都失败了,所以这里会拿到的都是第三次还没过期的内容
    Assertions.assertThat(third).isEqualTo(fourth).isEqualTo(fifth);
    Assertions.assertThat(globeCount).isEqualTo(2);
  }

  // 第四次以后就会抛出异常
  private String mockException(String key) {
    log.debug("重新加载缓存");
    if (globeCount >= 2) {
      throw new RuntimeException("Can't perform this action!");
    } else {
      globeCount++;
      return key;
    }
  }

  // 第四次以后就会抛出异常
  private String mockExceptionWithTimeStamp(String key) {
    log.debug("重新加载缓存");
    if (globeCount >= 2) {
      throw new RuntimeException("Can't perform this action!");
    } else {
      globeCount++;
      return key + Instant.now();
    }
  }
}
```