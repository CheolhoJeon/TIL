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
package com.saasovation.agilepm.application;
...
public class TeamService ... {
    
    @Autowired
    private ProductOwnerRepository productOwnerRepository;
    
    @Autowired
    private TeamMemberRepository teamMemberRepository;
    
    ...
    
    @Transactional
    public void enableProductOwner(EnableProdctOwnerCommand aCommand) {
        TenantId tenantId = new TenantId(aCommand.getTenantId());
        
        ProductOwner productOwner = this.productOwnerRepository.productOwnerOfIdentity(tenantId, aCommand.getUsername());
        
        if (productOwner != null) {
            productOwner.enable(aCommand.getOccurredOn());
        } else {
            productOwner = new ProductOwner(
                tenantId,
                aCommand.getUsername(),
                aCommand.getFirstName(),
                aCommand.getLastName(),
                aCommand.getEmailAddress(),
                aCommand.getOccurredOn()
            );
            
            this.prodcutOwnerRepository.add(productOwner);
        }
    }
}
```

### 당신은 책임을 감당할 수 있는가

* 여기까진 모든 점이 괜찮고 좋아 보이며, 충분히 단순해 보임
* `ProductOwner`와 `TeamMember` 애그기거트 타입이 있으며, 이들 각각은 그 이면에 있는 외부 바운디드 컨텍스트의 User에 관한 정보를 일부 보관하도록 설계되었음
* 하지만 이런 방식으로 애그리거트를 설계함으로써 얼마나 많은 책임을 지우게 되는지 알고 있는가??



* 협업 컨텍스트 내에서 팀이 단지 비슷한 정보를 갖는 불변하는 값 객체를 생성하기로 결정했다는 점을 상기해보자
  * '부패 방지 계층을 통한 REST 클라이언트의 구현' 참
* 값들이 불변하기 때문에 이 팀은 공유된 정보를 최신으로 유지할 걱정은 전혀 할 필요가 없음
* 물론 이런 이점에서 오는 단점도 있는데, 공유된 정보의 일부가 업데이트됐을 때 협업 컨텍스트는 절대 과거에 생성된 관련 객체를 업데이트하지 않음(?)
* 즉 애자필 프로젝트 관리 팀은 상충점의 반대편을 선택한 셈임(?)



* 그러나 애그리거트의 정보를 최신으로 유지하는 데에는 몇 가지 문제가 있음
* 그 이유는 무엇일까?
* 그저 단순하게 ProductOwner와 TeamMember 인스턴스에 해당하는 User 인스턴스의 변화를 반영하는 추가적인 이벤트 운반 알림을 받으면 안될까?
* 그렇다. 그렇게 하면 되고 그렇게 해야만 함
* **그러나 우리가 메시징 인프라를 사용하고 있다는 점은 눈에 보이는 수준보다 더 큰 어려움을 초래함**

&#x20;

*



### 장기 실행 프로세스와 책임의 회피

{% file src="../../../.gitbook/assets/DDD_13.drawio" %}

> 이번 절에서는 여러 바운디드 컨텍스트에 걸쳐 실행되는 장기 실행 프로세스를 메시징을 통해 어떻게 연계하는지 살펴본다

* 예제에서는 제품을 생성하는 유스케이스를 살펴볼 것임
* 전제 조건: 협업 기능이 활성화되어 있어야한다(옵션을 구입했다).
  * 사용자는 **제품(Product)** 설명 정보를 제공한다.
  * 사용자는 팀 **토론(Dicussion)**에 대한 의사를 표시한다.
  * 사용자는 정의된 제품을 만들도록 요청한다.
  * 시스템은 포럼과 토론이 들어간 제품을 만든다.

![애자일 관리 컨텍스트(Agile PM)과 협업 컨텍스트(Collaboration)이 존재한다](../../../.gitbook/assets/DDD\_13.drawio.png)



> 먼저 Product를 만드는데 사용된 애플리케이션 서비스를 살펴보자

```java
package com.saasovation.agilepm.application;

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private ProductOwnerRepository productOwnerRepository;

    //...

    @Transactional
    public String newProductWithDiscussion(NewProductCommand aCommand) {
        // Product 생성자를 호출
        return this.newProductWith(
                aCommand.getTenantId(),
                aCommand.getProductOwnerId(),
                aCommand.getName(),
                aCommand.getDescription(),
                this.requestDiscussionIfAvailable()
        );
    }

    // ...

}
```

> newProductWith()은 Product의 생성자를 호출한다

```java
package com.saasovation.agilepm.model.product;

public class Product extends ConcurrencySafeEnity {

    // ...

    // ProductService.newProductWith()에 의해 호출되며
    // 협업 옵션이 사전에 구입되었다면, DiscussionAvailability.REQUESTED가 전달됨
    public Product(
        TenantId aTenantId,
        ProductId aProductId,
        ProductOwnerId aProductOwnerId,
        String aName,
        String aDescription,
        // READY, ADD_ON_NOT_ENABLED, NOT_REQUESTED, REQUESTED
        DiscussionAvailability aDiscussionAvailability
    ) {
        this();

        this.setTenantId(aTenantId);
        this.setProdctId(aProductId);
        this.setProductOwnerId(aProductOwnerId);
        this.setName(aName);
        this.setDescription(aDescription);

        this.setDiscussion(ProductDiscussion.fromAvailability(aDiscussionAvailability));

        // 도메인 이벤트를 발행
        DomainEventPublisher
            .instance()
            .publish(new ProductCreated(
               this.tenantId(),
               this.productId(),
               this.productOwnerId(),
               this.name(),
               this.description(),
               // True이면 장기 실행 프로세스 시작
               // 단, false라고 해서 이벤트가 발행되지 않는 것이 아님
               // false이면 이벤트가 무시될 뿐임
               this.discussion.availability().isRequested()
            ));
    }

    // ...

}
```

> 다음은 Product 생성자에서 호출하는 ProductDiscussion 팩토리 메서드이다

```java
package com.saasovation.agilepm.model.product;

public class ProductDiscussion implements Serializable {

    // ...

    public static ProductDiscussion fromAvailability(DiscussionAvailablity anAvailability) {
        if (anAvailability.isReady()) {
            throw new IllegalArgumentException("Cannot be created ready.");
        }

        DiscussionDecriptor descriptor = new DiscussionDescriptor(
            DiscussionDescriptor(DiscussionDescriptor.UNDEFIEND_ID)
        );

        return new ProductDiscussion(descriptor, anAvailability);
    }

    // ...

}

```

> 앞에서는 제품 생성과 함께 토론이 요청되는 케이스를 살펴보았음\
> 여기서는 제품이 기존에 생성된 상태에서 토론을 요청하는 케이스를 살펴보자

```java
package com.saasovation.agilepm.model.product;

public class Product extends ConcurrencySafeEnity {

    // ...

    // Product만 생성된 후에, 따로 토론을 요청할 때 호
    public void requestDiscussion(DiscussionAvailability aDiscussionAvailability) {
        if (!this.discussion().availability().isReady()) {
            this.setDiscussion(ProductDiscussion.fromAvailability(aDiscussionAvailability));
        }

        DomainEventPublisher
            .instance()
            // ProductDiscussionRequested는 ProductCreated와 동일한 속성을 가지며,
            // 두 이벤트 모두 장기 실행 프로세스의 트리거로서 활용됨
            .publish(new ProductDiscussionRequested(
                this.tenantId(),
                this.productId(),
                this.productOwnerId(),
                this.name(),
                this.description(),
                // 마찬가지로, True이면 장기 실행 프로세스 시작
                this.discussion.availability().isRequested()
            ));
    }

    // ...

}
```

> ProductCreated 이벤트와 ProductDiscussionRequested 이벤트를 처리하는 리스너를 살펴보자

```java
```



### 프로세스 상와 타임아웃 트래커

### 좀 더 복잡한 프로세스 설계하기

### 메시징이나 시스템을 활용할 수 없을&#x20;

## 마무리

