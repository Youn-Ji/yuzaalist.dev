---
emoji: 🪧
title: 바운디드 컨텍스트 이 좌시가
date: '2023-09-16 17:38:00'
author: 유자
tags: bounded-context DDD
categories: challenge
---

## 서론
패키지 분리를 바운디드 컨텍스트 기준으로 한다는 무슨 뜻일까. 프론트 테크 리드님이 '저희는 바운디드 컨텍스트 기준으로 패키지를 분리하고 있습니다' 라고 하실 때마다 음 그렇지 그렇지 
근데 그게 뭐라꽁..? ㅋㅋ 싶은 순간이 있었다. 이김에 바운디드 컨텍스트가 뭔지? 코드 상으로 대략 어떻게 적용되었는지 확인해보려고 한다.

## 본론
### DDD 였구나.. 
DDD는 도메인의 모델을 기반으로 소프트웨어를 설계하는 것이다. 규모가 큰 시스템의 도메인일수록 하나의 모델로 만드는 것은 어려워진다. 그래서 이 큰 도메인을 바운디드 컨텍스트로 나누고 각 컨텍스트 안에서 모델을 갖게 해서 소프트웨어를 설계하는 것이 DDD이다.
바운디드 컨텍스트는 모델이 적용되는 범위라고 생각하면 된다. 즉 바운디드 컨텍스트는 도메인의 특정 부분에 대한 용어와 의미를 한정하고 구분짓는 역할을 한다. 이건 도메인 모델의 복잡성을 낮춰준다.
이런걸 다중 표준 모델을 구조화하는 방식이라고 한다는데 이건 나중에 보자 생략할 수도 있음 
이렇게 생긴 여러 바운디드 컨텍스트 간 관계를 컨텍스트 맵이라는 것으로 묘사한다. 

### 도메인은 무엇이당가?
도메인은 소프트웨어가 해결하려는 문제의 영역이다. 특정 비즈니스나 산업, 조직의 핵심 지식과 활동, 그리고 그에 관련된 개념과 규칙을 포함하는 영역이다. 
예를 들어 은행 도메인은 계좌 개설, 이체, 대출 승인 등에 대한 규칙이 포함될 수 있다.
도메인에 대한 전문 지식을 갖고 있는 전문가가 따로 있고 그와 함께 규칙을 세워야 한다. 

그럼 비즈니스 로직과의 차이는 무엇일까? 비즈니스 로직은 도메인 로직을 사용해서 어플리케이션에서 수행되는 전체적인 비즈니스 프로세스를 나타낸다. 애플리케이션의 작업 흐름, 유효성 검사, UI와의 상호작용 등이 포함된다. 도메인 로직보다는 더 넓은 의미로 사용된다. 

### 모델은 무엇이당가? 
도메인 주도 개발에서 모델은 특정 도메인의 주요 개념과 관계를 추상화하여 표현한 것이다. 복잡한 비즈니스 로직과 요구사항을 단순화해서 개발자와 도메인 전문가 사이의 공통의 언어로 의사소통할 수 있는 수단을 제공한다. 
도메인 모델을 구성하는 핵심 요소로 엔터티(Entity), 값 객체(Value Object), 집합체(Aggregate)가 있다.
엔터티는 고유한 식별자를 가지고 시간에 따라 상태가 변경될 수 있다. 예를 들어 사용자, 주문, 상품등은 각각의 고유한 ID를 가지고 있고 시간에 따라 상태가 변할 수 있기 때문에 엔터티로 모델링된다. 도메인 모델에서 핵심적인 역할을 하는 객체를 표현할 때 사용된다.
값 객체는 식별자를 가지지 않고 그 자체로 값이 된다. 값 객체는 한번 생성된 이후 상태가 변하지 않기 때문에 불변성을 가진다. 예를 들어 날짜, 주소, 색상 등처럼 그 값 자체로 의미를 가지게 된다. 변경이 필요한 경우 객체를 새로 만들어서 사용한다. 
집합체는 엔터티와 값 객체의 그룹이다. 집합체는 하나의 루트 엔터티를 기반으로 외부에서는 루트 엔터티를 통해서만 집합체 내의 다른 객체에 접근할 수 있다. 예를 들어 주문 집합체는 주문 엔터티를 루트로 가지고 주문 항목, 주소 등의 값 객체를 포함할 수 있다. 도메인 모델의 일관성을 유지하고 불필요한 외부 접근을 막는데 좋다.


### 모델링은 어떻게 할 수 있는가? 
자바스크립트로 각 요소별로 구현해보았다. 
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
엔터티로 모델링된 Product는 id라는 식별자를 가지고 price라는 상태를 업데이트할 수 있는 메소드를 가지고 있다.

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
값 객체로 모델링된 Address는 식별자가 없고, 다른 주소와 같은지 비교하는 메소드를 가지고 있을 뿐, 어떤 상태도 업데이트 할 수 없다.

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
집합체로 모델링된 Order는 OrderItem과 PaymentInfo를 포함합니다. OrderItem은 product를 고유한 값으로 가지고 quantity를 수정할 수 있는 엔터티이고, PaymentInfo는 값 객체입니다. 

위 3개가 가장 기본이 되는 구성 요소이다. 그 외에 도메인 서비스, 어플리케이션 서비스, 레포지토리, 팩토리, 도메인 이벤트로 모델을 구현할 수 있다. 

도메인 서비스는 도메인 로직이 엔터티나 값 객체에 위치하는게 자연스럽지 않을 때 사용된다. 예를 들어 두 사용자 간의 비밀번호 일치 여부를 확인하는 로직은 'UserService'라는 도메인 서비스에 위치할 수 있다. 

어플리케이션 서비스는 앱의 작업 흐름과 비즈니스 로직을 조정하고 외부 시스템과의 인터페이스 역할을 한다. 주로 UI나 외부 시스템에서 호출되고, 도메인 객체들과의 협력을 통해 비즈니스 로직을 실행한다. use case를 구현하는데 사용될 수 있다. 

레포지토리는 데이터베이스나 다른 저장소와의 인터페이스 역할을 하며 도메인 모델과 저장소 간의 중개자 역할을 한다. 

팩토리는 복잡한 객체 생성 로직을 캡슐화하여 객체의 생성과 관련된 로직을 분리하는데 사용된다. 객체 생성 과정이 복잡하고 여러 단계를 거쳐야할 때 팩토리를 사용하면 단계를 명확하게 관리할 수 있다. 

도메인 이벤트는 도메인 내에서 중요한 사건이나 변화를 표현하는 이벤트이다. 예를 들어 주문이 완료되었을 때 발생하는 orderCompletedEvent와 같은 이벤트를 사용하여 다른 시스템이나 모듈에 해당 사실을 알릴 수 있다.

위에 언급된 집합체는 엔터티, 값 객체 말고도 도메인 서비스, 팩토리, 도메인 이벤트가 들어갈 수 있다. 
어플리케이션 서비스나 레포지토리는 계층을 나누었을 때, 각각 Application Layer, Infrastructure Layer로 분류할 수 있기 때문에 Domain Layer로 구분되는 집합체에 다른 요소들과 그루핑되지 않는다.

소프트웨어 아키텍쳐를 계층으로 나눈다면, Presentation, Application, Domain, Infrastructure가 있다. 

### 컨텍스트 매핑은 무엇이고, 패키지들에 적용하면 어떻게 볼 수 있을까? 
바운디드 컨텍스트끼리를 컨텍스트 매핑으로 관계를 정의할 수 있다고 한다. 어떤 종류가 있는지 보겠다. 

## 결론
바운디드 컨텍스트는 도메인 주도 설계에서 사용되는 용어인데용. 도메인 주도 설계에서 모델이 기반이 되어 소프트웨어를 설계하게 되는데 이떄, 도메인 모델이 적용되는 경계를 의미해요. 다양한 의미를 가질 수 있는 용어를 상위그룹의 하위 그룹으로 이동시켜서 의미를 한정시켜서 이해할 수 있게끔 도와줍니다.  이렇게 경계를 그어줌으로써 모델의 복잡성을 관리하고 모델간의 충돌을 방지시켜줍니다. 

## 참고 문서
마틴 파울러의 Bounded Context [https://martinfowler.com/bliki/BoundedContext.html](https://martinfowler.com/bliki/BoundedContext.html)

```toc
```




