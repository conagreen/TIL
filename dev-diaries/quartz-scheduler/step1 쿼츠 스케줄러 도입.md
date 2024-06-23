# 왜 Quartz Scheduler를 선택했을까?

회사에서 정기 결제를 위한 스케줄링 작업을 직접 구현하기 위해 스케줄러를 사용하기로 결정했다. 기능을 구현하기에 앞서 Spring Batch, Spring Scheduler, Quartz Scheduler 각각에 대해 알아본 내용을 기록하고자 한다.

## Spring Batch

> **배치**: 사용자와 상호작용 없이 여러 작업을 미리 정해진 순서에 따라 처리하는 것


배치는 실시간으로 처리하는 것이 아니라 처리할 작업들을 모아 한 번에 일괄 처리하는 방식이다. 
대용량 데이터를 처리하기 위한 다양한 기능을 지원하기 때문에 스케줄러와 배치를 조합하여 많이 사용한다. 
예를 들어, 매일 자정에 하루 동안의 주문 데이터를 집계하여 매출 보고서를 생성하는 등의 작업을 처리한다.

Spring Batch는 스케줄러가 아니기 때문에 이번 고려 대상이 아니었다. 
또한 작업 대상이 대용량 데이터도 아니었기 때문에 스케줄러 단독으로 충분하다고 판단하여 제외했다.

많은 분들이 헷갈려하는 내용이기 때문에 한 번 더 강조하고 넘어가겠다. 배치와 스케줄러는 비교 대상이 아니다. 
**배치는 대량의 데이터를 일괄적으로 처리하는 방식**에 관한 것이고, 
**스케줄러는 특정 작업이 언제 처리되는지를 관리**하는 것이다.

## Spring Scheduler VS Spring Quartz

> **스케줄러**:  특정 시간에 등록된 작업을 자동으로 실행시키는 것


스프링에서 스케줄러를 사용할 때는 주로 Spring Scheduler와 Quartz Scheduler 중 하나를 선택하게 된다. 그럼 어떤 기준으로 라이브러리를 선택하는 것이 좋을지 알아보자.

### Spring Scheduler

Spring Scheduler는 `spring-boot-starter`에 기본적으로 포함되어 있어 추가적인 의존성이 필요 없다. 사용을 위해서는 아래와 같이 `@EnableScheduling` 어노테이션만 추가해주면 된다.

```java
@EnableScheduling
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

가장 큰 장점은 `@Scheduled` 어노테이션을 사용하여 손쉽게 스케줄링 기능을 구현할 수 있다는 점이다. 해당 어노테이션의 속성으로 스케줄러 실행 시간을 설정할 수 있다.

- `fixedDelay`: 작업이 끝난 시점부터 시간을 카운트
- `fixedRate`: 작업의 시작 시점부터 시간을 카운트
- `cron` & `zone`: 크론 표현식으로 특정 시간을 설정하며, 타임존은 생략 가능

스케줄링할 메서드는 다음 조건을 만족해야 한다.

- 반환 타입이 `void`일 것
- 파라미터가 없을 것

```java
public class Scheculer() {
    // 끝나는 시간 기준으로 5000ms 간격으로 실행
    @Scheduled(fixedDelay = 5000)
    public void fixedDelayScheduler() {
        System.out.println("이 작업이 끝나고 나면 다시 5000ms 후에 실행");
    }

    // 시작 시간 기준으로 5000ms 간격으로 실행
    @Scheduled(fixedRate = 5000)
    public void fixedRateScheduler() {
        System.out.println("시작 시간을 기준으로 5000ms 마다 실행");
    }

    // 크론 표현식으로 시간 설정 (타임존 생략 가능)
    @Scheduled(cron = "0 0 * * * ?", zone = "Asia/Seoul")
    public void cronExpressionScheduler() {
        System.out.println("한국 서울 시간을 기준으로 매일 자정마다 실행");
    } 
}
```

Spring Scheduler는 기본적으로 단일 스레드로 동작한다. 
따라서 1번 스케줄이 완료되지 않으면 2번 스케줄이 시작 시간이 되어도 실행되지 않는다. 
비동기 방식으로 동작되길 원하면 `@EnableAsync` 어노테이션을 사용하여 설정할 수 있다.

### Quartz Scheduler

Quartz Scheduler는 자바에서 사용할 수 있는 오픈소스 Job Scheduling 라이브러리로, Spring Scheduler에 비해 다양한 기능을 제공한다.

- **클러스터링 기능 지원**: 메모리 기반 스케줄러뿐만 아니라 DB 기반 스케줄러도 지원하여 다중 서버 간 스케줄링이 가능하다. 즉, DB를 기반으로 스케줄러 간 클러스터링 기능을 지원한다.
- **고가용성**: 한 서버가 셧다운되더라도 다른 서버에 의해 Job이 실행되어 다운 타임이 없다.
- **확장성**: Quartz 서버를 구동하면 자동으로 DB에 스케줄 인스턴스로 등록된다. 셧다운된 서버는 다른 서버에 의해 DB에서 삭제된다.
- **로드밸런싱**: 클러스터 구성을 통해 여러 Job이 여러 서버에 분산되어 실행된다. 단, Quartz에서는 최소한의 구현으로 랜덤 알고리즘을 제공한다.
- **스케줄러 실패에 대한 후처리 기능**: 스케줄러가 실패할 경우 이를 처리하는 로직을 정의할 수 있다.
- **이벤트 처리 (Job, Trigger)**: Job과 Trigger의 실행 전후 이벤트를 처리할 수 있다.

Quartz는 Spring Scheduler에 비해 지원 기능이 많고 세밀한 제어가 가능하기 때문에 러닝 커브가 다소 있다. 특히 클러스터링 기능을 지원하기 때문에 다중 서버 환경에서 안전한 스케줄링이 가능하다. Quartz 사용법에 대한 자세한 내용은 다음 포스트에서 다룰 예정이다.

## 결론

결과적으로 본인은 Quartz Scheduler를 선택했다. 스케줄러를 도입하려는 서비스가 단일 인스턴스로 구성되어 있으면서 단순한 스케줄링 작업만 필요하다면 Spring Scheduler를 추천한다. 나 역시 초기에는 Quartz를 크게 고려하지 않았지만, 운영하는 서비스가 서버 이중화가 되어 있어 클러스터링 지원이 되는 Quartz Scheduler를 택했다. Quartz를 이용해 스케줄링 작업을 구현하는 방법은 다음 포스트에서 자세히 다루겠다.