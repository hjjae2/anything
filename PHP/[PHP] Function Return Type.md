PHP7부터 `function` 에 return type 을 명시할 수 있다.

```php
function isTrue(): bool {
    return true;
}
```

<br><br>

### 'function 내부의 return 값' 과 '명시된 type' 이 다를 경우

(암묵적으로) function 내부의 return 값의 타입을 변환한다.

```php
function isTrue(): bool {
    return 1;
}

var_dump(isTrue()); // 출력: bool(true)
```

**`declare(strict_types=1);` 선언을 통해 위와 같은 암묵적인 변환을 막을 수도 있다. 즉, 엄격하게 타입을 체크할 수 있다.**

> 위 선언을 해주면, IDE에서는 곧바로 빨간줄이 생긴다.

<br><br>

### 참고

1. https://wiki.php.net/rfc/return_types
2. https://stackoverflow.com/a/38970809