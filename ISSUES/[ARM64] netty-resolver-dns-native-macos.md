> 로컬 환경 : m1 arm64

<br>

### Caused by: java.io.FileNotFoundException: META-INF/native/libnetty_resolver_dns_native_macos_aarch_64.jnilib

Spring Cloud Gateway + Mock API 구성 후 간단한 테스트 시 다음과 같은 오류가 발생한다.

```log
java.lang.reflect.InvocationTargetException: null
    at java.base/jdk.internal.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method) ~[na:na]
    at java.base/jdk.internal.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62) ~[na:na]

    ...
    
    at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) ~[netty-common-4.1.78.Final.jar:4.1.78.Final]
    at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30) ~[netty-common-4.1.78.Final.jar:4.1.78.Final]
    at java.base/java.lang.Thread.run(Thread.java:829) ~[na:na]
Caused by: java.lang.UnsatisfiedLinkError: failed to load the required native library
    at io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider.ensureAvailability(MacOSDnsServerAddressStreamProvider.java:110) ~[netty-resolver-dns-classes-macos-4.1.78.Final.jar:4.1.78.Final]
    at io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider.<init>(MacOSDnsServerAddressStreamProvider.java:120) ~[netty-resolver-dns-classes-macos-4.1.78.Final.jar:4.1.78.Final]
    ... 125 common frames omitted
Caused by: java.lang.UnsatisfiedLinkError: could not load a native library: netty_resolver_dns_native_macos_aarch_64
    at io.netty.util.internal.NativeLibraryLoader.load(NativeLibraryLoader.java:239) ~[netty-common-4.1.78.Final.jar:4.1.78.Final]
    at io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider.loadNativeLibrary(MacOSDnsServerAddressStreamProvider.java:92) ~[netty-resolver-dns-classes-macos-4.1.78.Final.jar:4.1.78.Final]
    at io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider.<clinit>(MacOSDnsServerAddressStreamProvider.java:77) ~[netty-resolver-dns-classes-macos-4.1.78.Final.jar:4.1.78.Final]
    at java.base/java.lang.Class.forName0(Native Method) ~[na:na]
    at java.base/java.lang.Class.forName(Class.java:398) ~[na:na]
    at io.netty.resolver.dns.DnsServerAddressStreamProviders$1.run(DnsServerAddressStreamProviders.java:50) ~[netty-resolver-dns-4.1.78.Final.jar:4.1.78.Final]
    at java.base/java.security.AccessController.doPrivileged(Native Method) ~[na:na]
    at io.netty.resolver.dns.DnsServerAddressStreamProviders.<clinit>(DnsServerAddressStreamProviders.java:46) ~[netty-resolver-dns-4.1.78.Final.jar:4.1.78.Final]
    ... 120 common frames omitted
    Suppressed: java.lang.UnsatisfiedLinkError: could not load a native library: netty_resolver_dns_native_macos
        at io.netty.util.internal.NativeLibraryLoader.load(NativeLibraryLoader.java:239) ~[netty-common-4.1.78.Final.jar:4.1.78.Final]
        at io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider.loadNativeLibrary(MacOSDnsServerAddressStreamProvider.java:95) ~[netty-resolver-dns-classes-macos-4.1.78.Final.jar:4.1.78.Final]
        ... 126 common frames omitted
    Caused by: java.io.FileNotFoundException: META-INF/native/libnetty_resolver_dns_native_macos.jnilib
        at io.netty.util.internal.NativeLibraryLoader.load(NativeLibraryLoader.java:181)
        ... 127 common frames omitted
        Suppressed: java.lang.UnsatisfiedLinkError: no netty_resolver_dns_native_macos in java.library.path: [/Users/leehyunjae/Library/Java/Extensions, /Library/Java/Extensions, /Network/Library/Java/Extensions, /System/Library/Java/Extensions, /usr/lib/java, .]
            at java.base/java.lang.ClassLoader.loadLibrary(ClassLoader.java:2673)
            at java.base/java.lang.Runtime.loadLibrary0(Runtime.java:830)
            at java.base/java.lang.System.loadLibrary(System.java:1873)
            at io.netty.util.internal.NativeLibraryUtil.loadLibrary(NativeLibraryUtil.java:38)
            at io.netty.util.internal.NativeLibraryLoader.loadLibrary(NativeLibraryLoader.java:391)
            at io.netty.util.internal.NativeLibraryLoader.load(NativeLibraryLoader.java:161)
            ... 127 common frames omitted
            Suppressed: java.lang.UnsatisfiedLinkError: no netty_resolver_dns_native_macos in java.library.path: [/Users/leehyunjae/Library/Java/Extensions, /Library/Java/Extensions, /Network/Library/Java/Extensions, /System/Library/Java/Extensions, /usr/lib/java, .]
                at java.base/java.lang.ClassLoader.loadLibrary(ClassLoader.java:2673)
                at java.base/java.lang.Runtime.loadLibrary0(Runtime.java:830)
                at java.base/java.lang.System.loadLibrary(System.java:1873)
                at io.netty.util.internal.NativeLibraryUtil.loadLibrary(NativeLibraryUtil.java:38)
                at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
                at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
                at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
                at java.base/java.lang.reflect.Method.invoke(Method.java:566)
                at io.netty.util.internal.NativeLibraryLoader$1.run(NativeLibraryLoader.java:425)
                at java.base/java.security.AccessController.doPrivileged(Native Method)
                at io.netty.util.internal.NativeLibraryLoader.loadLibraryByHelper(NativeLibraryLoader.java:417)
                at io.netty.util.internal.NativeLibraryLoader.loadLibrary(NativeLibraryLoader.java:383)
                ... 128 common frames omitted
Caused by: java.io.FileNotFoundException: META-INF/native/libnetty_resolver_dns_native_macos_aarch_64.jnilib
    at io.netty.util.internal.NativeLibraryLoader.load(NativeLibraryLoader.java:181) ~[netty-common-4.1.78.Final.jar:4.1.78.Final]
    ... 127 common frames omitted
    Suppressed: java.lang.UnsatisfiedLinkError: no netty_resolver_dns_native_macos_aarch_64 in java.library.path: [/Users/leehyunjae/Library/Java/Extensions, /Library/Java/Extensions, /Network/Library/Java/Extensions, /System/Library/Java/Extensions, /usr/lib/java, .]
        at java.base/java.lang.ClassLoader.loadLibrary(ClassLoader.java:2673) ~[na:na]
        at java.base/java.lang.Runtime.loadLibrary0(Runtime.java:830) ~[na:na]
        at java.base/java.lang.System.loadLibrary(System.java:1873) ~[na:na]
        at io.netty.util.internal.NativeLibraryUtil.loadLibrary(NativeLibraryUtil.java:38) ~[netty-common-4.1.78.Final.jar:4.1.78.Final]
        at io.netty.util.internal.NativeLibraryLoader.loadLibrary(NativeLibraryLoader.java:391) ~[netty-common-4.1.78.Final.jar:4.1.78.Final]
        at io.netty.util.internal.NativeLibraryLoader.load(NativeLibraryLoader.java:161) ~[netty-common-4.1.78.Final.jar:4.1.78.Final]
        ... 127 common frames omitted
        Suppressed: java.lang.UnsatisfiedLinkError: no netty_resolver_dns_native_macos_aarch_64 in java.library.path: [/Users/leehyunjae/Library/Java/Extensions, /Library/Java/Extensions, /Network/Library/Java/Extensions, /System/Library/Java/Extensions, /usr/lib/java, .]
            at java.base/java.lang.ClassLoader.loadLibrary(ClassLoader.java:2673)
            at java.base/java.lang.Runtime.loadLibrary0(Runtime.java:830)
            at java.base/java.lang.System.loadLibrary(System.java:1873)
            at io.netty.util.internal.NativeLibraryUtil.loadLibrary(NativeLibraryUtil.java:38)
            at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
            at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
            at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
            at java.base/java.lang.reflect.Method.invoke(Method.java:566)
            at io.netty.util.internal.NativeLibraryLoader$1.run(NativeLibraryLoader.java:425)
            at java.base/java.security.AccessController.doPrivileged(Native Method)
            at io.netty.util.internal.NativeLibraryLoader.loadLibraryByHelper(NativeLibraryLoader.java:417)
            at io.netty.util.internal.NativeLibraryLoader.loadLibrary(NativeLibraryLoader.java:383)
            ... 128 common frames omitted
```

> 참고 : https://github.com/netty/netty/issues/11020

<br>

`libnetty_resolver_dns_native_macos_aarch_64.jnilib`  파일이 없다고 하니 추가해주면 될 것 같다.

<br>

### libnetty_resolver_dns_native_macos_aarch_64 의존성 추가

```gradle
implementation("io.netty:netty-resolver-dns-native-macos:4.1.79.Final:osx-aarch_64")
```

**의존성 추가 전**

![](../images/[ARM64]%20netty-resolver-dns-native-macos_50.png)

**의존성 추가 후**

![](../images/[ARM64]%20netty-resolver-dns-native-macos_04.png)

<br>

### 참고

> - https://github.com/netty/netty/blob/4.1/resolver-dns-native-macos/pom.xml
> - https://github.com/netty/netty/blob/4ebc4ee66f9202c5b49e4f403cc19a719444d0d9/resolver-dns-native-macos/pom.xml#L217

기본적으로 `arm64` 아키텍처의 경우, 다음과 같이 `oxs-aarch_64` 의존성이 추가되게끔 설정되어 있다.

```gradle
    ...

	if (!"$nettyVersion".endsWithAny("SNAPSHOT")) {
		if (osdetector.classifier == "osx-x86_64" || osdetector.classifier == "osx-aarch_64") {
			api "io.netty:netty-resolver-dns-native-macos:$nettyVersion$os_suffix"
		}
		else {
			api "io.netty:netty-resolver-dns-native-macos:$nettyVersion:osx-x86_64"
		}
	}
	else {
		// MacOS binaries are not available for Netty SNAPSHOT version
		api "io.netty:netty-resolver-dns-native-macos:$nettyVersion"
	}

    ...
```

> 참고 : https://github.com/reactor/reactor-netty/blob/4251a324a967ffdc6a3ceee4830814662b114060/reactor-netty-http/build.gradle#L67

<br>

**`os_suffix` 는 다음 코드에서 설정된다.**

```gradle
    ...

	os_suffix = ""
	if (osdetector.classifier in ["linux-x86_64"] || ["osx-x86_64"] || ["osx-aarch_64"] || ["windows-x86_64"]) {
		os_suffix = ":" + osdetector.classifier
	}

    ...
```

<br>

**osdetector**

`osdetector.classifier` 를 확인하면, `osx-x86_64` 값이 나온다. <br>
= 즉, 아키텍처 디텍딩에서 오류

> 참고 : https://github.com/google/osdetector-gradle-plugin

<br>

**kr.motd.maven:os-maven-plugin:1.7.0**
```
osdetector(google) -> os-maven-plugin (trustin)
```

> 참고 : https://github.com/trustin/os-maven-plugin

<br>

```java

    ...

    protected void detect(Properties props, List<String> classifierWithLikes) {
        log("------------------------------------------------------------------------");
        log("Detecting the operating system and CPU architecture");
        log("------------------------------------------------------------------------");

        final String osName = systemPropertyOperationProvider.getSystemProperty("os.name");
        final String osArch = systemPropertyOperationProvider.getSystemProperty("os.arch");
        final String osVersion = systemPropertyOperationProvider.getSystemProperty("os.version");

        ...
    }

    ...
```

> 참고 : https://github.com/trustin/os-maven-plugin/blob/6bd9cfa16757ac8b81ab1a7f380b0aabc0295c97/src/main/java/kr/motd/maven/os/Detector.java#L72

<br>

```java
  private final class ConfigurationTimeSafeSystemPropertyOperations
      implements SystemPropertyOperationProvider {
    @Override
    public String getSystemProperty(String name) {
      return getProviderFactory().systemProperty(name).forUseAtConfigurationTime().getOrNull();

      // getProviderFactory() : org.gradle.api.internal.provider.DefaultProviderFactory_Decorated@a046f6c
      // getProviderFactory().systemProperty(name) : Provider<String> (DefaultValueSourceProviderFactory.NonConfigurationTimeProvider)
    }

    ...
```

```java
// DefaultValueSourceProviderFactory.class

    public Provider<T> forUseAtConfigurationTime() {
        return new ConfigurationTimeProvider(this.value);
    }
```

```java
// AbstractMinimalProvider.class

    public T getOrNull() {
        return this.calculateOwnValue(ValueConsumer.IgnoreUnsafeRead).orNull();
    }
```

```java
// DefaultValueSourceProviderFactory.class
        protected ValueSupplier.Value<? extends T> calculateOwnValue(ValueSupplier.ValueConsumer consumer) {
            this.vetoAtConfigurationTime();
            return Value.ofNullable(this.value.obtain().get());
        }
```

```java
        public Try<T> obtain() {
            if (this.obtainValueForThe1stTime()) {
                DefaultValueSourceProviderFactory.this.valueObtained(this.obtainedValue());
            }

            return this.value;
        }

```

> 위 코드 진입 시 this.value(org.gradle.internal.Try) 설정된다.

![](../images/[ARM64]%20netty-resolver-dns-native-macos_26.png)

![](../images/[ARM64]%20netty-resolver-dns-native-macos_58.png)

<br>

**DefaultProviderFactory**

```java
    ...

    public Provider<String> systemProperty(Provider<String> propertyName) {
        return this.of(SystemPropertyValueSource.class, (spec) -> {
            ((AbstractPropertyValueSource.Parameters)spec.getParameters()).getPropertyName().set(propertyName);
        });
    }
```

**DefaultValueSourceProviderFactory**

```java
    public abstract static class ValueSourceProvider<T, P extends ValueSourceParameters> extends AbstractMinimalProvider<T> {
        protected final LazilyObtainedValue<T, P> value;

        public ValueSourceProvider(LazilyObtainedValue<T, P> value) {
            this.value = value; // 
        }

        ...

        public ValueSupplier.ExecutionTimeValue<T> calculateExecutionTimeValue() {
            return this.value.hasBeenObtained() ? ExecutionTimeValue.ofNullable(this.value.obtain().get()) : ExecutionTimeValue.changingValue(this);
        }

        protected ValueSupplier.Value<? extends T> calculateOwnValue(ValueSupplier.ValueConsumer consumer) {
            this.vetoAtConfigurationTime();
            return Value.ofNullable(this.value.obtain().get());
        }

        protected abstract void vetoAtConfigurationTime();
    }
```

<br>

**org.gradle.api.provider**
**org.gradle.api.internal.provider**

org.gradle.api.internal.provider.sources.SystemPropertyValueSource