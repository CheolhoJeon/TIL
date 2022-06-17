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

> 이번 절에서는 여러 바운디드 컨텍스트에 걸쳐 실행되는 장기 실행 프로세스를 메시징을 통해 어떻게 연계하는지 살펴본다

* 예제에서는 제품을 생성하는 유스케이스를 살펴볼 것임
* 전제 조건: 협업 기능이 활성화되어 있어야한다(옵션을 구입했다).
  * 사용자는 **제품(Product)** 설명 정보를 제공한다.
  * 사용자는 팀 **토론(Dicussion)**에 대한 의사를 표시한다.
  * 사용자는 정의된 제품을 만들도록 요청한다.
  * 시스템은 포럼과 토론이 들어간 제품을 만든다.



![애자일 관리 컨텍스트(Agile PM) 협업 컨텍스트(Collaboration)이 존재한다](../../../.gitbook/assets/DDD\_13.drawio-2.png)



> 먼저 <mark style="color:blue;">`Product`</mark>를 만드는데 사용된 애플리케이션 서비스를 살펴보자

```java
package com.saasovation.agilepm.application;

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private ProductOwnerRepository productOwnerRepository;

    //...

    // 제품(Product) 생성과 동시에 토론(Discussion)을 '함께' 요청하는 오퍼레이션
    @Transactional
    public String newProductWithDiscussion(NewProductCommand aCommand) {
        // Product 생성자를 호출
        return this.newProductWith(
                aCommand.getTenantId(),
                aCommand.getProductOwnerId(),
                aCommand.getName(),
                aCommand.getDescription(),
                // 토론 생성이 가능한지에 대한 여부를 나타내는 프로퍼티를 반환
                this.requestDiscussionIfAvailable()
        );
    }

    // ...

}
```

> <mark style="color:blue;">`newProductWith()`</mark>은 <mark style="color:blue;">`Product`</mark>의 생성자를 호출한다

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

> 다음은 <mark style="color:blue;">`Product`</mark> 생성자에서 호출하는 <mark style="color:blue;">`ProductDiscussion`</mark> 팩토리 메서드이다

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

> <mark style="color:blue;">`ProductCreated`</mark> 이벤트와 <mark style="color:blue;">`ProductDiscussionRequested`</mark> 이벤트를 처리하는 리스너를 살펴보자

```java
package com.saasovation.agilepm.infrastructure.messaging;

public class ProductDiscussionRequestedListener extends ExchangeListener {
    // ...

    private static final String COMMAND = "com.saasovation.collaboration.discussion.CreateExclusiveDiscussion";

    @Override
    protected String exchangeName() {
        return Exchanges.AGILEPM_EXCHANGE_NAME;
    }

    @Override
    protected String [] listensToEvents() {
        return new String[] {
            "com.saasovation.agilepm.domain.model.product.ProductCreated",
            "com.saasovation.agilepm.domain.model.product.ProductDiscussionRequested",
        };
    }

    @Override
    protected void filteredDispatch(String aType, String aTextMessage) {
        NotificationReader reader = new NotificationReader(aTextMessage);

        // requestingDiscussion의 값이 거짓이라면 이벤트 무시
        // 그렇지 않은 경우, 이벤트의 상태로부터 CreateExclusiveDiscussion 커맨드를 만들어서 협업 컨텍스트의 메시지 익스체인지로 보냄
        if (!reader.eventBooleanValue("requestingDiscussion")) {
            return;
        }

        // 협업 컨텍스트로 커맨드를 보낸다
        Properties parameters = this.parametersFrom(reader);
        PropertiesSerializer serializer = PropertiesSerializer.instance();

        String serialization = serializer.serialize(parameters);
        String commandId = this.commandIdFrom(parameters);

        this.messageProducer()
            .send(
                serialization,
                MessageParameters.durableTextParameters(
                    COMMAND, commandId, new Date()
                )
            )
            .close();
    }

    // ...
}

```



> 이제 협업 컨텍스트의 <mark style="color:blue;">`ExclusiveDiscussionCreationListener`</mark>를 살펴보

```java
package com.saasovation.collaboration.infrastructure.messaging;

public class ExclusiveDiscussionCreationListener extends ExchangeListener {

    @Autowired
    private ForumService forumService;
    
    // ...

    @Override
    protected void filteredDispatch(String aType, String aTextMessage) {
        NotificationReader reader = new NotificationReader(aTextMessage);

        String tenantId = reader.eventStringValue("tenantId");
        String exclusiveOwnerId = reader.eventStringValue("exclusiveOwnerId");
        String forumSubject = reader.eventStringValue("forumTitle");
        String forumDescription = reader.eventStringValue("forumDescription");
        String discussionSubject = reader.eventStringValue("discussionSubject");
        String creatorId = reader.eventStringValue("creatorId");
        String moderatorId = reader.eventStringValue("moderatorId");

        // 관리 컨텍스트에서 전달된 커맨드를 수신하면 애플리케이션 서비스인 ForumSerivce를 호출
        forumService.startExclusiveForumWithDiscussion(
            tenantId,
            creatorId,
            moderatorId,
            forumSubject,
            forumDescription,
            discussionSubject,
            exclusiveOwnerId
        );
    }

}

```



* <mark style="color:blue;">`ForumService`</mark>는 <mark style="color:blue;">`Discussion`</mark> 인스턴스를 생성하며, 생성된 인스턴스는 <mark style="color:blue;">`DiscussionStarted`</mark> 이벤트를 <mark style="color:blue;">`COLLABORATION_EXCHANGE`</mark>로 정의된 익스체인지를 통해 발생함
* 이는 애자일 프로젝트 관리 컨텍스트 내의 <mark style="color:blue;">`DiscussionStartedListener`</mark>가 <mark style="color:blue;">`DiscussionStarted`</mark> 이벤트를 수신하는 이유

> 아래의 예제는 리스너가 이벤트를 수신할 때 어떤 일을 하는지 보여준다

```java
package com.saasovation.agilepm.infrastructure.messaging;

public class DiscussionStartedListener extends ExchangeListener {

    @Autowired
    private ProductService productService;

    // ...
    @Override
    protected String exchangeName() {
        return Exchanges.COLLABORATION_EXCHANGE_NAME;
    }

    @Override
    protected String [] listensToEvents() {
        return new String[] {
            "com.saasovation.collaboration.domain.model.forum.DiscussionStarted"
        };
    }

    @Override
    protected void filteredDispatch(String aType, String aTextMessage) {
        NotificationReader reader = new NotificatinoReader(aTextMessage);

        String tenantId = reader.eventStringValue("tenant.id");
        String productId = reader.eventStringValue("exclusiveOwner");
        String discussionId = reader.eventStringValue("discussionId.id");

        // 애플리케이션 서비스 호
        productService.initiateDiscussion(
            new InitiateDiscussionCommand(tenantId, productId, discussionId)
        );
    }

    // ...
}

```

> <mark style="color:blue;">`initiateDiscussion()`</mark> 메서드는 다음과 같이 동작한다.

```java
package com.saasovation.agilepm.application;

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    // ...

    @Transactional
    public void initiateDiscussion(InitiateDiscussionCommand aCommand) {
        Product product = productRepository.productOfId(
                new TenantId(aCommand.getTenantId()),
                new ProductId(aCommand.getProductId())
        );

        if (product == null) {
            throw new IllegalStateException(
                    "Unknown product of tenant id: "
                    + aCommand.getTenantId()
                    + "and product id: "
                    + aCommand.getProductId());
        }
        
        // DiscussionDescriptor.UNDEFIEND_ID -> aCommand.getDiscussionId()
        product.initiateDiscussion(new DiscussionDescriptor(aCommand.getDiscussionId()));
    }

    // ...

}
```

> 결국 <mark style="color:blue;">`Product`</mark>의 <mark style="color:blue;">`initiateDiscussion()`</mark> 메서드가 수행됨

```java
package com.saasovation.agilepm.model.product;

public class Product extends ConcurrencySafeEnity {

    // ...

    public void initiateDiscussion(DiscussionDescriptor aDescriptor) {
        if (aDescriptor == null) {
            throw new IllegalArgumentException("The descriptor must not be null.");
        }

        if (this.discussion().availability().isRequested()) {
            this.setDiscussion(this.discussion().nowReady(aDescriptor));

            DomainEventPublisher
                .instance()
                .publish(
                    new ProductDiscussionInitiated(
                        this.tenantId(),
                        this.productId(),
                        this.discussion()
                    )
                );
        }
    }

    // ...

}
```



* <mark style="color:blue;">`Product`</mark>의 <mark style="color:blue;">`discussion`</mark> 속성이 여전히 <mark style="color:blue;">`REQUESTED`</mark> 상태라면, 협업 컨텍스트의 <mark style="color:blue;">`Discussion`</mark>의 ID를 참조하는 <mark style="color:blue;">`DiscussionDescriptor`</mark>와 함께 <mark style="color:blue;">`READY`</mark> 상태로 전환됨
* 결과적으로 일관성을 만족하게 됨
* 만약 <mark style="color:blue;">`initiateDiscussion()`</mark> 메서드가 호출된 시점에 이미 <mark style="color:blue;">`discussion`</mark> 필드가 <mark style="color:blue;">`READY`</mark> 상태였다면 어떻게 될까?
  * 즉, 이미 <mark style="color:blue;">`DiscussionStarted`</mark> 이벤트를 수신한 이후에 해당 <mark style="color:blue;">`Product`</mark>에 대해 같은 이벤트가 중복되어 전달되었다는 의미임
* <mark style="color:blue;">`initiateDiscussion()`</mark> 메서드는 멱등성을 만족하므로 문제를 일으키지 않는다.
  * 즉, 이벤트가 여러번 전달되는 상황을 멱등성을 통해 해결하였음

> 만약 장기 실행 프로세스가 메시징 메커니즘으로 인해 어떤 문제를 겪게 된다면 어떻게 될까?
>
> 프로세스가 끝까지 실행된다고 확신할 수 있을까?
>
> 다음 절에서 자세히 살펴보자

### 프로세스 상태 머신과 타임아웃 트래커

> 해당 절에서는 <mark style="color:blue;">**프로세스 트래커(Process Tracker)**</mark>라는 개념을 추가함으로써 장기 실행 프로세스를 더욱 성숙하게 만들어 본다

![프로세스 트래커 개념을 추가한 장기 실패 프로세스의 흐름도](../../../.gitbook/assets/DDD\_13\_V2.drawio.png)



* 사스오베이션 개발자는 <mark style="color:blue;">`TimeContrainedProcessTracker`</mark>라고 이름 지은 개념을 만듬
* 트래커는 완료까지 부여된 시간이 만료된 프로세스를 감시하는데, 이런 프로세스는 만료되기 전에는 몇 번이고 재시도 될 수 있음
* 트래커는 원하는 경우 고정된 시간 간격을 두고 재시도하도록 설계할 수 있고, 재시도를 전혀 하지 않거나 정해진 횟수만큼을 재시도한 후에 타임아웃을 발생시키도록 할수도 있음



* 프로세스의 현재 상태를 가지고 있는 것은 <mark style="color:blue;">`Product`</mark>이며, 컨텍스트는 아래의 조건이 만족되면 <mark style="color:blue;">`ProductDiscussionRequestTimedOut`</mark> 이벤트를 발행함
  * 재시도 타임에 도달함
  * 프로세스가 완전히 타임아웃됨

```java
package com.saasovation.agilepm.model.product;

public class ProductDiscussionRequestTimedOut extends ProcessTimedOut {

    public ProductDiscussionRequestTimedOut(
        String aTenantId,
        ProcessId aProcessId,
        int aTotalRetriesPermitted,
        int aRetryCount
    ) {
        super(aTenantId, aProcessId, aTotalRetriesPermitted, aRetryCount);
    }

}
```



* 재시도와 타임아웃에 관련된 알림을 수신할 수 있는 능력으로 무장한 <mark style="color:blue;">`Product`</mark>는 보다 개선된 프로세스에 참여할 수 있음
* 우선 기존과 동일하게 <mark style="color:blue;">`ProductDiscussionRequestedListerner`</mark>를 통해 프로세스를 시작하며, 리스너는 <mark style="color:blue;">`ProductService`</mark>의 <mark style="color:blue;">`startDiscussionInitiation()`</mark> 메서드를 호출함

```java
package com.saasovation.agilepm.application;

@Service
public class ProductService {

    // ...

    @Transactional
    public void startDiscussionInitiation(StartDiscussionInitiationCommand aCommand) {
        Product product = productRepository.productOfId(
            new TenantId(aCommand.getTenantId()),
            new ProductId(aCommand.getProductId())
        );

        if (product == null) {
            throw new IllegalStateException(
                "Unknown product of tenant id: "
                + aCommand.getTenantId()
                + "and product id: "
                + aCommand.getProductId());
        }

        String timedOutEventName = ProductDiscussionRequestTimedOut.class.getName();

        // 프로세스 트래커 생성
        TimeConstrainedProcessTracker tracker = new TimeConstrainedProcessTracker(
            product.tenantId().id(),
            ProcessId.newProcessId(),
            "Create discussion for product: " + product.name(),
            new Date(),
            5L * 60L * 1000L, // 5분 마다 재시도한다
            3, // 총 3회 재시도한다
            timedOutEventName
        );

        // 트래커를 영속화시킴
        processTrackerRepository.add(tracker);

        product.setDiscussionInitiationId(tracker.processId().id());
    }

    // ...

}
```



* 이 즈음에서 <mark style="color:blue;">`ProductCreated`</mark> 이벤트가 협업 컨텍스트에서 해석되도록 하지 않고 로컬에서 처리한 이유을 생각해보자
  * <mark style="color:blue;">`CreateExclusiveDiscussion`</mark> 커맨드를 협업 컨텍스트로 전달할 것이 아니라 <mark style="color:blue;">`ProductCreated`</mark> 이벤트를 바로 전달해도 되지 않았을 것 아닌가?
* 하지만 <mark style="color:blue;">`ProductCreated`</mark> 이벤트를 로컬에서 처리한 이유는 바로 <mark style="color:blue;">`Product`</mark>를 위한 트래커를 생성하는 이 접근법 때문임



* 프로세스의 경과 시간을 확인하기 위해 백그라운드 타이머가 정기적으로 활성화됨
* 이 타이머는 <mark style="color:blue;">`ProcessService`</mark>의 메소드인 <mark style="color:blue;">`checkForTimedOutProcess()`</mark>로 작업을 위임함

```java
package com.saasovation.agilepm.application;

@Service
public class ProductService {

    //...

    @Transactional
    public void checkForTimedOutProcesses() {
        Collection<TimeConstrainedProcessTracker> trackers = processTrackerRepository.allTimedOut();

        for (TimeConstrainedProcessTracker tracker : trackers) {
            // 프로세스의 재시도나 타임아웃이 필요한지 여부를 확인하고, 만약 확인된 경우에는 ProcessTimedOut의 서브클래스를 발행함
            tracker.informationProcessTimedOut();
        }
    }

    // ...

}
```



> 이어서 재시도와 타임아웃을 처리하는 <mark style="color:blue;">`ProductDiscussionRetryListener`</mark>를 살펴보자

```java
package com.saasovation.agilepm.infrastructure.messaging;

public class ProductDiscussionRetryListener extends ExchangeListener {
    // ...

    @Autowired
    private ProcessService processService;

    // ...

    @Override
    protected String exchangeName() {
        return Exchanges.AGILEPM_EXCHANGE_NAME;
    }

    @Override
    protected String [] listensToEvents() {
        return new String[] {
            "com.saasovation.agilepm.domain.model.product.ProductDiscussionRequestTimedOut",
        };
    }

    @Override
    protected void filteredDispatch(String aType, String aTextMessage) {
        Notification notification = NotificationSerializer.instance().deserialize(aTextMessage, Notification.class);

        ProductDiscussionRequestTimedOut event = notification.event();

        // 완전히 타임아웃되었는가?
        if (event.hasFullyTimedOut()) {
            // 타임아웃 시 처리해야할 로직 요청
            productService.timeOutProductDiscussionRequest(
                new TimeOutProductDiscussionRequestCommand(
                    event.tenantId(),
                    event.processId().id(),
                    event.occurredOn()
                )
            );
        } else {
            // 재시도에 필요한 로직 요
            productService.retryProductDiscussionRequest(
                new RetryProductDiscussionRequestCommand(
                    event.tenantId(),
                    event.processId().id()
                )
            );
        }
    }

    // ...
}
```



> 먼저 완전한 타임아웃이 발생했을 때의 처리 로직을 살펴보자

```java
package com.saasovation.agilepm.application;

@Service
public class ProductService {

    //...

    @Transactional
    public void timeOutProductDiscussionRequest(TimeOutProductDiscussionRequestCommand aCommand) {
        ProcessId processId = ProcessId.existingProcessId(aCommand.getProcessId());

        TenantId tenantId = new TenantId(aCommand.getTenantId());

        Product product = productRepository.productOfDiscussionInitiationId(tenantId, processId);

        // 실패 메일 전송
        this.sendEmailForTimedOutProcess(product);

        // Discussion 생성 실패에 따른 Product의 적절한 상태를 가질 수 있도록 메서드 호출
        product.failDiscussionInitiation();
    }

    // ...

}
```

```java
package com.saasovation.agilepm.model.product;

public class Product extends ConcurrencySafeEnity {

    // ...

    public void failDiscussionInitiation() {
        if (!this.discussion().availability().isReady()) {
            this.setDiscussionInitiationId(null);

            this.setDiscussion(ProductDiscussion.fromAvailability(DiscussionAvailability.FAILED));
        }
    }

    // ...

}
```

> 이제 재시도가 필요한 상황에서 실행되는 로직을 살펴보자

```java
package com.saasovation.agilepm.application;

@Service
public class ProductService {

    //...

    @Transactional
    public void retryProductDiscussionRequest(RetryProductDiscussionRequestCommand aCommand) {
        ProcessId processId = ProcessId.existingProcessId(aCommand.getProcessId());

        TenantId tenantId = new TenantId(aCommand.getTenantId());

        Product product = productRepository.productOfDiscussionInitiationId(tenantId, processId);

        if (product == null) {
            throw new IllegalStateException(
                "Unknown product of tenant id: "
                + aCommand.getTenantId()
                + "and product id: "
                + aCommand.getProductId());
        }

        // ProductDiscussionRequested 이벤트를 다시 발
        this.requestProductDiscussion(new RequestProductDiscussionCommand(aCommand.getTenantId(), product.productId().id()));
    }

    // ...

}
```

> 기존과 동일하게 협업 컨텍스에서 <mark style="color:blue;">`Discussion`</mark>이 생성되면, 결과적으로 <mark style="color:blue;">`ProductService`</mark>의 <mark style="color:blue;">`initiatationDiscussion()`</mark> 메서드가 실행되는데 여기에는 새로운 행동이 추가된다.

```java
package com.saasovation.agilepm.application;

@Service
public class ProductService {

    //...

    @Transactional
    public void initiateDiscussion(InitiateDiscussionCommand aCommand) {
        Product product = productRepository.productOfId(
                new TenantId(aCommand.getTenantId()),
                new ProductId(aCommand.getProductId())
        );

        if (product == null) {
            throw new IllegalStateException(
                "Unknown product of tenant id: "
                + aCommand.getTenantId()
                + "and product id: "
                + aCommand.getProductId());
        }

        product.initiateDiscussion(new DiscussionDescriptor(aCommand.getDiscussionId()));

        /* 트래커를 읽어와 완료 처리하는 로직이 추가 */
        timeContrainedProcessTracker tracker =
                this.processTrackerRepository.trackerOfProcessId(ProcessId.existingProcessId(product.discussionInitiationId()));

        // 이 시점 이후론 재시도나 타임아웃을 알리기 위해 트래커를 사용하지 않음. 프로세스는 끝남
        tracker.completed();
    }

    // ...

}
```



* 나름 만족스러운 결과이지만, 이 설계에는 약간의 문제가 남아있음
* 이런 방법을 사용해 <mark style="color:blue;">`Product`</mark>의 토론을 생성하는 요청을 재시도할 때, 현재 협업 컨텍스트는 중복된 이벤트에 대한 처리가 고려되어 있지 않음
* 우리는 이미 <mark style="color:blue;">`ProductService.initiateDiscussion()`</mark>를 멱등하게 설계함으로써, 비슷한 문제를 해결하였음
* 마찬가지로 <mark style="color:blue;">`ForumSerivce`</mark>의 <mark style="color:blue;">`startExclusiveForumWithDiscussion()`</mark> 메서드를 멱등하게 설계하여 중복된 이벤트 문제를 해결해볼 것임

```java
package com.saasovation.collaboration.application;

public class ForumService {

    // ...

    @Transactional
    public Discussion startExclusiveForumWithDiscussion(
        String aTenantId,
        String aCreatorId,
        String aModeratorId,
        String aForumSubject,
        String aForumDescription,
        String aDiscussionSubject,
        String anExclusiveOwner
    ) {
        Tenant tenant = new Tenant(aTenantId);

        Forum forum = forumRepository.exclusiveForumOfOwner(tenant, anExclusiveOwner);

        // 멱등성을 위한 코드
        if (forum == null) {
            forum = this.startForum(
                tenant,
                aCreatorId,
                aModeratorId,
                aForumSubject,
                aForumDescription,
                anExclusiveOwner
            );
        }

        Discussion discussion = discussionRepository.exclusiveDiscussionOfOwner(tenant, anExclusiveOwner);

        // 멱등성을 위한 코드
        if (discussion == null) {
            Author author = collaboratorService.authorFrom(tenant, aModeratorId);

            discussion = forum.startDiscussion(
                forumNavigationService,
                    author,
                    aDiscussionSubject
            );

            discussionRepository.add(discussion);
        }

        return discussion;
    }

    // ...

}
```

### 메시징이나 시스템을 활용할 수 없을 때

* 복잡한 소프트웨어 시스템을 개발하는 데 있어서 어떤 접근법이라 해도 만병통치약이 될 수는 없음
* 모든 접근법에는 나름의 쟁점과 단점이 있음
* 메시징 시스템의 한 가지 문제점은 일정 기간 동안 사용이 불가능할 수 있다는 점임



* 메시징 메커니즘이 일정 기간 동안 오프라인이라면, 알림 발행자는 메시지를 보낼 수 없음
* 그렇다면 메시징 시스템이 다시 사용 가능할 때까지 알림 발송 시도를 잠시 미루는 편이 최선일 수 있음
* 하나의 발송 건만 성공해도 사용가능한 상태가 됐음을 알아차릴 수 있는데, 그 전까지는 발송 횟수를 줄이는 것이 바람직함
* 만약 개발하는 시스템이 이벤트 저장소를 포함하고 있다면, 저장소에 저장된 이벤트를 메시징이 다시 가능해졌을 때 이를 발송할 수 있다는 점을 기억하자



**Materials**

{% file src="../../../.gitbook/assets/DDD_13_V2-2 (3).drawio" %}
