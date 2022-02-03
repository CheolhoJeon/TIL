# CQRS Documents by Greg Young

[![npm](https://img.shields.io/badge/version-2019.42-brightgreen.svg)](broken-reference)

## 전통적인 아키텍처

도메인 주도 설계 기반 프로젝트의 아키텍처를 살펴보기 전에, 일반적으로 많은 사람들이 프로젝트에 적용하려고하는 표준 아키텍처가 무엇인지 분석할 필요가 있습니다. 이를 통해 생산성 측면의 비용을 최소화하면서 전통적인 아키텍처를 더 나은 아키텍처로 개선할 수 있습니다.

아래의 그림은 전통적인 아키텍처를 보여줍니다.

***

![](https://user-images.githubusercontent.com/42791260/54966396-81fd9580-4fb7-11e9-8e4c-e1fb97c57dee.png)

***

**\[그림 1] 전통적인 아키텍처**

### 애플리케이션 서버

위의 아키텍처는 백업 저장소 시스템을 중심으로 구성되었습니다. 데이터 저장소는 일반적으로 RDBMS이지만 [키-값 데이터베이스](https://en.wikipedia.org/wiki/Key-value\_database), [객체 지향 데이터베이스](https://en.wikipedia.org/wiki/Object\_database), XML 파일이 될 수도 있습니다. 여기서 중요한 점은 백업 저장소가 도메인 객체의 현재 상태를 나타낸다는 것입니다.

백업 저장소는 애플리케이션 서버 위에 위치합니다. \[그림 1]에서 도메인으로 표시된 논리 영역은 시스템의 비즈니스 로직을 담당합니다. 애플리케이션 서버에게 전달된 요청을 처리하기 위한 유효성 검사 및 오케스트레이션 로직이 이 영역에서 처리됩니다.

\[그림 1]에서는 데이터 계층(data tier)이 표현되지 않았지만 애플리케이션 서버와 데이터 저장소 사이에 데이터 계층이 존재할 수 있다는 점에 유의해야합니다. 또한 이 아키텍처를 구현하는데 도메인 영역이 필수 사항은 아니며 테이블 모듈(Table Module)이나 트랜잭션 스크립트(Transaction Script)와 같은 패턴을 사용할 수도 있다는 점도 중요합니다. With these only existing as Application Services.

도메인 영역을 제거하면 애플리케이션 서비스 영역을 만날수 있습니다. 애플리케이션 서비스는 도메인 및 데이터와의 인터페이스 역할을 하며, 도메인 소비자와 도메인 간의 결합을 줄여줍니다.

애플리케이션 서버 바깥에서는 여러 종류의 원격 파사드(Remote Facade)를 확인 할 수 있습니다. 원격 파사드는 SOAP, 커스텀된 TCP/IP, HTTP를 통한 XML, 톰캣 심지어 비둘기 다리에 매달은 쪽지가 될 수도 있습니다. 원격 파사드는 관련 상황과 도구에 따라 기본 기술 메커니즘에서 제외 될 수도 있습니다.(The Remote Facade may or may not be abstracted away from its underlying technology mechanism depending on the situation and tools that are involved.)

시스템의 데이터 저장소를 추상화하고 비즈니스 로직을 집중시키기 위해 전통적인 애플리케이션 아키텍처는 여러해 동안 매우 인기가 높았으며 많은 시스템에 적용되었습니다.

### 클라이언트와의 상호 작용

클라이언트의 일반적인 상호 작용은 \[그림 2]에서 살펴볼 수 있습니다.

***

![](https://user-images.githubusercontent.com/42791260/54973038-98b0e600-4fd1-11e9-8d89-bff68af88659.png)

***

**\[그림 2] Typical Client Interaction**

클라이언트의 기본적인 상호작용은 DTO(Data Transfer Object)의 상/하 상호작용으로 설명 할 수 있습니다. 작업의 수명주기를 살펴 보면 API의 기능을 쉽게 이해할 수 있습니다. 화면을 통해 사용자가 \_Customer\_의 정보를 수정한다고 합시다. 먼저 클라이언트는 \_Customer\_의 id가 포함된 DTO를 포함한 요청을 원격 파사드로 보냅니다. 원격 파사드는 응답에 필요한 도메인 객체를 로드하고 도메인 객체를 DTO에 매핑 한 다음 클라이언트에 반환합니다. \[그림 3]은 XML 형식의 DTO 예제를 보여줍니다. 여기서 전통적인 아키텍처에서 DTO는 클라이언트의 요청에 필요한 객체의 현재 상태를 저장하고 있음을 인지해야합니다.

이 후, 클라이언트는 원격 파사드에서 수신한 정보를 화면에 표시하여 사용자가 상호 작용할 수 있게합니다. This is very often done utilizing a view model and/or data binding with the view.

특정 시점에서 사용자는 화면(UI)을 통해 데이터 편집을 완료하고 데이터를 저장하게 됩니다. 일반적으로 저장 버튼을 통해 구현되만 일부 사용자 인터페이스는 현재 데이터를 강제적으로 저장하는 경우도 있습니다.

***

![](https://user-images.githubusercontent.com/42791260/54978115-aae75000-4fe2-11e9-826d-b1993ed1c9fd.png)

***

**\[그림 3] Example in XML of a DTO**

클라이언트는 데이터를 저장하기 위해 사용자가 화면에서 편집한 데이터를 DTO로 옮깁니다. 일반적으로 사용자에게 표시하기 위해 원격 파사드에서 요청한 DTO와 동일합니다. 이 후 클라이언트는 애플리케이션 서버로 완성한 DTO를 전송합니다.

DTO를 수신 한 응용 프로그램 서버는 트랜잭션 / 세션을 시작하고 도메인 객체에 DTO를 다시 매핑하고 도메인 객체가 변경 사항을 확인하도록 허용 한 다음 도메인 객체 내의 변경 사항을 사용하여 가능성이있는 데이터 저장소로 다시 저장합니다. 도메인 객체를 변경 한 것을 구별하고 그에 따라 데이터 저장소를 업데이트 할 수있는 객체 관계 맵퍼를 활용할 수 있습니다. 작업을 마친 애플리케이션 서버는 변경이 완료 되었다는 정보 또는 변경 실패의 이유에 대한 에러 메세지를 Ack를 통해 클라이언트에게 전송합니다.

### 전통적인 아키텍처에 대한 분석

모든 아키텍처와 마찬가지로 위에서 기술된 아키텍처에는 많은 특성이 있습니다. 몇몇 특성은 특정 시나리오에 대해 적합할 수 있으나 다른 시나리오에 대해서 매우 나쁠 수 있습니다. 설계자로서 우리는 요구사항에 대해 가장 적합한 특성들을 활용하기 위해 노력해야합니다.

#### 단순함

아키텍처의 특성을 살펴볼때 주어진 아키텍처에서 가장 인기있는 특징이 무엇인지 살펴보는 것이 좋습니다. 위의 아키텍처에서 가장 인기 있다고 볼 수 있는 특징은 \*\*심플(Simple)\*\*하다는 것입니다. 주니어 개발자는 이 아키텍처를 이용하여 구축된 시스템과 상호작용하는 방법을 빠른시간내에 익힐수 있습니다. 또한 이 아키텍처는 매우 일반적인 아키텍처이며, 모든 프로젝트에서 이 아키텍처를 사용할 수도 있습니다. 기존에 많은 사람들이 전통적인 아키텍처를 사용하고 있기 때문에, 팀에 새로운 멤버가 추가 되었을 때 신입 멤버는 전통적인 아키텍처가 적용된 시스템에 친숙하므로 적응에 필요한 비용을 줄일 수 있습니다.

간단하며 많은 사람들에게 익숙하다는 장점으로 인해 개발 팀은 해당 아키텍처를 능숙하게 적용할 수 있으며, 새로운 프로젝트를 진행함에 있어 기본 아키텍처로 사용할 수 있습니다. The thought process of needing to align non-functional requirements really goes away as they know that this architecture will be good enough for 80% of the projects that they run into.

#### Tooling

전통적인 아키텍처를 활용하여 시스템을 구축하는데 필요한 시간을 최소화 하는 프레임워크는 기존에 많이 존재합니다. ORM은 복잡한 객체 그래프로 변경 추적 및 트랜잭션 관리와 같은 중요한 서비스를 제공하는 가장 큰 단일 사례입니다. 다른 예로는 도메인 객체에서 양측의 DTO로 매핑되는 자동 매핑 프레임 워크가 포함되며 이는 Application Server에서 DTO를 앞뒤로 매핑하는 데 필요한 "plumping code"양을 대부분 제거합니다.

물론 위에서 언급된 아키텍처와 관련하여 단점들도 있습니다. 그 중 가장 중요한 단점은 전통적인 아키텍처로 DDD를 적용하는 것이 불가능하다는 것입니다.

#### 도메인 주도 설계

DDD를 적용하는 많은 사람들이 전통적인 아키텍처를 사용하지만 위의 아키텍처에서는 DDD를 적용 할 수 없습니다. 유비쿼터스 언어(Ubiquitous Language)가 어떻게 객체 모델에 의해 표현되는지 살펴보면 왜 불가능한지 쉽게 알 수 있습니다. 위의 아키텍처에는 단 4개의 동사가 있습니다. 네 가지 동사는 Create, Read, Update, Delete(CRUD)입니다. Remote Facade에는 데이터 지향 인터페이스가 있기 때문에 응용 프로그램 서비스는 반드시 동일한 인터페이스를 가져야합니다.

"Anemic Model"라는 기존에 잘알려진 [안티패턴(anti-pattern)](https://ko.wikipedia.org/wiki/%EC%95%88%ED%8B%B0%ED%8C%A8%ED%84%B4)이 있습니다.

"Anemic Model은 언뜻 보기에는 괜찮은 패턴처럼 보입니다. 도메인 공간에 명사의 이름을 딴 객체가 많이 있으며,이 객체들은 실제 도메인 모델이 가지고있는 풍부한 관계와 구조로 연결되어 있습니다. The catch comes when you look at the behavior, and you realize that there is hardly any behavior on these objects, making them little more than bags of getters and setters. 실제로 이러한 모델에는 도메인 논리를 도메인 객체에 포함시키지 말라는 설계 규칙이 있습니다. 대신 모든 도메인 로직을 담당하는 일련의 서비스 객체가 있습니다. 이러한 서비스는 도메인 모델의 최상위에 있으며 데이터를 얻기 위해 도메인 모델을 사용합니다." (Fowler, 2003)

위의 아키텍처에서 구현중인 모델은 처음에는 Anemic Model 처럼 보입니다. 응용 프로그램 서비스는 데이터를 DTO에 앞뒤로 매핑하기 위해 도메인 객체는 동작이 거의 없으며 매핑하는 과정에서 사용하기 위해 getter 및 setter를 사용합니다. There is a structure to the domain showing how objects relate with one another but...

이 아키텍처를 사용하여 도메인 구조 모델을 생성하고 채울 수는 없으며, 서비스 로직 자체가 서비스에 포함될 것입니다. 여기서는 서비스 자체는 실제로 DTO를 도메인 객체에 매핑하기 때문에 실제 비즈니스 로직은 없습니다. 이 경우 많은 양의 비즈니스 로직은 도메인이나 애플리케이션 서버에 존재 하지 않습니다. 클라이언트에 존재할 수 있겠지만 메뉴얼 문서에 기재되어 있거나 시스템 사용자가 알고 있을 가능성이 큽니다.

Architectures like the one being viewed tend to come with instructions of how to complete complex tasks by editing data in many parts of the system. 하나의 예로 직원의 성별이 수정됨으로써 건강 보험 정보도 함께 수정되어야하는 상황이 있습니다. This is far worse than the creation of an anemic model, this is the creation of a glorified excel spreadsheet.

#### 스케일링

위의 아키텍처를 살펴보면 스케일링 관점에서 큰 병목현상이 발생할 수 있음을 쉽게 알 수 있습니다. The bottleneck in terms of scaling is the data storage. When using a RDBMS as 90%+ currently use this becomes even more of a problem most RDBMS are at this point not horizontally scalable and vertically scaling becomes prohibitively expensive very quickly. It is however also extremely important to remember that most systems do not need to scale and as such scalability is really not a grave issue in all cases.

### 요약

많은 프로젝트에서 채택 된 DTO up / down 아키텍처는 많은 애플리케이션에 사용될 수 있으며 팀이 개발 하는데 있어 단순성 측면에서 많은 이점을 제공 받을 수 있습니다. 그러나 DDD 기반 프로젝트에 활용할 수 없습니다. 이 아키텍처는 좋은 기준을 보여주지만, 이 문서의 나머지 부분에서는 각 추가 단계에서 비즈니스 가치를 추가하면서 비용을 제한하거나 제거하려고 시도하면서 점진적 단계로이 아키텍처를 개선하는 데 중점을 둘 것입니다.

## Task Based User Interface

이 장에서는 작업 기반 사용자 인터페이스의 개념을 소개하고 이를 CRUD 스타일의 사용자 인터페이스와 비교합니다. 또한 더 많은 태스크 지향 스타일이 API에 적용될 때 애플리케이션 서버 내에서 발생하는 변경 사항을 보여줍니다.

전통적인 아키텍처에서 가장 큰 문제점 중 하나는 사용자가 취하고자 하는 것이 사라졌다는 것입니다. 클라이언트는 데이터 중심의 DTO를 애플리케이션 서버와 주고 받으며 상호 작용했기 때문에 도메인에 동사를 포함 할 수 없었습니다. 도메인은 그저 데이터 모델을 추상화한 결과물이 되었습니다. 클라이언트, 종이 조각 또는 소프트웨어 사용자의 머리 속에 존재했던 행동, 즉 존재해야 했던 행동이 없었습니다.

이와 관련된 애플리케이션들이 많이 있습니다. 이러한 응용 프로그램의 많은 예가 언급 될 수 있습니다. 사용자에게는 문서화 된 "작업 흐름"정보가 있습니다. 화면 xyz로 이동하여 foo를 막대로 편집 한 다음이 다른 화면으로 이동하여 xyz를 abc로 편집하십시오. 많은 유형의 시스템에서이 유형의 워크 플로우는 문제가 없습니다. 이러한 시스템은 일반적으로 비즈니스 측면에서 낮은 가치를 지닙니다. DDD를 사용하기에 충분히 복잡하고 ROI가 충분한 영역에서는 이러한 유형의 워크 플로가 다루기 힘들어집니다.

One reason that is commonly cited for wanting to build a system such as described is that "the business logic and work flows can be changed at any time to anything without need of a change to the software." While this may be true it must be asked at what cost. 누군가가 프로세스의 한 단계를 빠드리거나 공통적인 상황에서 각 사용자들이 다르게 행동하면 어떤 일이 벌어질까요? 당신은 보고를 위해 시스템에서 정보를 얻어 낼 때에 어떻게 의미있는 정보를 얻어 올 건가요?

이 문제를 해결할 수 있는 한가지 방법은 전통적인 아키텍처에서 설명된 DTO up/down 아키텍처에서 벗어나는 것입니다. \[그림 1]은 DTO up/down 아키텍처에서 클라이언트의 상호작용을 보여줍니다.

***

![](https://user-images.githubusercontent.com/42791260/55783779-2d473800-5aea-11e9-86ba-b27a7036ce7f.png)

***

**\[그림 4] Interaction in a DTO up/down Architecture**

UI가 DTO를 요청하는 가장 기본적인 예로서 애플리케이션 서버에게 Customer 1234를 요청하는 예가 있습니다.

전체적인 동작의 기본 설명으로서, UI가 애플리케이션에게 Customer 1234에 대한 DTO를 요청하면 해당 DTO는 클라이언트에게 반환되고 화면에 보여질 것입니다. 이 후 사용자는 어떤식으로는 DTO와 상호작용할 것입니다. 결국 클라이언트는 저장 버튼 또는 DTO를 취할 수 있는 트리거를 통해 DTO를 애플리케이션 서버로 반환할 것입니다. 그런 다음 애플리케이션 서버는 데이터를 내부적으로 도메인 모델에 매핑하고 변경 내용을 저장한 뒤 성공 또는 실패를 반환합니다.

앞에서 언급했듯이 클라이언트의 작업이 완료된 후 객체의 현재 상태를 나타내는 DTO만 전송되기 때문에 사용자의 취지가 손실됩니다. 사용자의 취지를 이끌어냄으로써 애플리케이션 서버가 데이터를 단순 저장만 하는 것이 아닌 동작을 처리 할 수 있도록 할 수 있습니다. 아래의 그림은 사용자의 의도를 포함하여 상호 작용하는 상황을 보여줍니다.

***

![](https://user-images.githubusercontent.com/42791260/55785555-99776b00-5aed-11e9-92a4-e80491cab482.png)

***

**\[그림 5] Behavioral Interface**

클라이언트 상호 작용의 취지를 포착하는 것은 상호 작용 측면에서 DTO up/down 방법과 매우 유사합니다. 먼저 클라이언트는 Customer 1234 객체에 대한 DTO를 애플리케이션 서버에게 요청합니다. 애플리케이션 서버는 Customer의 상태를 나타낼 수 있는 DTO를 클라이언트에게 반환하고 클라이언트는 DTO를 화면에 표시 한 다음 보통 사용자가 직접 또는 뷰 모델을 통해 상호 작용할 수 있도록합니다. 클라이언트의 의도를 포함한 상호 작용과 DTO up/down 아키텍처에서의 상호작용의 공통점은 여기 까지입니다.

사용자가 작업을 완료 할 때 단순히 동일한 DTO를 다시 보내는 대신 클라이언트는 애플리케이션 서버에 어떤 작업을 한다는 메시지를 보냅니다. 이 메세지는 "판매 완료", "구매 주문 승인", "대출 신청서 제출" 같은 것들이 될 수 있습니다. 간단히 말하면 클라이언트는 사용자가 완료하고자하는 작업을 완료하기 위해 애플리케이션 서버에 메시지를 보내야합니다. 애플리케이션 서버에게 사용자가 수행하고자하는 것을 알려줌으로써 사용자의 의도를 알 수 있습니다.

### Commands

애플리케이션 서버가 수행 할 작업을 지시하는 방법은 Command를 사용하는 것입니다. Command는 동작을 수행하기 위한 데이터와 동작을 나타내는 이름을 가진 단순한 객체입니다. 많은 사람들은 Command를 직렬화 가능한 메서드의 호출이라고 생각합니다. \[목록 1]은 기본적인 Command의 코드를 보여줍니다.

```
public class DeactivateInventoryItemCommand {
	public readonly Guid InventoryItemId;
    public readonly string Comment;
    
    public DeactivateInventoryItemCommand(Guid id, string comment) {
        InventoryId = id;
        Comment = comment;
    }
}
```

**\[목록 1] A Simple Command**

_참고로 Listing 1의 예제에는 명령 이름 뒤에 패턴 이름이 포함되어있습니다. 이것은 언어 적으로나 운영상으로 많은 긍정과 부정적인 결정을 내린 결정입니다. 클래스 이름에 패턴 이름을 사용할지 여부는 개발 팀에서 가볍게 받아 들여서는 안되는 것입니다._

Commands의 한 가지 중요한 측면은 항상 명령 시제를 사용한다는 것입니다. 즉, 애플리케이션 서버에 무언가를하도록 지시하고 있습니다. 만약 판매가 이미 이루어지고 "SaleOccurred"라는 Command 객체를 보내는 상황을 상상해봅시다. 이 Command 객체를 분석할 때 "이 상황은 사실 벌어지지 않았습니다"라고 할 수 있나요? Command를 명령 시제로 작성하면 애플리케이션 서버에서 명령을 거부 할 수 있음을 언어학적으로 나타낼 수 있습니다. 만약 이를 허용하고 싶지 않다면 Command가 아닌 Event가 되어야 될 것이며, 더 자세한 내용은 **Events** 절에서 찾아볼수 있습니다.

Occasionally there exist funny examples of language in English. A perfect example of this would be "Purchase" which can be used either as a verb in the imperative or as a noun describing the result of its usage in the imperative. When dealing with these situations, ensure that the concept being pushed forward represents the imperative of the verb and not the noun. As an example a purchase should be including what to purchase and expecting the domain to possibly fill in some information like when the item was purchased as opposed to sending up a purchase DTO that fully describes the purchase.

\[목록 1]의 Command는 두 개의 데이터를 가집니다. 여기에는 비활성화 할 InventoryItem을 나타내는 id가 포함되며 항목이 비활성화되는 이유에 대한 설명이 포함됩니다. comment은 Command와 연관된 속성에서 아주 일반적이며, 동작을 처리하는 데 필요한 데이터 중 하나입니다. 주어진 행동을 처리하는 데 필요한 데이터만 존재해야합니다. 이는 객체의 전체 데이터가 애플리케이션 서버로 다시 전달되는 일반적인 아키텍처와 크게 대조됩니다.

데이터 중 가장 중요한 것은 연관된 인벤토리 항목의 ID입니다. 모든 명령이 의도된 객체로 변환 될 수 있도록, 상태를 업데이트하는 모든 Command는 하나 이상의 ID가 있어야합니다. 그러나 Create를 위한 Command를 생성할 때는 id를 포함하지 않아도 됩니다. 클라이언트가 UUID의 형태로 ID를 생성하게하는 것은 분산 시스템에서 매우 중요합니다. 보통 개발자는 Command에 대해 배우고 "ChangeAddress", "CreateUser"또는 "DeleteClass"와 같 친숙한 어휘를 사용하여 명령을 매우 빠르게 작성합니다. 하지만 이는 기본적으로 지양해야합니다. 대신 팀은 유스 케이스가 실제로 무엇인지에 초점을 맞추어야합니다.

"ChangeAddress"가 맞을까요? "Relocating the Customer"와 "Correcting an Address"의 차이점은 무엇일까요? It likely will be if the domain in question is for a telephone company that sends the yellow pages to a customer when they move to a new location.

"CreateUser"일까요, 아니면 "RegisterUser"일까요? "DeleteClass"일까요, "DeregisterStudent"일까요? 이렇게 이름을 정하는 과정을 통해 도메인에 대한 많은 통찰을 얻을 수 있습니다. To begin defining Commands, the best place to begin is in defining use cases, as generally a Command and a use case align.

종종 데이터의 한 부분을 오직 “create”, “edit”, “update”, “change”, 또는 “delete” 하는 유스케이스만 있을때가 있습니다. 또한 주목해야합니다. All applications carry information that is simply supporting information. It is important though to not fall into the trap of mistaking places where there are use cases associated with intent for these CRUD only places.

개념으로서의 명령은 어렵지 않지만 개발자들에게는 다릅니다. 많은 개발자들은 Command를 만들어내는것은 오래 걸린다고 생각합니다. If the creation of Commands is a bottleneck in the workflow, many of the ideas being discussed are likely being applied in an incorrect location.

### User Interface

Command를 만들기 위해 사용자 인터페이스는 일반적으로 DTO up/down 시스템과는 약간 다르게 작동합니다. UI는 Command 객체를 작성해야하기 때문에 사용자의 취지에 따라 사용자 취지를 파생시킬 수 있어야합니다.

이 문제를 해결하는 방법은 Microsoft 세계에서 "유도 사용자 인터페이스(Inductive Interface)"라고도 알려진 "작업 기반 사용자 인터페이스(Task Based User Interface)"를 사용하는 것입니다. 이 UI 스타일은 전혀 새로운 것이 아니며 사용자 인터페이스 디자인에 대한 완전히 다른 관점을 제공합니다. Microsoft는 Inductive UI를 연구 할 때 Deductive UI의 세 가지 주요 문제점을 확인했습니다.

**사용자는 제품의 적절한 Mental Model을 구성하지 않는 것처럼 보입니다.** 대부분의 최신 소프트웨어 제품의 인터페이스 디자인은 디자이너가 신중하게 제작 한 개념 모델을 사용자가 이해할 것으로 가정합니다. 불행히도 대부분의 사용자는 자신의 탐색을 안내 할만큼 철저하고 정확한 정신 모델을 얻는 것만 큼 보이지 않습니다. 사용자가 바보여서가 아니라 너무 바쁘고 정보가 너무 많기 때문입니다. 그들은 자신의 소프트웨어에 대한 개념적 모델에 대해 궁금해 할 시간, 에너지 또는 열망이 없습니다.

**대부분의 오랜된 사용자조차도 일반적인 절차를 숙지하지 않습니다.** 디자이너는 처음에는 새로운 사용자가 문제를 겪을 수도 있지만 사용자가 일반적인 작업을 배울 때 이러한 문제가 사라질 것으로 예상합니다. Usability data indicates this often doesn't happen. 한 연구에서 연구자들은 집에서 비디오 테이프를 찍기 위해 장비를 설치했습니다. 이 테이프는 현재 진행중인 작업에 중점을 둔 사용자가 반드시 따르고있는 절차에 주목하지 않고 경험을 통해 배우지 않는다는 것을 보여주었습니다. 다음 번에 사용자가 동일한 작업을 수행하면 똑같은 방식으로 문제가 발생할 수 있습니다.

**사용자는 각 기능이나 화면을 파악하기 위해 열심히 노력해야합니다.** 대부분의 소프트웨어 제품은 개념 모델을 이해하고 일반적인 절차를 숙달 한 (소수의) 사용자를 위해 설계되었습니다. 대다수의 고객에게 각 기능 또는 절차는 좌절감을 주는 원치 않는 퍼즐입니다. 사용자는 이러한 수수께끼를 컴퓨터 사용에 불가피한 비용이라고 생각할 수 있지만 이러한 부담 없이 사용하길 원할것입니다.

작업 기반 또는 유도 UI의 기본 아이디어는 사용자가 소프트웨어를 사용하는 방법을 파악하고 해당 프로세스를 통해 사용자를 안내하는 것이 중요하다는 것입니다. (Microsoft Corporation, 2001)

_Many commercial software applications include user interfaces in which a screen presents a set of controls, but leaves it to the user to deduce the page's purpose and how to use the controls to accomplish that purpose. (Microsoft Corporation, 2001)_

목표는 프로세스를 통해 사용자를 안내하는 것입니다. 앞서 살펴본 DeactivateInventoryItem 예제에서 차이점을 찾아볼 수 있습니다. 일반적인 Deductive UI에는 모든 인벤토리 항목을 포함하는 편집 가능한 데이터 격자가있을 수 있습니다. 다양한 데이터에 대해 편집 가능한 필드가 있고 인벤토리 항목 상태에 대한 드롭 다운(셀렉트 태그)이있을 수 있습니다. 인벤토리 아이템을 비활성화하려면 사용자가 그리드의 항목을 이동해야하고, 비활성화 이유에 대한 설명을 입력 한 다음 드롭 다운을 비활성 상태로 변경해야합니다. 인벤토리 항목을 편집하기 위해 화면을 클릭했지만 그림 3에서와 동일한 프로세스를 거치는 유사한 예가있을 수 있습니다.

***

![](https://user-images.githubusercontent.com/42791260/55953683-ba7bbf80-5c97-11e9-8237-4a375eac470b.png)

***

**\[그림 6] A CRUD screen for an Inventory Item**

사용자가 "비활성화"된 항목을 제출하려고 시도하고 Comment를 입력하지 않은 경우 비활성화 된 항목의 필수 입력란이므로 Comment를 입력해야한다는 오류 메시지가 표시됩니다. 좀 더 사용자 친화적인 UI는 사용자가 드롭 다운에서 비활성화 된 것을 선택하여 화면에 표시 할 때까지 주석 필드를 표시하지 않을 수 있습니다. This is far more intuitive to the user as it is a cue that they should be putting data in that field but one can do even better.

***

![](https://user-images.githubusercontent.com/42791260/55954464-b8b2fb80-5c99-11e9-9edd-0427e4c210aa.png)

***

**\[그림 7] Listing Screen with Link**

작업 기반 UI는 인벤토리 항목 옆에 인벤토리 항목 목록을 표시하는 다른 접근 방식을 취하고, 인벤토리 항목 옆에 그림 7과 같이 항목을 "비활성화"할 수있는 링크가있을 수 있습니다. 이 링크를 클릭하면 그림 8과 같은 항목을 비활성화하는 이유에 대한 설명을 묻는 화면이 나타납니다. 이 경우 사용자의 의도는 명확하며 소프트웨어는 인벤토리 항목을 비활성화하는 프로세스를 안내합니다. 이 인터페이스 스타일로 사용자의 의도를 나타내는 명령을 작성하는 것도 매우 쉽습니다.

***

![](https://user-images.githubusercontent.com/42791260/55957500-7a213f00-5ca1-11e9-90a4-b62134b6dbd2.png)

***

**\[그림 8] Deactivating an Inventory Item**

웹, 모바일 및 특히 Mac UI는 작업 기반의 방향으로 기울어졌습니다. UI는 프로세스를 안내하고 상황에 맞는 지침을 제공하여 올바른 방향으로 이끌어줍니다. 이것은 훨씬 더 나은 사용자 경험을 제공하는 스타일 덕분입니다. **사용자가 소프트웨어를 사용하는 방법과 이유에 대한 확실한 초점이 있습니다. 사용자의 경험은 프로세스의 필수적인 부분이됩니다.** 이 외에도 사용자가 소프트웨어를 사용하는 방법에 더 많은 관심을 기울일 필요가 있습니다. 이것은 도메인의 일부 동사를 정의하는 데있어 훌륭한 첫 걸음입니다.

## Command and Query Responsibility Segregation

이 절은 Command and Query Responsibility Segregation의 개념을 소개합니다. 시스템에서의 역할 분리가 어떻게 훨씬 효과적인 아키텍처로 이어질 수있는 방법을 살펴볼 것입니다. 또한 CQRS가 적용된 시스템에 존재하는 아키텍처의 다양한 특징을 분석합니다.

### Origins

\*\*Command and Query Responsibility Segregation(CQRS)\*\*는 Bertrand Meyer의 Command and Query Separation(CQS) 원칙에서 시작되었습니다. 위키피디아는 다음과 같이 정의합니다.

> 모든 메서드는 행동을 수행하는 명령이거나 데이터를 리턴하는 조회 중 하나여야 하며 둘다 아닌 경우는 없다. 즉, 질문(question)이 답변(answer)을 변경해서는 안된다. 더 형식적으로, 메소드는 참조가 투명하여 부작용이없는 경우에만 값을 리턴해야합니다. (Wikipedia)

요약하자면, 반환 값이 있는 경우 상태를 변경할 수 없음을 의미합니다. 상태를 변경하면 반환형이 void가 되어야합니다. 하지만 여기에 몇 가지 문제가있을 수 있습니다. 마틴 파울러는 bliki에서 다음과 같은 예제를 보여줍니다.

> Meyer는 모든 상황에서 CQS를 사용하기를 원하지만 예외가 있습니다. 그 예로서 스택의 pop 연산이 있습니다. Meyer는이 방법을 사용하는 것을 피할 수 있다고 단호히 말합니다. but it is a useful idiom. 그래서 저는 최대한 원칙을 따르는 것을 선호하지만, 때에 따라 원칙을 깨뜨릴 준비가 되어있습니다. (Fowler)

원래 CQRS는 CQS 개념을 확장 한 것으로 간주되고 오랫동안 CQRS는 단순히 더 높은 수준의 CQS로 논의되었습니다. 결국 두 개념 사이에 많은 혼란이 있었지만 정확하게 다른 패턴으로 간주되었습니다.

**CQRS와 CQS에서의 명령과 조회에 대한 정의는 동일합니다.** 하지만 두 패턴의 근본적인 차이점은 CQRS는 하나의 객체를 각각 명령과 조회를 담당하는 두 개의 객체로 분리한다는 것입니다.

CQRS 패턴은 그다지 흥미롭지는 않을수 있지만 아키텍처 관점에서 보았을 때 매우 흥미롭습니다.

\[그림 1]은 첫 번째 장에서 다룬 전통적인 아키텍처를 보여줍니다. 전통적인 아키텍처에서 중요한 점은 서비스가 명령과 쿼리를 모두 처리한다는 것입니다. 종종 도메인도 명령과 쿼리 모두에 사용됩니다. 이 아키텍처에 대한 CQRS의 적용은 매우 단순하지만 아키텍처에서 얻을 수 있는 기회를 획기적으로 바꿀 수 있습니다. \[목록 2]의 서비스들을 변환해보겠습니다.

***

![](https://user-images.githubusercontent.com/42791260/57061357-82482980-6cf7-11e9-853a-0cdc45b69dcd.png)

***

**\[목록 2] Original Customer Service**

CQRS를 CustomerService에 적용한 결과는 \[목록 3]과 같습니다.

***

![](https://user-images.githubusercontent.com/42791260/57061365-8a07ce00-6cf7-11e9-8bdb-705320f7df53.png)

***

**\[목록 3] Customer Service after CQRS**

상대적으로 간단한 과정이지만 전통적인 아키텍처의 많은 문제를 해결할 것입니다. 서비스들은 리드 사이드와 라이트 사이드, 또는 커맨드 사이드와 쿼리 사이드의 두 가지 개별 서비스로 나뉘어져 있습니다.

이러한 분리는 커맨드 사이드와 쿼리 사이드는 각각 다른 요구사항을 가진다는 개념을 강제합니다. 유스케이스와 관련된 아키텍처 속성은 각 사이드 마다 상당히 다른 경향이 있습니다. 몇가지 말하자면:

#### Consistency - 다시

**Command:** It is far easier to process transactions with consistent data than to handle all of the edge cases that eventual consistency can bring into play.

**Query:** 대부분의 시스템은 결국 쿼리 사이드에서 일관성을 유지할 수 있습니다.

#### Data Storage

**Command:** 관계형 모델에서 커맨드 사이드는 정규화된 데이터를 저장하려고합니다. 아마도 3NF 근처에있을 것입니다.

**Query:** 쿼리 사이드에서는 비정규화 된 방식으로 데이터를 저장하여 주어진 데이터 집합을 얻는 데 필요한 조인 수를 최소화할 수 있습니다. 관계형 구조에서 1NF일 것입니다.

#### Scalability

**Command:** 대부분의 시스템, 특히 웹 시스템의 경우 커맨드 사이드는 일반적으로 매우 적은 수의 트랜잭션을 처리합니다. 따라서 확장성은 항상 중요하지 않습니다.

**Query:** 대부분의 시스템, 특히 웹 시스템에서 쿼리 사이드는 일반적으로 매우 많은 수의 트랜잭션을 처리합니다(종종 2 배 이상 증가합니다). Scalability는 쿼리 측면에서 가장 자주 필요합니다.

**단일 모델을 사용하여 트랜잭션을 검색, 리포팅 및 처리하는 최적의 솔루션을 생성하는 것은 불가능합니다.**

### The Query Side

처음 언급한것 처럼 쿼리 사이드에는 데이터를 가져오는 메서드만 포함된다. 전통적인 아키텍처에서 클라이언트의 화면에 데이터를 표시하기 위해 DTO를 반환하는 모든 메서드가 조회 사이드에 포함될 수 있다.

전통적인 아키텍처에서 DTO를 설계할 때 도메인 객체를 투영하여 처리하였습니다. 이 과정은 많은 고통을 가져올 수 있습니다. 고통의 큰 원인은 DTO가 도메인과 다른 모델이므로 맵핑이 필요하다는 것입니다.

DTO는 서버와의 다중 왕복을 방지하기 위해 클라이언트 화면과 일치하도록 구축됩니다. 많은 클라이언트가 있는 경우 모든 클라이언트가 사용할 수 있는 정식 모델(Canonical Model)을 사용하는 것이 좋습니다. 두 경우 모두 **DTO 모델**은 **트랜잭션을 나타내고 처리하기 위해 만들어진 도메인 모델**과 매우 다릅니다.

이 밖에 많은 영역에서 문제가 발견될 수 있습니다.

* 리포지토리의 많은 읽기 메서드는 종종 페이징 또는 정보 정렬을 포함합니다.
* DTO를 구축하기 위해 도메인 객체의 내부 상태를 노출 시키는 Getter 메서드들.
* Use of prefetch paths on the read use cases as they require more data to be loaded by the ORM.(?)
* DTO를 작성하기 위해 여러 애그리거트 루트를 로드하면 데이터 모델에 최적이 아닌 쿼리가 발생합니다. 또는 애그리거트 경계가 DTO 구축 작업으로 인해 혼동 될 수 있습니다.

가장 큰 문제는 쿼리 최적화가 매우 어렵다는 것입니다. 쿼리가 객체 모델에서 작동하고 ORM에 의해 데이터 모델로 변환되기 때문에 이러한 쿼리를 최적화하는 것이 어려워 질 수 있습니다. 개발자는 ORM과 데이터베이스에 대해 자세히 알고 있어야합니다. 개발자는 임피던스 불일치 문제를 다루고 있습니다(자세한 내용은 "Events as a Storage Mechanism"참조).

The largest smell though is that the optimization of queries is extremely difficult. Because queries are operating on an object model then being translated to a data model, likely by an ORM it can become difficult to optimize these queries. A developer needs to have intimate knowledge of the ORM and the database. The developer is dealing with a problem of Impedance Mismatch (for more discussion see “Events as a Storage Mechanism”).

CQRS를 적용한 후에는 자연적인 경계가 형성됩니다. 분리된 경로는 명백함을 만듭니다. 이제 DTO를 투영하기 위해 도메인을 사용하지 않는 것이 좋습니다. 대신 DTO를 투영하는 새로운 방식을 도입하는 것이 가능합니다.

***

![](https://user-images.githubusercontent.com/42791260/57115266-dfd88680-6d88-11e9-98b3-39373f0fd42e.png)

***

**\[그림 10] The Query Side**

도메인 계층이 사라지고 새로운 계층인 \*\*얇은 읽기 계층(Thin Read Layer)\*\*가 등장했습니다. 이 계층은 데이터베이스로 부터 데이터를 읽어와 바로 DTO에 투영시킵니다. There are many ways that this can be done with handwritten ADO.NET and mapping code and a full blown ORM on the high end. Which choice is right for a team depends largely on the team itself and what they are most comfortable with. 아마 ORM이 제공하는 것의 상당 부분이 필요하지 않으며 수동으로 매핑 코드를 만드는 데 많은 시간을 낭비하게됩니다. 가능한 해결책은 작은 규칙 기반 매핑 유틸리티를 사용하는 것입니다.

얅은 읽기 계층은 데이터베이스와 격리될 필요가 없으며, 읽기 계층에서 데이터베이스 벤더와 연결되는 것이 반드시 나쁜 것은 아닙니다. 읽기를 위해 저장 프로 시저를 사용하는 것은 반드시 나쁜 것은 아니며 팀의 기능과 시스템의 비기능적 요구 사항에 따라 달라집니다.

얅은 읽기 계층는 유지 보수가 지루할 수 있지만 복잡한 코드는 아닙니다. 리드 계층을 분리함으로써 얻을 수 있는 이점 중 하나는 임피던스 불일치 문제에 고통 받지 않는다는 것입니다. 이 모델은 데이터 모델에 직접 연결되어 있으므로 쿼리를 훨씬 쉽게 최적화 할 수 있습니다. 시스템의 쿼리 사이드에서 작업하는 개발자는 도메인 모델이나 ORM 도구가 사용되는 모든 것을 이해할 필요가 없습니다. 가장 단순한 수준에서 데이터 모델만 이해하면됩니다.

Thin Read Layer와 읽기용 도메인 바이 패스의 분리는 도메인의 전문화를 가능하게합니다.

### The Command Side

전체적으로 커맨드 사이드는 "전통적인 아키텍처"와 매우 유사합니다. \[그림 11]의 그림은 이전에 언급된 아키텍처와 거의 동일해야합니다. 주요 차이점은 DDD를 적용하기 위해 필요한 데이터 중심이 아닌 동작(behavioral)이 있으며 읽기가 분리됬다는 것입니다.

***

![](https://user-images.githubusercontent.com/42791260/57116016-5d9e9100-6d8d-11e9-94a5-ee9b7bddabd7.png)

***

**\[그림 11] The Command Side**

"전통적인 아키텍처"에서는 도메인이 명령과 쿼리를 모두 처리하고 있었기 때문에 도메인 자체에서 많은 문제가 발생했습니다. 그 중 일부는 다음과 같습니다.

* 리포지토리의 많은 읽기 메서드는 종종 페이징 또는 정보 정렬을 포함한다.
* DTO를 구축하기 위해 도메인 객체의 내부 상태를 공개하는 Getter 메서드들이 있다.
* Use of prefetch paths on the read use cases as they require more data to be loaded by the ORM.
* DTO를 만들기 위해 여러 애그리거트를 로드하면 데이터 모델에 최적이 아닌 쿼리가 발생합니다. 또는 애그리거트의 경계가 DTO 구축 작업으로 인해 혼동 될 수 있습니다.

읽기 계층이 분리되면 도메인은 명령 처리에만 초점을 맞춥니다. 도메인 객체는 갑자기 내부 상태를 노출 할 필요가 없으며, 저장소는 GetById를 제외하고 쿼리 메소드가 거의 없습니다. 또한 애그리거트 경계에서 보다 많은 동작에 초점을 맞출 수 있습니다.

Once the read layer has been separated the domain will only focus on the processing of Commands. These issues also suddenly go away. Domain objects suddenly no longer have a need to expose internal state, repositories have very few if any query methods aside from GetById, and a more behavioral focus can be had on Aggregate boundaries.

변경에 필요한 비용은 기존의 아키텍처에 비해 더 저렴하거나 전혀 들지 않습니다. 대부분 도메인 모델에 구현된 경우보다 쿼리의 최적화가 단순한 읽기 계층에서 더 간단하므로 비용이 많이 들지 않습니다. 아키텍처는 또한 쿼리가 분리되어있을 때 도메인 모델로 작업 할 때 개념 conceptual overhead가 낮습니다. 이것은 또한 낮은 비용으로 이어질 수 있습니다. 최악의 경우, 비용은 동일해야합니다. 실제로 완료된 모든 것은 책임의 이동이며, 읽기 사이드에서 여전히 도메인을 사용하도록하는 것이 가능합니다.

CQRS를 적용함으로써 읽기와 쓰기가 분리되었습니다. 실제로 두 모델이 동일한 데이터 모델을 읽어야하는지 아니면 두 개의 통합 된 시스템처럼 처리되는지 여부에 대한 질문을 제기합니다. \[그림 12]는 이러한 개념을 보여줍니다. 일관성 또는 궁극적으로 일관된 방식으로 동기화를 유지하기 위해 여러 데이터 소스간에 잘 알려진 통합 패턴이 많이 있습니다. 두 개의 별개의 데이터 소스를 통해 데이터 모델을 현재 작업에 맞게 최적화 할 수 있습니다. 예를 들어 읽기 측은 1NF로 모델링 할 수 있고 트랜잭션 모델은 3NF로 모델링 할 수 있습니다.

**통합 모델의 선택은 모델 간의 변환과 동기화가 매우 힘은 일이 될 수 있기 때문에 매우 중요한 쟁점입니다. 가장 적합한 모델은 이벤트의 도입입니다. 이벤트는 잘 알려진 통합 패턴이며 모델 동기화를 위한 최상의 메커니즘을 제공합니다.**

***

![](https://user-images.githubusercontent.com/42791260/57121325-c861c380-6db1-11e9-9be2-f0aa9846bb2a.png)

***

**\[그림 12] Separated Data Models with CQRS**

## Events as a Storage Mechanism

## Reference

* **CQRS Documents by Greg Young**
