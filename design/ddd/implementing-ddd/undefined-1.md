# 바운디드 컨텍스트의 통합

## 통합의 기본

## 레스트풀 리소스를 사용한 통합

## 메시징을 사용한 통합

* 메시지 기반의 접근법을 사용한 통합은 한 시스템이 의존적인 시스템으로부터 좀 더 높은 수준의 자율성을 달성할 수  있도록 해줌
* 메시징 인프라가 기능적으로 유지되는 한, 메시지는 한 시스템을 사용할 수 없는 상황에서도 전달될 수 있



* DDD를 활용해 시스템을 자율성 있게 만드는 한 가지 방법은 **도메인 이벤트**를 사용하는 것
* 한 시스템에 뭔가 중요한 일이 발생하면 이와 관련된 이벤트를 만듬
* 각 시스템 안에는 이런 이벤트가 서너 개 이상 있을 수 있고, 이런 각각의 이벤트를 기록하기 위해 고유한 종류의 이벤트를 생성함
* 이트가 발생하면 메시징 메커니즘을 통해 이해 당사자들에게 발행됨

### 제품 소유자와 팀 멤버의 정보를 계속해서 제공받는 것

* 애자일 프로젝트 관리 컨텍스트는 서비스를 구독하는 각 테넌트의 스크럼 제품 소유자와 팀 멤버의 그룹을 관리해야함
* 제품 소요자는 언제든 새로운 제품을 생성하고 팀 멤버를 해당 팀에 할당할 수 있음
* 어떻게 스크럼 프로젝트 관리 애플리케이션은 누가 이러한 역할을 수행하는지 알 수 있을까?
* 정답은 스스로만의 힘으로 하지 않는다는 데 있음(?)



* 실제로 애자일 프로젝트 관리 컨텍스트는 이런 역할이 자연스럽고 적절하게 ID와 액세스 컨텍스트에 의해 관리되도록 해줌
* 이 시스템에서 스크럼 서비스를 구하는 각 테넌트는 <mark style="color:blue;">`ScrumProductOwner`</mark>와 <mark style="color:blue;">`ScrumTeamMembe`</mark>의 두 <mark style="color:blue;">`Role`</mark> 인스턴스를 가짐
* 이런 역할을 수행해야 하는 각 <mark style="color:blue;">`User`</mark>는 역할에 맞게 할당됨
* 다음은 ID와 액세스 컨텍스트에서 <mark style="color:blue;">`User`</mark>를 <mark style="color:blue;">`Role`</mark>에 할당하는 역할을 관리하는 애플리케이션 메서드임

```java
package com.saasovation.identityaccess.application;
...
public class AccessService ... {
    ...
    @Transactional
    public void assignUserToRole(AssignUserToRoleCommand aCommand) {
        TenantId tenantId = new TenantId(aCommand.getTenantId());
        
        User user = this.userRepository.userWithUsername(tenantId, aCommand.getUsername());
        
        if (user != null) {
            Role role = this.roleRepository.roleNamed(tenantId, aCommand.getRoleName());
            if (role != null) {
                role.assignUser(user);
            }
        }
    }
    ...
}
```



* 그런데 여기선 누가 <mark style="color:blue;">`ScrumTeamMember`</mark>나 <mark style="color:blue;">`ScrumProductOwner`</mark>의 역할을 수행하는지 앶일 프로젝트 관리 컨텍스트가 알 수 있도록 도와주는 건 무엇일까?
* <mark style="color:blue;">`Role`</mark>의 메서드 <mark style="color:blue;">`assignUser()`</mark>가 완료되고 나면, 마지막 임무로서 이벤트를 발행함으로써 이를 해결함



```java
package com.saasovation.identityaccess.domain.model.access;
...
public class Role extends Entity {
    ...
    public void assignUser(User aUser) {
        if (aUser == null) {
            throw new NullPointerException("User must not be null.");
        }
        if (!this.tenantId().equals(aUser.tenantId())) {
            throw new IllegalArgumentException("Wrong tenant for this user.");
        }
        
        this.group().addUser(aUser);
        
        DomainEventPublisher.instance().publish(
            new UserAssignedToRole(
                this.tenantId(),
                this.name(),
                aUser.username(),
                aUser.person().name().firstName(),
                aUser.person().name().lastName(),
                aUser.person().name().emailAddress(),
            );
        )
    }
    ...
}
```



* <mark style="color:blue;">`User`</mark>의 이름과 이메일 주소 속성이 강화된 <mark style="color:blue;">`UserAssignedToRole`</mark> 이벤트는 결과적으로 모든 이해 당사자들에게 전달됨
* 애자일 프로젝트 관리 컨텍스트가 이벤트를 받으면, 새로운 <mark style="color:blue;">`TeamMember`</mark>나 <mark style="color:blue;">`ProductOwner`</mark>가 해당 모델 안에서 만들어짐
* 이는 난처하게 어려운 유스케이스는 아니지만, 보이는 것보다는 더 많은 세부사항을 관리해야함
* 좀 더 나눠서 살펴볼 것임



* RabbitMQ로부터 알림을 수신하는 데에는 재사용이 충분히 가능한 몇 가지 측면들이 있음(?)
* 우리는 이미 RabbitMQ 자바 클라이언트를 좀 더 쉽게 사용하도록 도와주는 단순한 객체지향 라이브러리를 가지고 있음
* 이제 익스체인지(exchange) 큐 컨슈머의 역할을 아주 간단히 수행할 수 있도록 해주는 좀 더 단순한 클래스를 추가해보자



```java
package com.saasovation.common.port.adapter.messaging.rabbitmq;
...
public abstract class ExchangeListener {
    
    private MessageConsumer messageConsumer;
    private Queue queue;
    
    public ExchangeListener() {
        super();
        
        this.attachToQueue();
        this.registerConsumer();
    }
    
    protected abstract String exchangeName();
    
    protected abstract void filteredDispatch(String aType, String aTextMessage);
    
    protected abstract String[] listensToEvents();
    
    protected String queueName() {
        return this.getClass().getSimleNmae();
    }
    
    private void attachToQueue() {
        Exchange exchage = Exchange.fanOutInstance(
            ConnectionSettings.instance(),
            this.exchangeName(),
            true
        );
        
        this.queue = Queue.individualExchangeSubscriberInstance(
            exchagne,
            this.exchangeName() + "." + this.queueName()
        );
    }
    
    private Queue queue() {
        return this.queue;
    }
    
    private void registerConsumer() {
        this.messageConsumer = MessageConsumer.instance(this.queue(), false);
        
        this.messageConsumer.receiveOnly(
            this.listensToEvents(),
            new MessageListener(MessageListener.Type.TEXT) {
                @Override
                public void handleMessage(
                    String aType,
                    String aMessageId,
                    Date aTimestamp,
                    String aTextMessage,
                    long aDeliveryTag,
                    boolean isRedelivery
                ) throws Exception {
                    filterDispatch(aType, aTextMessage);
                }
            }
        );
    }
    
    
}
```



* <mark style="color:blue;">`ExchangeListener`</mark>는 구체적 리스너 서브클래스가 재사용하는 추상 기본 클래스임
* 구체적 서브클래스는 추상 기본 클래스를 확장한 후 아주 약간의 코드만 추가하면 됨
* 먼저 기본 클래스의 빈 생성자가 호출되도록 해야 하며 이는 어떤 상화에서도 필요함
* 이어서 <mark style="color:blue;">`exchangeName()`</mark>과 <mark style="color:blue;">`filteredDispatch()`</mark>와 <mark style="color:blue;">`listensToEvents()`</mark>의 세 가지 추상 메서드를 구현하는 일만 는데, 이 중 두 가지는 아주 간단히 구현할 수 있음



* <mark style="color:blue;">`exchangeName()`</mark>을 구현하기 위해선 구체적 리스너가 알림을 소비할 `String` 익스체인지의 이름을 반환하는 부분만 있으면 됨
* 추상 메서드 `listensToEvents()`를 구현하기 위해선 받고자 하는 알림 타입의 `String[]`을 응답해야함
* 많은 리스너가 오직 한 가지 타입의 알림만을 소비할 것이, 그래서 오직 한 가지 컴포넌트의 배열만을 돌려주게 됨(?)
* 마지막 메서드인 `filteredDispatch()`는 세 가지 중 가장 복잡한데, 수신된 메시지를 처리하는 무거운 책임을 갖고 있기 때문임
* 이떻게 작동하는지 살펴보기 위해 `UserAssignedToRole`을 위한 알림(이벤트를 담고 있는)의 리스너를 살펴보자

```java
package com.saasovation.ailepm.infrastructure.messaging;
...
public class TeamMemberEnableListener extends ExchangeListener {
    
    @Autowired
    private TeamService teamService;
    
    public TeamMemberEnableListener() {
        super();
    }    
    
    @Override
    protected String exchangeName() {
        return Exchanges.IDENTITY_ACCESS_EXCHANGE_NAME;
    }
    
    @Override
    protected void filteredDispatch(String aType, String aTextMessage) {
        NotificationReader reader = new NotificationReader(aTextMessage);
        
        String roleName = reader.eventStringValue("roleName");
        
        if (!roleName.equals("ScrumProductOwner") && !roleName.equals("ScrumTeamMember") {
            return;
        }
        
        String emailAddress = reader.eventStringValue("emailAddress");
        String firstName = reader.eventStringValue("firstName");
        String lastName = reader.eventStringValue("lastName");
        String tenantId = reader.eventStringValue("tenantId.id");
        String username = reader.eventStringValue("username");
        Date occurredOn = reader.occurredOn();
        
        if (roleName.equals("ScrumProductOwner")) {
            this.teamService.enableProductOwner(
                new EnableProductOwnerCommand(
                    tenantId,
                    username,
                    firstName,
                    lastName,
                    emailAddress,
                    occurredOn
                )
            );
        } else {
            this.teamService.enableTeamMember(
                new EnableTeamMemberCommand(
                    tenantId,
                    username,
                    firstName,
                    lastName,
                    emailAddress,
                    occurredOn
                )
            );
        }
    }
    
    @Override
    protected String[] listensToEvents() {
        return new String[] {"com.saasovation.identityaccess.domain.model.access.UserAssignedToRole"};
    }
}
```



* `ExchangeListener` 기본 생성자가 올바르게 호출되고, `exchageName()`은 ID와 액세스 컨텍스트에 발행된 익스체인지의 이름을 알려주고, `listensToEvents()` 메서드는 이벤트 `UserAssignedToRole`의 완전히 정규화된 클래스 이름을 하나의 항목을 갖는 배열로 반환함
* **발행자와 구독자는 모듈 이름과 클래스 이름을 포함하는 클래스 이름을 포함하는 정규화된 클래스 이름을 사용해야함**
* 이는 다른 바운디드 컨텍스트의 동일하거나 유사한 이름의 이벤트 때문에 발생할 수 있는 모든 충돌이나 모호성을 제거해줌



* 다시 한번 말하지만, `filteredDispatch()`는 행동의 대부분을 포함하고 있음
* 알림을 애플리케이션 서비스 API로 디스패치하기 전에 필터링하기 때문에 이 메서드 이름을 이렇게 지었음
* 여기선 `ScurmProductOwner`와 `ScrumTeamMember`라는 역할에 관한 이벤트가 아닌 `UserAssignedToRole` 타입의 모든 알림을 무시함으로써 디스패치에 앞서 필터링을 수행하고 있음
* 반면에 이벤트를 수신하는데 관심이 있는 역할(처리하려는 역할)이라면, 알림으로부터 `UserAssignedToRole`의 세부사항을 가져와서 `TeamService`라는 애플리케이션 서비스로 디스패치함



* 처음에는 멤버가 단순히 이런 이벤트 중 하나의 결과로서 생성될 것처럼 보
* 그러나 각 `User`가 이런 `Roles` 중 하나에 할당된 할당 해제되며 그 이후에 다시 재할당되기 때문에, 수신한 알림의 `User`가 나타내는 멤버가 이미 존재할 가능성이 있음
* 다음은 `TeamService`가 이런 상황을 어떻게 대응하는지 보여줌

```java
```



### 당신은 책임을 감당할 수 있는가

### 장기 실행 프로세스와 책임의 회피

### 프로세스 상태 머신과 타임아웃 트래커

### 좀 더 복잡한 프로세스 설계하기

### 메시징이나 시스템을 활용할 수 없을&#x20;

## 마무리

