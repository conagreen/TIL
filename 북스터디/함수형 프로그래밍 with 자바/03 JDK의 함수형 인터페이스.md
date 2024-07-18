# 03. JDK의 함수형 인터페이스
자바는 모든 것에 구체적인 타입을 필요로 하는 엄격한 언어이다. 
이런한 이유로 JDK는 함수형 도구를 시작하는 데 도움이 되도록 `java.util.function` 패키지를 통해 40개 이상의 함수형 인터페이스를 제공한다.

3장에서는 가장 중요한 함수형 인터페이스를 소개하고 변형이 왜 여러 가지 존재하는지에 대해 설명한다.

## 네 가지 함수형 인터페이스
`java.util.function` 패키지의 40개 이상의 함수형 인터페이스는 아래 네 가지 주요 카테고리로 나뉜다.

### Function
`Function<T, R>`: T 타입의 인수를 받아 R 타입의 결과를 반환한다.

```java
public class FunctionExample {
    public static void main(String[] args) {
        Function<Integer, String> intToString = i -> "Number is " + i;
        System.out.println(intToString.apply(5));  // Number is 5
    }
}
```

### Consumer
`Consumer<T>`: T 타입의 인수를 받아 어떤 동작을 수행하지만 결과를 반환하지는 않는다.
```java
public class ConsumerExample {
    public static void main(String[] args) {
        Consumer<String> print = s -> System.out.println(s);
        print.accept("Hello, World!");  // Hello, World!
    }
}
```

### Supplier
`Supplier<T>`: 인수를 받지 않고 T 타입의 결과를 반환한다.
```java
public class SupplierExample {
    public static void main(String[] args) {
        Supplier<Double> randomValue = () -> Math.random();
        System.out.println(randomValue.get());  // 랜덤 값 출력
    }
}
```
Supplier는 종종 지연 실행에 사용된다. 예를 들어 비용이 많이 드는 작업을 Supplier로 래핑하고 필요할 때에만 get을 호출하는 경우가 있다.


### Predicate
`Predicate<T>`: T 타입의 인수를 받아 boolean 결과를 반환한다.
```java
public class PredicateExample {
    public static void main(String[] args) {
        Predicate<Integer> isEven = i -> i % 2 == 0;
        System.out.println(isEven.test(4));  // true
        System.out.println(isEven.test(5));  // false
    }
}
```
Predicate는 필터 메서드와 같이 의사 결정에 주로 사용된다.

## 함수형 인터페이스 변형이 많은 이유
앞에서 살펴본 네 가지 함수형 인터페이스보다 더 특화된 변형도 존재한다.
이런 다양한 타입은 자바가 람다를 도입하는 과정에서 하위 호완성을 유지하기 위해 필수적이다.


### 함수 아리티
함수의 아리티(arity)는 함수가 받을 수 있는 인수의 수를 의미한다. 
JDK는 다양한 함수 아리티를 지원하기 위해 여러 가지 변형된 인터페이스를 제공한다. 예를 들어 BiFunction<T, U, R>는 두 개의 인수를 받고 하나의 결과를 반환합니다.

더 높은 아리티를 추가할 수도 있다. 하지만 이런 방식을 사용하지 않는 것을 권장한다. 
함수형 인터페이스는 정적 및 기본 메서드를 통해 추가 기능을 많이 제공한다. 따라서 이러한 기능을 사용하는 것이 최상의 호환성과 이해하기 쉬운 사용 패턴을 보장한다.

### 원시 타입
박싱된 객체(Integer, Double 등)는 성능 상의 이유로 비효율적일 수 있다. 
JDK는 원시 타입을 지원하는 함수형 인터페이스를 제공하여 이 문제를 해결한다. 
예를 들어, IntFunction<R>은 int를 입력으로 받아 R 타입을 반환한다.

## 함수 합성
함수형 프로그래밍의 강력한 기능 중 하나는 함수 합성이다. 
JDK의 함수형 인터페이스는 andThen, compose 등의 기본 메서드를 통해 함수를 합성할 수 있는 기능을 제공한다.

## 함수형 지원 확장
### 기본 메서드 추가
기본 메서드를 통해 인터페이스에 새로운 메서드를 추가하면서도 기존 구현과의 호환성을 유지할 수 있다.

인터페이스의 계약을 변경하고 이를 구현하는 모든 타입에 새로운 메서드를 추가하는 대신 기본 메서드를 사용하여 '상식적인' 구현을 제공할 수 있다.
이렇게 하면 코드의 하위 호환성을 유지할 수 있다. 인터페이스 자체만 변경되었기 때문에 인터페이스를 구현하는 모든 타입은 필요한 경우에 자체적으로 더 적합한 구현을 생성할 수 있다.

### 함수형 인터페이스 명시적으로 구현하기
익명 클래스나 람다식을 사용하지 않고 함수형 인터페이스를 명시적으로 구현할 수도 있다. 

인터페이스는 다중 상속을 허용하기 때문에 함수형 인터페이스를 추가하는 것은 문제가 되지 않는다. 함수형 인터페이스의 SAM은 실제 작업을 수행하는 메서드를 호출하는 간단한 메서드이다.
이렇게 함으로써 어떤 커맨드도 변경할 필요 없이 모든 커맨드들이 `Supplier<String>`를 인수로 받는 어떤 고차 함수와도 호환성을 갖추게 된다.
이 경우 메서드 참조를 브리지로 사용하지 않아도 된다.

### 정적 헬퍼 생성하기


## 회고
함수 함성 너무 어렵다..
