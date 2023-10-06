---
emoji: 🪧
title: 바운디드 컨텍스트는 무엇인가요
date: '2023-09-19 17:38:00'
author: 유자
tags: DDD model bounded-context context-map
categories: challenge
---

## 패키지 분리를 이렇게 저렇게
'패키지 분리를 바운디드 컨텍스트 기준으로 한다'는 무슨 뜻일까요. 전 회사 테크 리드님이 '저희는 바운디드 컨텍스트 기준으로 패키지를 분리하고 있습니다' 라고 말씀해주실 때마다 '오 그렇구나.. 근데 그게 뭐죠..?' 싶은 순간이 있었습니다. 현생에 치여 제대로 알아보지 못한 내용을 이제 한번 알아보려고 합니다.

## 바운디드 컨텍스트여 내게로 오라
### DDD 였구나
DDD(Domain Driven Design)은 도메인 모델을 기반으로 소프트웨어를 설계하는 것입니다. 규모가 큰 시스템의 도메인일수록 하나의 모델로 만드는 것은 어려워집니다. 그래서 이 큰 도메인의 모델을 나누고 바운디드 컨텍스트로 감싸는 방식의 소프트웨어를 설계합니다.
바운디드 컨텍스트는 모델이 적용되는 범위라고 생각하면 됩니다. 도메인의 특정 부분을 떼어낸 모델은 유비쿼터스 언어로도 사용될 수 있어야 하는데 이때 사용되는 용어들의 의미를 구분하고 한정하는 역할을 합니다. 

### 도메인은 무엇인가요?
도메인은 소프트웨어가 해결하려는 문제 영역입니다. 특정 비즈니스나 산업, 조직의 핵심 지식과 활동, 그리고 그에 관련된 개념과 규칙을 포함합니다.
예를 들어 은행 도메인은 계좌 개설, 이체, 대출 승인 등에 대한 규칙이 포함될 수 있습니다. 도메인에 대한 전문 지식을 갖고 있는 전문가가 따로 있고 그와 함께 규칙을 세울 수 있습니다. 

그럼 비즈니스 로직과의 차이는 무엇일까요? 비즈니스 로직은 도메인 로직을 사용해서 어플리케이션에서 수행되는 전체적인 비즈니스 프로세스를 나타냅니다. 애플리케이션의 작업 흐름, 유효성 검사, UI와의 상호작용 등이 포함됩니다. 때문에 개발에서 도메인 로직보다는 더 넓은 의미로 사용됩니다. 

### 모델은 무엇인가요? 
도메인 주도 설계에서 모델은 특정 도메인의 주요 개념과 관계를 추상화하여 표현하는 것입니다. 추상화된 것은 유비쿼터스 언어로 사용할 수 있어야 합니다. 그래서 개발자와 도메인 전문가 사이의 공통의 언어로 의사소통할 수 있는 수단이 됩니다.

### 모델링은 어떻게 할 수 있나요? 
도메인 모델을 구성하는 핵심 요소로 엔터티(Entity), 값 객체(Value Object), 집합체(Aggregate)가 있습니다.

엔터티는 고유한 식별자를 가지고 시간에 따라 상태가 변경될 수 있습니다. 예를 들어 사용자, 주문, 상품 등은  고유한 ID를 가지고 있고 시간에 따라 상태가 변할 수 있기 때문에 엔터티로 모델링됩니다. 주로 핵심 역할을 하는 객체를 표현할 때 사용됩니다.

값 객체는 식별자를 가지지 않고 그 자체로 값이 됩니다. 값 객체는 한번 생성된 이후 상태가 변하지 않기 때문에 불변성을 가집니다. 예를 들어 날짜, 주소, 색상 등 처럼 값 자체로 의미를 가지게 됩니다. 변경이 필요한 경우 객체를 새로 만들어서 사용합니다. 

집합체는 엔터티와 값 객체의 그룹입니다. 집합체는 하나의 루트 엔터티를 기반으로 하고, 외부에서는 루트 엔터티를 통해서만 집합체 내의 다른 객체에 접근할 수 있습니다. 예를 들어 주문 집합체는 주문 엔터티를 루트로 가지고 주문 항목, 주소 등의 값 객체를 포함할 수 있습니다. 도메인 모델의 일관성을 유지하고 불필요한 외부 접근을 막는데 유용합니다.

각 요소별로 간단하게 구현해보았습니다.

1. 엔터티
```js
class Product {
  constructor(id, name, price) {
    this.id = id; // 고유한 식별자
    this.name = name;
    this.price = price;
  }

  updatePrice(newPrice) {
    this.price = newPrice;
  }
}
```
엔터티로 모델링된 `Product`는 `id`라는 식별자를 가지고 `price`라는 상태를 업데이트할 수 있는 메소드를 가지고 있습니다.

2. 값 객체
```js
class Address {
  constructor(street, city, zipCode) {
    this.street = street;
    this.city = city;
    this.zipCode = zipCode;
  }

  equals(otherAddress) {
    return this.street === otherAddress.street &&
           this.city === otherAddress.city &&
           this.zipCode === otherAddress.zipCode;
  }
}

```
값 객체로 모델링된 `Address`는 식별자가 없고, 다른 주소와 같은지 비교하는 메소드를 가지고 있을 뿐, 어떤 상태도 업데이트 할 수 없습니다.

3. 집합체
```js
class PaymentInfo {
  constructor(cardNumber, expirationDate, cvv) {
    this.cardNumber = cardNumber;
    this.expirationDate = expirationDate;
    this.cvv = cvv;
  }

  equals(otherPaymentInfo) {
    return this.cardNumber === otherPaymentInfo.cardNumber &&
           this.expirationDate === otherPaymentInfo.expirationDate &&
           this.cvv === otherPaymentInfo.cvv;
  }
}

class OrderItem {
  constructor(product, quantity) {
    this.product = product;
    this.quantity = quantity;
  }

  updateQuantity(quantity) {
   this.quantity = quantity;
  }

  getTotalPrice() {
    return this.product.price * this.quantity;
  }
}

class Order {
  constructor(orderId, customer) {
    this.orderId = orderId;
    this.customer = customer;
    this.orderItems = [];
    this.shippingAddress = null;
    this.paymentInfo = null;
  }

  setShippingAddress(address) {
    this.shippingAddress = address;
  }

  setPaymentInfo(paymentInfo) {
   this.paymentInfo = new PaymentInfo(paymentInfo)
  }

  addOrderItem(product, quantity) {
    const orderItem = new OrderItem(product, quantity);
    this.orderItems.push(orderItem);
  }

  getTotalOrderPrice() {
    return this.orderItems.reduce((total, item) => total + item.getTotalPrice(), 0);
  }
}

```
집합체로 모델링된 `Order`는 `OrderItem`과 `PaymentInfo`를 포함합니다. `OrderItem`은 `product`를 고유한 값으로 가지고 `quantity`를 수정할 수 있는 엔터티이고, `PaymentInfo`는 값 객체입니다. 

가장 기본이 되는 위 3개의 구성 요소 외에 도메인 서비스, 어플리케이션 서비스, 레포지토리, 팩토리, 도메인 이벤트로 모델을 구현할 수 있습니다. 

도메인 서비스는 도메인 로직이 엔터티나 값 객체에 위치하는게 자연스럽지 않을 때 사용됩니다. 예를 들어 두 사용자 간의 비밀번호 일치 여부를 확인하는 로직은 `UserService`라는 도메인 서비스에 위치할 수 있습니다. 

어플리케이션 서비스는 앱의 작업 흐름과 비즈니스 로직을 조정하고 외부 시스템과의 인터페이스 역할을 합니다. 주로 UI나 외부 시스템에서 호출되고, 도메인 객체들과의 협력을 통해 비즈니스 로직을 실행합니다. Use Case를 구현하는데 사용될 수 있습니다. 

레포지토리는 데이터베이스나 다른 저장소와의 인터페이스 역할을 하며 도메인 모델과 저장소 간의 중개자 역할을 합니다. 

팩토리는 복잡한 객체 생성 로직을 캡슐화하여 객체의 생성과 관련된 로직을 분리하는데 사용됩니다. 객체 생성 과정이 복잡하고 여러 단계를 거쳐야할 때 팩토리를 사용하면 단계를 더 명확하게 관리할 수 있습니다. 

도메인 이벤트는 도메인 내에서 중요한 사건이나 변화를 표현하는 이벤트입니다. 예를 들어 주문이 완료되었을 때 발생하는 `orderCompletedEvent`와 같은 것을 사용하여 다른 시스템이나 모듈에 해당 사실을 알릴 수 있습니다.

그럼 위에 언급된 집합체는 엔터티, 값 객체가 함께 있는 것이라고 했는데, 이것 말고도 다른 요소가 들어갈 수 있는 것일까요? 맞습니다. 도메인 서비스, 팩토리, 도메인 이벤트가 들어갈 수 있습니다. 
어플리케이션 서비스나 레포지토리는 [소프트웨어 아키텍쳐 계층](https://dev.to/moyousry/ddd-layered-architecture-2f73)을 나누었을 때, 각각 Application Layer, Infrastructure Layer로 분류할 수 있기 때문에 Domain Layer로 구분되는 집합체에 다른 요소들과 그루핑되지 않습니다.


### 컨텍스트 맵은 무엇인가요?
전 직장에서 소스코드를 패키지로 분리하다보니 패키지들 중에서도 저수준, 고수준이 대략적으로 보였습니다. 분리된 패키지에는 App 패키지, User 패키지, 디자인 시스템 패키지, 유틸 패키지 등 종류가 다양했는데요. Nx에서 제공하는 그래프 기능으로는 패키지별 의존성 방향만 확인할 수 있었습니다. 패키지의 역할이 다양했기 때문에 그들의 관계를 의존성으로만 표현하기엔 부족하다는 생각이 들었습니다. 역시나 바운디드 컨텍스트 간의 관계를 정의하는 컨텍스트 맵이 있었습니다. 정의할 수 있는 관계에는 어떤 것들이 있는지, 현업에서 분리했던 패키지들에는 어떤 관계를 부여할 수 있을지 알아보겠습니다.

1. Shared kernel (공유 커널)
두 바운디드 컨텍스트가 공유하는 핵심 도메인 모델 또는 코드를 나타냅니다. 이 관계에서는 두 팀이 함께 핵심 부분을 유지 관리합니다. 

2. Customer/Supplier (고객/공급자)
한 바운디드 컨텍스트(고객)이 다른 바운디드 컨텍스트(공급자)에 의존하며, 공급자는 고객의 요구 사항을 충족시키기 위해 작업합니다.

3. Conformist(순응자)
한 바운디드 컨텍스트가 다른 바운디드 컨텍스트의 모델과 API를 수용하며 독립적인 모델링을 추구하지 않습니다. 이 관계는 리드하는 컨텍스트의 모델이 충분히 좋거나, 순응자 컨텍스트가 모델링에 크게 관여하지 않을 때 유용합니다.

4. AntiCorruption Layer (ACL, 부패 방지 계층)
한 바운디드 컨텍스트가 다른 바운디드 컨텍스트와의 직접적인 의존성을 피하기 위해 중간 계층을 둡니다. ACL은 외부 시스템의 모델이 내부 시스템에 부정적인 영향을 주지 않도록 보호합니다. 

5. Open Host Service (개방형 호스트 서비스)
한 바운디드 컨텍스트가 다른 바운디드 컨텍스트에게 표준화된 API나 프로토콜을 제공합니다. 이 API는 안정적이고 자주 변경되지 않아야 합니다.

6. Published Language (공개 언어)
여러 바운디드 컨텍스트 간에 공유되는 표준화된 언어나 데이터 구조를 제공합니다.

7. Separate Ways 
두 바운디드 컨텍스트가 서로 독립적으로 발전하며, 통합을 추구하지 않는 관계입니다.

8. Big Ball of Mud 
명확한 경계나 구조 없이 여러 바운디드 컨텍스트가 혼재된 형태입니다. 피하는게 좋습니다.

#### [적용해보자 1] App과 디자인시스템 패키지의 관계
 Customer/Supplier라고 할 수 있을 것 같습니다. App이 Customer가 되고 디자인시스템은 Supplier입니다. 앱에서 요구하는 UI를 디자인 시스템에서 제공하고 있기 때문입니다. 물론 디자인시스템을 UI의 SSOT로 삼고 UI 디자인이 나와준다면 좋겠지만, 해당 프로젝트의 경우 antd 라는 UI 라이브러리를 메인으로 사용했고 디자인 시스템 패키지 내부에는 antd에서 제공하지 않는 커스텀 UI 또는 아이콘 등을 분류해놨습니다. Customer의 필요에 의해 디자인 시스템을 계속 수정해나갔기 때문에 Supplier가 적절하다고 생각했습니다.

#### [적용해보자 2] App과 multi-use 패키지의 관계
multi-use는 도메인의 다회성 기능과 관련된 패키지입니다. 도메인 로직 수정 사항으로 multi-use가 변경되면 App은 순응해야하므로 Conformist로 보았습니다. 또한 multi-use는 핵심 도메인으로 분류된 패키지로 다른 많은 패키지에서 multi-use를 사용하고 있습니다. 때문에 Shared Kernel로 볼 수 있습니다.

#### [적용해보자 3] multi-use와 date-util 패키지의 관계 
date-util은 앱 내에서 날짜를 다루는 여러 기능을 제공하는 패키지입니다. 핵심 도메인 로직은 아닙니다. 서로 다른 바운디드 컨텍스트나 서버 통신을 위해 공통된 (날짜)데이터 포맷을 제공하고 있습니다. 그래서 Published Language로 볼 수 있을 듯 합니다. 

#### [적용해보자 4] App과 다른 App 패키지의 관계
Seperate Ways로 보았습니다. 두 개의 App 패키지가 있고, 배포도 따로 되고 있기 때문에 독립적으로 발전해야한다고 생각합니다. 

## 결론
바운디드 컨텍스트는 도메인 주도 설계에서 사용되는 용어입니다. 도메인 주도 설계는 모델이 기반이 되어 소프트웨어를 설계하는데, 이때 모델이 적용되는 경계를 바운디드 컨텍스트라 합니다. 다양한 의미를 가질 수 있는 모델(=단어)를 바운더리 안에 집어넣어서 의미를 축소시킵니다. 이로 인해 다른 의미로 해석될 수 있는 여지를 줄일 수 있고 모델의 복잡성을 관리하거나 모델 간의 충돌을 방지합니다. 

'팀의 크기와 구조가 아키텍쳐를 모방한다'는 Conway의 법칙은 팀의 조직 구조가 시스템의 아키텍쳐와 밀접한 관련이 있다는 것을 의미합니다.

컨텍스트 매핑까지 살펴보니 규모가 큰 시스템을 가진 곳에서 도메인을 모델로 잘게 분리하고 바운디드 컨텍스트에 맞게 조직을 구성할 수 있다는 것을 알게 되었습니다. 서로 다른 바운디드 컨텍스트를 가진 팀들이 협업해야할 때, 컨텍스트 매핑을 통해 상호 작용의 방향성을 정의하고 관리할 수 있습니다. 

전 직장에서 해본 패키지 분리는 한 팀 내에서 소스코드의 의존성을 관리하며 효과적으로 작업하기 위해 분리한 것이었습니다. 그래서 패키지간 상호 작용에 대해서 깊게 고민해본 적이 없었습니다. 이번 기회를 통해 자세하게 알아볼 수 있어 재미있었습니다. 실제로 패키지 분리를 진행하는 동안 이에 대해 알고 있었다면 더욱 재밌는 작업이 되었을 것 같은데 그러지 못해 살짝 아쉽지만 이렇게라도 한발짝씩 나아가면서 발전하리라 믿습니다. 


## 참고 문서
- 마틴 파울러의 Bounded Context [https://martinfowler.com/bliki/BoundedContext.html](https://martinfowler.com/bliki/BoundedContext.html)
	
- 마이크로서비스 모델링 ③ : DDD의 전략적 설계 [https://engineering-skcc.github.io/microservice%20modeling/ddd-Srategic-design/#%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-%EB%A7%B5](https://engineering-skcc.github.io/microservice%20modeling/ddd-Srategic-design/#%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-%EB%A7%B5)

- DDD Quickly ( 도메인 주도 설계란 무엇인가?) 정리 [https://read-write-developer.tistory.com/139](https://read-write-developer.tistory.com/139)

```toc
```




