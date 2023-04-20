# 설계 원칙

SOLID 원칙은 함수와 데이터 구조를 클래스로 배치하는 방법, 그리고 이들 클래스를 서로 결합하는 방법을 설명해준다. '클래스'라는 단어를 사용했다고 해서 SOLID원칙이 객체 지향 소프트웨어에만 적용된다는 뜻은 아니다. 클래스는 단순히 함수와 데이터를 결합하는 집합을 의미한다.

SOLID원칙의 목적은 중간 수준의 소프트웨어 구조가 아래와 같도록 만드는 데 있다.
1. 변경에 유연하다.
2. 이해하기 쉽다.
3. 많은 소프트웨어 시스템에 사용할 수 있는 컴포넌트의 기반이 된다.

중간 모듈: 프로그래머가 이들 원칙을 모듈 수준에서 작업할 때 적용할 수 있다는 뜻. 즉, 코드 수준보다는 조금 상위에서 적용되며 모듈과 컴포넌트 내부에서 사용되는 소프트웨어 구조를 정의하는 데 도움을 준다.

- SRP<단일 책임 원칙>: 각 소프트웨어 모듈은 변경의 이유가 하나, 단 하나여야만 한다.
- OCP<개방-폐쇄 원칙>: 요구사항에 있어 기존 코드를 수정하기보다는 반드시 새로운 코드를 추가하는 방식으로 시스템의 행위를 변경할 수 있도록 설계
- LSP<리스코프 치환 원칙>: 상호 대체 가능한 구성요소를 이용해 소프트웨어 시스템을 만들 수 있으려면, 이들 구성요소는 반드시 서로 치환 가능해야 한다.
- ISP<인터페이스 분리 원칙>: 소프트웨어 설계자는 사용하지 않은 것에 의존하지 않아야 한다.
- DIP<의존성 역전 원칙>: 고수준 정책을 구현하는 코드는 저수준 세부사항을 구현하는 코드에 절대로 의존해서는 안 된다. 대신 세부사항이 정책에 의존해야 한다.

---

# 단일 책임 원칙
**하나의 모듈은 하나의, 오직 하나의 시스템이 동일한 방식으로 변경되기를 원하는 사용자나 이해관계자 집단에 대해서만 책임져야 한다.**

여기서 모듈은 소스 파일을 의미하며, 단일 액터(이해관계 집단)를 책임지는 코드를 함께 묶어주는 힘이 바로 응집성이다.

``` swift
class Employee {
  func calculatePay() { }
  func reportHours() { }
  func save() { }
}
```

위 코드에서 세 가지 메서드를 Employee라는 단일 클래스에 배치하여 세 액터가 서로 결합된 상태이다. 해당 결합으로 입해 각각에 영향을 줄 수 있다. '단일 책임 원칙'은 서로 다른 액터가 의존하는 코드를 서로 분리하는 것으로 해결할 수 있다(우발적 중복의 문제). 또한 각 액터가 코드를 수정했을 때 충돌이 일어날 수 있으며, 이는 서로 다른 액터를 뒷받침하는 코드를 서로 분리하는 것으로 해결할 수 있다.(병합의 문제).


가장 확실한 방법은 각 메서드를 각각 다른 클래스로 이동시키는 것이다. 각 클래스는 서로의 존재를 모르게 된다. 하지만 개발자가 각 클래스를 인스턴스화하고 추적해야하는 단점이 있다. 

이를 해결하기 위해 '퍼사드 패턴 - Facade Pattern'이 있다. swift를 기반으로 작성된 코드는 아래와 같다.

``` swift
class PayCalculator {
  func calculatePay(for employee: Employee) {
    //...
  }
}

class HourReporter {
  func reportHours(for employee: Employee) {
    //...
  }
}

class EmployeeSaver {
  func save(employee: Employee) {
    //...
  }
}

class EmployeeFacade {
  let payCalculator: PayCalculator
  let hourReporter: HourReporter
  let employeeSaver: EmployeeSaver
    
    init(payCalculator: PayCalculator, 
         hourReporter: HourReporter, 
         employeeSaver: EmployeeSaver) {
         
        self.payCalculator = payCalculator
        self.hourReporter = hourReporter
        self.employeeSaver = employeeSaver
    }
    
    func calculatePay() {
        payCalculator.calculatePay(for: employee)
    }
    
    func reportHours() {
        hourReporter.reportHours(for: employee)
    }
    
    func save() {
        employeeSaver.save(employee: employee)
    }
}
```

현실적으로 모든 클래스가 하나의 메서드만 갖는 것은 불가능하다. 실제 구현에 필요한 다수의 private 메서드를 포함하게 된다. 이처럼 여러 메서드가 하나의 집단을 이루고, 메서드의 집단을 포함하는 각 클래스는 하나의 `유효범위`가 된다. 해당 유효범위 바깥에서는 이 집단에 감춰진 private 멤버가 있는지를 전혀 알 수 없다. -> `캡슐화`

**이처럼, 단일 책임 원칙은 메서드와 클래스 수준의 원칙이다. 하지만 더 상위 수준에서도 다른 형태로 등장하게 된다. 컴포넌트 수준에서는 `공통 폐쇄 원칙`이 되며 아키텍처 수준에서는 `아키텍처 경계`의 생성을 책임지는 변경의 축이 된다.**

---

# 개방 폐쇄 원칙
**소프트웨어 개체는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.** 즉, 개체의 행위는 확장할 수 있어야 하지만, 이때 개체를 `변경`해서는 안 된다.

책임을 분리하게 된다면, 여러 책임 중 하나에서 변경이 발생하더라도 다른 하나는 변경되지 않도록 소스 코드 의존성도 확실히 조직화해야 한다. 또한, 새롭게 조직화한 구조에서는 행위가 확장될 때 변경이 발생하지 않음을 보장해야 한다. 

A->B: A에서는 B를 호출하지만 B에서는 A를 호출하지 않는다. 즉, A컴포넌트에서 발생한 변경으로부터 B컴포넌트를 보호하려면 반드시 A컴포넌트가 B컴포넌트에 의존해야 한다. 

보호의 계층구조는 '수준'에 따르는 개념이다. 가장 높은 수준의 개념은 최고의 보호를 받아야한다. 

개발자는 기능이 어떻게, 왜, 언제 발생하는지에 따라서 기능을 분리하고, 분리한 기능을 컴포넌트의 계층구조로 조직화한다. 컴포넌트 계층구조를 이와 같이 조직화하면 저수준 컴포넌트에서 발생한 변경으로부터 고수준 컴포넌트를 보호할 수 있다. 

`추이 종속성`: A가 B에 의존하고, B가 C에 의존한다면, A는 C에 의존하게 된다. 이를 추이 종속성이라고 부른다. 만약 의존성이 순환적이라면, 모든 클래스가 서로 의존하게 되는 문제가 있다. 

추이 종속성을 가지게 되면, 엔티티는 '자신이 직접 사용하지 않는 요소에는 절대로 의존해서느 안된다'라는 원칙을 위반한다. 

Controller에서 발생한 변경으로부터 Interactor를 보호하는 일의 우선순위가 가장 높지만, 반대로 Interactor에서 발생한 변경으로부터 Controller도 보호되기를 바란다. 이를 위해 Interactor내부를 '은닉'한다. 

**개방 폐쇄 원칙의 목표는 시스템을 확장하기 쉬운 동시에 변경으로 인해 시스템이 너무 많은 영향을 받지 않도록 하는 데 있다. 이를 위해 시스템을 컴포넌트 단위로 분리하고, 저수준 컴포넌트에서 발생한 변경으로부터 고수준 컴포넌트를 보호할 수 있는 형태의 의존성 계층구조가 만들어지도록 해야한다.**

# 리스코프 치환 원칙
**S 타입의 객체 o1 각각에 대응하는 T 타입 객체 o2가 있고, T 타입을 이용해서 정의한 모든 프로그램 P에서 o2자리에 o1을 치환하더라도 P의 행위가 변하지 않으면, S는 T의 하위 타입이다.**

``` swift
class Billing {
  let license: License!
  
  init(license: License) {
    self.license = license
  }
}

class License {
  func calcFee()
}

class PersonalLicense: License { }
class BuisinessLicense: License { 
  var users: String?
}
```

Billing의 행위가 License 하위 타입 중 무엇을 사용하는지에 전혀 의존하지 않기 때문에 `리스코프 치환 원칙`을 준수한다. 

``` swift
class Rectangle {
  var width: Double
  var height: Double
  
  init(width: Double, height: Double) {
    self.width = width
    self.height = height
  }
  
  func setWidth(_ width: Double) {
    self.width = width
  }
  
  func setHeight(_ height: Double) {
    self.height = height
  }
  
  func area() -> Double {
    return width * height
  }
}

class Square: Rectangle {
  override var width: Double {
    didSet {
      height = width
    }
  }
  
  override var height: Double {
    didSet {
      width = height
    }
  }
  
  init(side: Double) {
    super.init(width: side, height: side)
  }
}

class User {
  func printArea(rectangle: Rectangle) {
    print("Area: \(rectangle.area())")
  }
}

let rectangle = Rectangle(width: 5, height: 10)
let square = Square(side: 7)

let a = User()
a.printArea(rectangle: rectangle) //50
let b = User()
b.printArea(rectangle: square)  //49
```

이 경우 Rectangle의 높이와 너비는 서로 독립적으로 변경될 수 있지만, Square의 높이와 너비는 반드시 함께 변경되기 때문에 User는 대화하고 있는 상대에 혼란이 생길 수 있다. 이를 해결하기 위해서는 Rectangle과 Square를 검사하는 메커니즘을 User에 추가하는 것이다. 하지만 이 경우 User의 `행위`가 사용하는 타입에 의존하기에 타입을 서로 치환할 수 없게 된다.

과거 LSP는 상속을 사용하도록 가이드하는 방법이었지만, 최근에는 인터페이스와 구현체에도 적용되는 광범위한 소프트웨어 설계 원칙으로 변경되었다. (protocol...)

**`리스코프 치환 원칙`은 아키텍처 수준까지 확장할 수 있고, 반드시 확장해야만 한다. 치환 가능성을 조금이라도 위배하면 시스템 아키텍처가 오염되어 상당량의 별도 메커니즘을 추가해야 할 수 있기 때문이다.**

---

# 인터페이스 분리 원칙
`swift는 정적 언어이다`

``` swift
class OPS {
  func op1() { }
  func op2() { }
  func op3() { }
}

class User1 {
  init() {
    var ops = OPS()
    ops.op1()
  }
}

class User2 {
  init() {
    var ops = OPS()
    ops.op2()
  }
}

class User3 {
  init() {
    var ops = OPS()
    ops.op3()
  }
}
```
위 코드에서 User1은 op2, op3을 전혀 호출하지 않지만, 두 메서드에 의존하게 된다. 이러한 의존성으로 인해 OPS 클래스에서 op2의 소스코드가 변경되면 User1도 다시 컴파일 후 새로 배포해야 한다. 


``` swift
protocol U1Ops {
  func op1()
}

protocol U2Ops {
  func op2()
}

protocol U3Ops {
  func op3()
}

class OPS: U1Ops, U2Ops, U3Ops {
  func op1() { }
  func op2() { }
  func op3() { }
}

class User1: U1Ops {
  init(ops: U1Ops) {
    ops.op1()
  }
}

class User2 {
  init(ops: U2Ops) {
    ops.op2()
  }
}

class User3 {
  init(ops: U3Ops) {
    ops.op3()
  }
}
```

위의 코드처럼 바꾸게 된다면, User1의 코드는 U1Ops와 op1에 의존하지만 OPS에는 의존하지 않게 된다. 따라서 OPS에서 발생한 변경이 User1과는 전혀 관계없는 변경이라면, User1을 다시 컴파일하고 새로 배포하는 상황은 초래되지 않는다. 

`인터페이스 분리 원칙`을 사용하는 근본적인 이유는 필요 이상으로 많은 걸 포함하는 모듈에 의존하는 것은 해롭기 때문이다. 소스 코드 의존성의 경우 불필요한 재컴파일과 재배포를 강제하기 때문이다. 

`S->F->D` S가 F에 의존하고, F가 D에 의존하게 된다면 S에서는 전혀 필요 없는 기능이 D에 포함될 수 있다. 이때 D내부가 변경되면 F를 재배포 해야할 수도 있으며, 더 나아가 S를 재배포 해야 할지 모른다. 

**인터페이스 분리 원칙을 정리하면 불필요한 짐을 실은 무언가에 의존하면 예상치도 못한 문제에 빠지기 때문에 해당 원칙을 지키자**


# 의존성 역전 원칙
**`의존성 역전 원칙`에서 말하는 유연성이 극대화된 시스템은 소스 코드 의존성이 추상에 의존하며 구체에는 의존하지 않는 시스템이다**

의존하지 않도록 피하고자 하는 것은 `변동선이 큰 구체적인 요소` 즉, 정의 부분이 아닌 구현 부분을 의미한다(출처: 오브젝트). 구체적인 구현체에 변경이 생기더라도 구현체가 구현하는 인터페이스는 대다수의 경우에 변경될 필요가 없다. 따라서 인터페이스는 구현체보다 변동성이 낮다. 

즉, 안정적인 소프트웨어 아키텍처란 변동성이 큰 구현체에 의존하는 일은 지양하고, 안정된 추상 인터페이스를 선호하는 아키텍처이다. 

**변동성이 큰 구체 클래스를 참조하지 말라**
> 추상 인터페이스를 참조하도록 설계해야 하며, 이 규칙은 객체 생성 방식을 강하게 제약하며, 일반적으로 `추상 팩토리`를 사용하도록 강제한다

**변동성이 큰 구체 클래스로부터 파생하지 마라**
> 정적 타임 언어에서 상속은 소스 코드에 존재하는 모든 관계 중에서 가장 강력한 동시에 유연하지 않아 변경하기 어렵다. 따라서 상속은 신중하게 사용해야 한다. 

**구체 함수를 오버라이드 하지 마라**
> 대체로 구체 함수는 소스 코드 의존성을 필요로 한다. 따라서 구체 함수를 오버라이드 하면 이러한 의존성을 제거할 수 없게 되며, 실제로는 그 의존성을 상속하게 된다. 이러한 의존성을 제거하려면, 차라리 추상 함수로 선언하고 구현체들에서 각자의 용도에 맞게 구현해야 한다.

**구체적이며 변동성이 크다면 절대로 그 이름을 언급하지 마라**
> 의존성 역전 원칙을 다른 방식으로 해석한 것

코드 상에서 DIP 위배를 모두 없앨 수는 없다. 하지만 DIP를 위배하는 클래스들은 적은 수의 구체 컴포넌트 내부로 모을 수 있고, 이를 통해 시스템의 나머지 부분과는 분리할 수 있다.

**제어흐름은 소스 코드 의존성과는 정반대 방향으로 곡선을 가로지른다. 즉, 소스 코드 의존성은 제어흐름과는 반대 방향으로 역전된다. **


출처: Clean Architecture
