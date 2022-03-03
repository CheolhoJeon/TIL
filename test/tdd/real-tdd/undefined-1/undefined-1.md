# 테스트 기법

### 수동 테스트

* 품질 담당자(QA)가 UI를 사용해 기능을 검증
* 최종 사용자의 사용 경험과 가장 비슷하게 검증
* <mark style="color:red;">**실행 비용이 높고 결과의 변동이 큼**</mark>
* 가장 온전한 코드 실행
* 인수 테스트

### 소프트웨어 회귀(Software regression)

* A software bug that **makes a feature stop functioning as intended after a certain event**
* QA가 매번 초록색 영역을 수동으로 테스트하기에는 비용 문제를 무시할 수 없음
* 많은 경우 새로 추가된 기능에 대해서만 수동 테스트를 진행함
* 기존에 구현되어 있던 기능에 대해서는 충실히 테스트하지 못하거나 생략하는 경우가 많음
* 때문에 테스트 자동화가 필요함

![](../../../../.gitbook/assets/IMG\_0013.PNG)

### 테스트 자동화

* **기능을 검증하는 코드**를 작성
* 테스트 코드 작성 비용이 소비되지만 **실행 비용이 낮고 결과의 신뢰도가 높음**
* 테스트 코드 작성과 관리가 **프로그래머 역량에 크게 영향을 받음**\
  ****

### 인수 테스트와 단위 테스트

![](../../../../.gitbook/assets/IMG\_0014.PNG)

{% tabs %}
{% tab title="인수 테스트" %}
* <mark style="color:blue;">**배치된 시스템을 대상**</mark>으로 검증
  * 만들어진 소프트웨어를 사용자에게 인수하기 전에 이루어지는 테스트
  * 시스템의 구성요소를 현장과 동일하거나 혹은 거의 유사한 환경으로 구축한 후 진행하는 테스트
* 전체 시스템 이상 여부에 대한 <mark style="color:blue;">**신뢰도가 높음**</mark>
* <mark style="color:blue;">**높은 비용**</mark>
  * 작성비용 / 관리비용 / 실행비용
* <mark style="color:blue;">**피드백 품질이 낮음**</mark>
  * 현상은 드러나지만 원은 숨겨 짐
{% endtab %}

{% tab title="단위 테스트" %}
* <mark style="color:blue;">**시스템의 일부(하위 시스템)을 대상**</mark>으로 검증
* <mark style="color:blue;">**낮은 비용**</mark>
  * 작성비용 / 관리비용 / 실행비용
* <mark style="color:blue;">**높은 피드백 품질**</mark>
* 전체 시스템 이상 여부 <mark style="color:blue;">**신뢰도가 낮음**</mark>
  * 단위 테스트가 실패한다면 전체 시스템이 온전히 동작하지 않을 확률이 매우 높지만,
  * 단위 테스트가 성공하더라도 시스템이 온전히 동작한다고 보장하기는 힘듦
{% endtab %}
{% endtabs %}
