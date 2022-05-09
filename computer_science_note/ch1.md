# Chapter 1

## 디자인 패턴과 프로그래밍 패러다임

### 1.1 디자인 패턴

디자인 패턴이란 프로그램을 설계할 때 발생했던 문제점들을 객체 간의 상호 관계 등을 이용하여 해결할 수 있도록 하나의 '규약' 형태로 만들어 놓은 것을 의미한다.

### 싱글톤 패턴

싱글톤 패턴(singleton pattern)은 하나의 클래스에 오직 하나의 인스턴스만 가지는 패턴이다. 보통 데이터베이스 연결 모듈에 많이 사용한다.

하나의 인스턴스를 만들어 놓고 해당 인스턴스를 다른 모듈들이 공유하며 사용하기 때문에 인스턴스를 생성할 때 드는 비용이 줄어드는 장점이 있다. 하지만 의존점이 높아진다는 단점을 가지고 있다.

#### 자바스크립트의 싱글톤 패턴

```javascript
const obj = {
  a: 27
};

const obj2 = {
  a: 27
};

console.log(obj === obj2); // false
```

자바스크립트에서는 리터럴 {} 또는 new Object로 객체를 생성하게 되면 다른 어떤 객체와도 같지 않기 때문에 이 자체만으로 싱글톤 패턴을 구현할 수 있다.

```javascript
class Singleton {
  constructor() {
    if (!Singleton.instance) {
      Singleton.instance = this;
    }
    return Singleton.instance;
  }
  getInstance() {
    return this.instance;
  }
}

const a = new Singleton();
const b = new Singleton();

console.log(a === b); // true
```

위의 코드는 Singleton.instance라는 하나의 인스턴스를 가지는 Singleton 클래스를 구현한 코드다. a와 b는 하나의 인스턴스를 가진다.

#### 데이터베이스 연결 모듈

앞서 말했듯 싱글톤 패턴은 데이터베이스 연결 모듈에 많이 쓰인다.

```javascript
const URL = 'mongodb://localhost:27017/kundolapp';
const createConnection = url => ({ url: url });

class DB {
  constructor(url) {
    if (!DB.instance) {
      DB.instance = createConnection(url);
    }
    return DB.instance;
  }
  connect() {
    return this.instance;
  }
}

const a = new DB(URL);
const b = new DB(URL);

console.log(a === b); // true
```

이렇게 DB.instance라는 하나의 인스턴스를 기반으로 a, b를 생성하는 것을 볼 수 있다. 이를 통해 데이터베이스 연결에 관한 인스턴스 생성 비용을 아낄 수 있다.

실제로 싱글톤 패턴은 Node.js에서 Mongo DB 데이터베이스를 연결할 때 쓰는 mongoose 모듈에서 볼 수 있으며, MySQL 데이터베이스를 연결할 때도 싱글톤 패턴이 쓰인다.

#### 싱글톤 패턴의 단점

싱글톤 패턴은 TDD(Test Driven Development)를 할 때 걸림돌이 된다. TDD를 할 때 단위 테스트를 주로 하는데, 단위 테스트는 테스트가 서로 독립적이어야 하며 테스트를 어떤 순서로든 실행할 수 있어야 한다.

하지만 싱글톤 패턴은 미리 생성된 하나의 인스턴스를 기반으로 구현하는 패턴이므로 각 테스트 마다 '독립적인' 인스턴스를 만들기가 어렵다.

#### 의존성 주입

싱글톤 패턴은 사용하기 쉽고, 실용적이지만 모듈 간의 결합을 강하게 만들 수 있다는 단점이 있다. 이 때 의존성 주입(DI, Dependency Injection)을 통해 모듈 간의 결합을 조금 더 느슨하게 만들어 해결할 수 있다.

> 의존성 : 종속성이라고도 하며, A가 B에 의존성이 있다는 것은 B의 변경 사항에 대해 A 또한 변해야 된다는 것을 의미한다.

<br />

<p align="center" >
  <img src="./../images/DI.gif" />
</p>

위 그림처럼 메인 모듈(main module)이 **직접** 다른 하위 모듈에 대한 의존성을 주기보다는 중간에 dependecy injector이 이 부분을 가로채 메인 모듈이 **간접**적으로 의존성을 주입하는 방식이다.

이를 통해 메인 모듈은 하위 모듈에 대한 의존성이 떨어지게 된다. 참고로 이를 '디커플링이 된다'고도 한다.

**의존성 주입의 장점**

- 모듈들을 쉽게 교체할 수 있는 구조가 되어 테스팅하기 쉽고 마이그레이션하기도 수월하다.
- 애플리케이션을 쉽게 추론할 수 있으며, 모듈 간의 관계들이 좀 더 명확해진다.

**의존성 주입의 단점**

- 모듈들이 더욱 더 분리되므로 클래스 수가 늘어나 복잡성이 증가될 수 있으며 약간의 런타임 페널티가 생기기도 한다.

**의존성 주입 원칙**

의존성 주입은 다음과 같은 의존성 주입 원칙을 지키며 만들어야한다.

- 상위 모듈은 하위 모듈에서 어떠한 것도 가져오지 않아야 한다. 둘 다 추상화에 의존해야 하며, 이 때 추상화는 세부 사항에 의존하지 말아야한다.

### 팩토리 패턴

팩토리 패턴(factory pattern)은 객체를 사용하는 코드에서 객체 생성 부분을 떼어내 추상화한 패턴이자 상속 관계에 있는 두 클래스에서 상위 클래스가 중요한 뼈대를 결정하고, 하위 클래스에서 객체 생성에 관한 구체적인 내용을 결정하는 패턴이다.

상위클래스와 하위 클래스가 분리되기 때문에 느슨한 결합을 가지며 상위 클래스에서는 인스턴스 생성 방식에 대해 전혀 알 필요가 없기 때문에 더 많은 유연성을 갖게된다.
그리고 객체 생성 로직이 따로 떼어져 있기 때문에 코드를 리팩터링하더라도 한 곳만 고칠 수 있게 되니 유지 보수성이 증가된다.

#### 자바스크립트의 팩토리 패턴

자바스크립트에서 팩토리 패턴을 구현한다면 간단하게 new Object()로 구현할 수 있다.

```javascript
const num = new Object(42)
const str = new Object('abc)

num.constructor.name; // Number
str.constructor.name; // String
```

숫자를 전달하거나 문자열을 전달함에 따라 다른 타입의 객체를 생성하는 것을 볼 수 있다. 즉, 전달받은 값에 따라 다른 객체를 생성하며 인스턴스의 타입 등을 정한다.

```javascript
class Latte {
  constructor() {
    this.name = 'Latte';
  }
}

class Espresso {
  constructor() {
    this.name = 'Espresso';
  }
}

class LatteFactory {
  static createCoffee() {
    return new Latte();
  }
}

class EspressoFactory {
  static createCoffee() {
    return new Espresso();
  }
}

const factoryList = { LatteFactory, EspressoFactory };

class CoffeeFactory {
  static createCoffee(type) {
    const factory = factoryList[type];
    return factory.createCoffee();
  }
}

const main = () => {
  // 라떼를 주문한다.
  const coffee = coffeeFactory.createCoffee('LatteFactory');
  // 커피 이름을 부른다.
  console.log(coffee.name); // Latte
};

main();
```

CoffeeFactory라는 상위 클래스가 중요한 뼈대를 결정하고 하위 클래스인 LatteFactory가 구체적인 내용을 결정한다.

이것은 의존성 주입이라고도 볼 수 있다. CoffeeFactory의 인스턴스를 생성하는 것이 아닌 LatteFactory에서 생성한 인스턴스를 CoffeeFactory에 주입하고 있기 때문이다.

그리고 CoffeeFactory에서 static으로 createCoffee() 정적 메서드를 정의했는데, 정적 메서드를 쓰면 클래스의 인스턴스 없이 호출이 가능하여 메모리를 절약할 수 있고 개별 인스턴스에 묶이지 않으며 클래스 내의 함수를 정의할 수 있는 장점이 있다.

### 전략 패턴

전략 패턴(strategy pattern)은 정책 패턴이라고도 하며, 객체의 행위를 바꾸고 싶은 경우 '직접' 수정하지 않고 전략이라고 부르는 '캡슐화한 알고리즘'을 컨텍스트 안에서 바꿔주면서 상호 교체가 가능하게 만드는 패턴이다.

예를 들면, 우리가 어떤 것을 살 때 네이버페이, 카카오페이 등 다양한 방법으로 결제하듯이 어떤 아이템을 살 때 LUNACard로 결제하거나 KAKAOCard로 결제하는 것이 있겠다. 결제 방식의 '전략'만 바꿔서 두 가지 방식으로 결제하는 것이다.

#### passport의 전략 패턴

전략 패턴을 활용한 라이브러리로는 passport가 있다.

passport는 Node.js에서 인증 모듈을 구현할 때 쓰는 미들웨어 라이브러리로, 여러 가지 '전략'을 기반으로 인증할 수 있게 한다. 서비스 내의 회원가입된 아이디와 비밀번호를 기반으로 인증하는 LocalStrategy 전략과 페이스북, 네이버 등 다른 서비스를 기반으로 인증하는 OAuth 전략 등을 지원한다.

```javascript
var passport = require('passport'),
  LocalStrategy = require('passport-local').Strategy;

passport.use(
  new LocalStrategy(function (username, password, done) {
    User.findOne({ username: username }, function (err, user) {
      if (err) {
        return done(err);
      }
      if (!user) {
        return done(null, false, { message: 'Incorrect username.' });
      }
      if (!user.validPassword(password)) {
        return done(null, false, { message: 'Incorrect password.' });
      }
      return done(null, user);
    });
  })
);
```

`passport.use(new LocalStrategy(...))` 처럼 `passport.use()`라는 메서드에 **전략**을 매개 변수로 넣어서 로직을 수행하는 것을 볼 수 있다.
