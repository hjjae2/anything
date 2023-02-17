### 참고
- https://docs.spring.io/spring-framework/docs/6.0.4/reference/html/integration.html#cache
- https://docs.spring.io/spring-boot/docs/current/reference/html/io.html


<br>

## Caching (from spring-boot docs)

> " You can also use the standard JSR-107 (JCache) annotations (such as @CacheResult) transparently. However, we strongly advise you to not mix and match the Spring Cache and JCache annotations. "

Spring Cache 어노테이션과 JCache 어노테이션을 혼합하여 사용하지 않는다.

> " If you do not add any specific cache library, Spring Boot auto-configures a simple provider that uses concurrent maps in memory. When a cache is required (such as piDecimals in the preceding example), this provider creates it for you. The simple provider is not really recommended for production usage ... "

기본적으로 concurrent map 기반의 캐시 프로파이더가 사용되는데, 실제 운영 환경에서는 사용하지 않도록 한다.

캐시 추상화(cache abstraction)은 캐시 프로바이더(실질적인 캐시 저장소)를 제공하지 않는다. 

> *" If you have not defined a bean of type CacheManager or a CacheResolver named cacheResolver (see CachingConfigurer), Spring Boot tries to detect the following providers (in the indicated order): "*
>
> - Generic
> - JCache (JSR-107) (EhCache 3, Hazelcast, Infinispan, and others)
> - Hazelcast
> - Infinispan
> - Couchbase
> - Redis
> - Caffeine
> - Cache2k
> - Simple

<br>

### CacheManagerCustomizer

CacheManager 빈이 완전히 설정되기 전에 CacheManagerCustomizer 를 통해 추가적인 설정이 가능하다.

```kotlin
@Configuration(proxyBeanMethods = false)
class MyCacheManagerConfiguration {

    @Bean
    fun cacheManagerCustomizer(): CacheManagerCustomizer<ConcurrentMapCacheManager> {
        return CacheManagerCustomizer { cacheManager ->
            cacheManager.isAllowNullValues = false
        }
    }
}
```

<br>

### Generic

> *" Generic caching is used if the context defines at least one org.springframework.cache.Cache bean. A CacheManager wrapping all beans of that type is created. "*

<br>

### Simple

> *" If none of the other providers can be found, a simple implementation using a ConcurrentHashMap as the cache store is configured. By default, caches are created as needed, but you can restrict the list of available caches by setting the cache-names property. For instance, if you want only cache1 and cache2 caches, set the cache-names property as follows: "*
> ```kotlin
> spring.cache.cache-names=cache1,cache2
> ```

<br>

### None

> *" When @EnableCaching is present in your configuration, a suitable cache configuration is expected as well. If you need to disable caching altogether in certain environments, force the cache type to none to use a no-op implementation, as shown in the following example: "*
> ```kotlin
> spring.cache.type=none
> ```


`spring.cache.type=none` 설정을 통해 캐시를 비활성화할 수 있다.