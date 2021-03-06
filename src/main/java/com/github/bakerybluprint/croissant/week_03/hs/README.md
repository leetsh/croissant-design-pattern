## 추상 팩토리 패턴

**정의**

한마디로 표현하자면 관련성 있는 여러 종류의 객체를 일관된 방식으로 생성하는 경우에 유용하다.

---

### 예시를 통해 살펴보자.

**엘리베이터 부품 업체 사용하기**

<img src="https://user-images.githubusercontent.com/40616436/82824171-cd754180-9ee3-11ea-8f98-63f74905064b.png" alt="image" style="zoom:50%;" />

- 추상 클래스로 Motor, Door를 정의한다.

  - 제조 업체가 다른 경우에는 **제어 방식**이 다르지만 엘리베이터 입장에서는 **모터를 구동해 엘리베이터를 이동시킨다는 면에서는 동일하다.**
  - 이로인해, 추상클래스로 Motor(모터 구동을 동일)를 정의하고 여러 제조 업체의 모터를 하위 클래스로 정의할 수 있다.

- Motor 클래스와 연관관계 : Door

  - Motor 클래스는 이동하기 전에 문을 닫아야 한다.
  - Door 객체의 getDoorStatus() 메소드를 호출하기 위해서는 연관관계가 필요하다.

---

**Motor의 move()와 Door의 open(), close()**

- Motor 클래스의 핵심 기능인 이동은 move() 메소드로 정의한다.

  ~~~java
  public void move(Direction direction) {
    // 1) 이미 이동 중이면 무시한다.
    // 2) 만약 문이 열려 있으면 문을 닫는다.
    // 3) 모터를 구동해서 이동시킨다. -> 제조 업체에 따라 다르다.
    // 4) 모터의 상태를 이동 중으로 설정한다.
  }
  ~~~

  - 1), 2), 4)단계 동작은 모두 공통(LGMotor, HyundaiMotor)이고 3)단계 부분만 달라진다.
  - **여기에서는 일반적으로는 동일하지만 특정 부분에서 다르니 템플릿 메소드 패턴으로 설게할 수 있다.**

- Door 클래스의 open(), close() 메소드 정의

  ~~~java
  public void open() {
  // 1) 이미 문이 열려 있으면 무시한다.
  // 2) 문을 연다. -> 제조 업체에 따라 다르다.
  // 3) 문의 상태를 '열림'으로 설정한다.
  }
  public void close() {
    // 1) 이미 문이 닫혀 있으면 무시한다.
    // 2) 문을 닫는다. -> 제조 업체에 따라 다르다.
    // 3) 문의 상태를 '닫힘'으로 설정한다.  
    }
  ~~~

  - 1), 3) 단계는 공통이고 2)단계만 달라진다.
  - **여기서도 마찬가지로 템플릿 메소드 패턴으로 설계할 수 있다.**

---

**Door 소스를 구현해보자**

  ~~~java
public enum DoorStatus {
  OPENED,
  CLOSED
}

public abstract class Door {
  private DoorStatus doorStatus;	//현재 문의 상태

  public Door() { 
    doorStatus = DoorStatus.CLOSED; 
  }
  public DoorStatus getDoorStatus() {
    return doorStatus; 
  }

  protected abstract void doClose();

  // 템플릿 메서드: 문을 닫는 기능 수행
  public void close() {
    if(doorStatus == DoorStatus.CLOSED)
      return;

    doClose(); // 실제로 문을 닫는 동작을 수행
    doorStatus = DoorStatus.CLOSED; // 문의 상태를 닫힘으로 기록
  }

  protected abstract void doOpen();

  // 템플릿 메서드: 문을 여는 기능 수행
  public void open() {
    if(doorStatus == DoorStatus.OPENED)
      return;

    doOpen(); // 실제로 문을 여는 동작을 수행
    doorStatus = DoorStatus.OPENED; // 문의 상태를 열림으로 기록
  }
}

public class LGDoor extends Door {
  protected void doClose() { System.out.println("close LG Door"); }
  protected void doOpen() { System.out.println("open LG Door"); }
}

public class HyundaiDoor extends Door {
  protected void doClose() { System.out.println("close Hyundai Door"); }
  protected void doOpen() { System.out.println("open Hyundai Door"); }
}
  ~~~



---

**Motor 팩토리 메소드 패턴**

~~~java
public enum VendorID { LG, HYUNDAI }

public abstract class Motor {...}
public class LGMotor extends Motor {}
public class HyundaiMotor extends Motor {}

public class MotorFactory {
  // vendorID에 따라 LGMotor 또는 HyundaiMotor 객체를 생성함
  public static Motor createMotor(VendorID vendorID) {
    Motor motor = null;

    if(vendorID == VendorID.LG) {
    	motor = new LGMotor();
    } else if(vendorID == VendorID.HYUNDAI) {
      motor = new HyundaiMotor();
    }
    
    return motor;
  }
}
~~~

**Door 팩토리 메소드 패턴**

~~~java
public class DoorFactory {
  // vendorID에 따라 LGDoor 또는 HyundaiDoor 객체를 생성함
  public static Door createDoor(VendorID vendorID) {
    Door door = null;

    if(vendorID == VendorID.LG) {
    	motor = new LGMotor();
    } else if(vendorID == VendorID.HYUNDAI) {
      motor = new HyundaiMotor();
    }
    
    return door;
  }
}
~~~

---

그럼 이제 마지막으로 Main에서 해당 메소드들을 호출하여 Motor와 Door를 만들 수 있는 것이다.

~~~java
//Main
public class Client {
  public static void main(String[] args) {
    Door lgDoor = DoorFactory.createDoor(VendorID.LG); // 팩토리 메서드 호출
    Motor lgMotor = MotorFactory.createMotor(VendorID.LG); // 팩토리 메서드 호출
    lgMotor.setDoor(lgDoor);	//문을 열고 닫기 위한 모터 연결

    lgDoor.open();	//LGDoor 오픈
    lgMotor.move(Direction.UP);	//LGMotor를 이용하여 엘리베이터 이동
  }
}
~~~

여기까지가 LG 및 Hyundai 브랜드를 이용한 Door와 Motor를 팩토리 메소드 패턴으로 만들어보았다. 

---

### **팩토리 메소드 패턴 문제점**

LG의 부품 대신 현대의 부품(HyundaiMotor, HyundaiDoor 클래스)을 사용한다면?

~~~java
public class Client {
  public static void main(String[] args) {
 Door hyundaiDoor = DoorFactory.createDoor(VendorID.HYUNDAI); // 팩토리 메서드 호출
 Motor hyundaiMotor = MotorFactory.createMotor(VendorID.HYUNDAI); // 팩토리 메서드 호출
 hyundaiMotor.setDoor(lgDoor);
 // 문을 먼저 연다.
 hyundaiDoor.open();
 hyundaiMotor.move(Direction.UP);
  }
}
~~~

또한, 만약에 LG나 Hyundai 말고 Samsung 부품이 추가된다면..?

~~~java
public class DoorFactory {
  // vendorID에 따라 LGDoor 또는 HyundaiDoor 객체를 생성함
  public static Door createDoor(VendorID vendorID) {
    Door door = null;
    // 삼성의 부품을 추가
    if(vendorID == VendorID.LG) {
      motor = new LGMotor();
    } else if(vendorID == VendorID.HYUNDAI) {
      motor = new HyundaiMotor();
    } else if(vendorID == VendorID.SAUMSUNG) {
      motor = new Samsung();
    }
    return door;
  }
}
~~~



---

### 추상 팩토리 패턴

관련 객체들을 일관성 있게 생성하는 Factory를 만들어 사용하자.

<img src="https://user-images.githubusercontent.com/40616436/82827708-850d5200-9eea-11ea-8b3a-a69792a10d1a.png" alt="image" style="zoom:50%;" />

**업체별로 Factory 클래스를 만들자**

Ex. LGElevatorFactory : LGMotor, LGDoor | HyundaiElevatorFactory : HyundaiMotor, HyundaiDoor

---

**소스를 통해 확인해보자.**

~~~java
public abstract class ElevatorFactory {
  public abstract Motor createMotor();
  public abstract Door createDoor();
}

/* LG 부품을 생성하는 팩토리 클래스 */
public class LGElevatorFactory extends ElevatorFactory {
  public Motor createMotor() { return new LGMotor(); }
  public Door createDoor() { return new LGDoor(); }
}
/* Hyundai 부품을 생성하는 팩토리 클래스 */
public class HyundaiElevatorFactory extends ElevatorFactory {
  public Motor createMotor() { return new HyundaiMotor(); }
  public Door createDoor() { return new HyundaiDoor(); }
}

/* 주어진 업체의 이름에 따라 부품을 생성하는 Client 클래스 */
public class Client {
  public static void main(String[] args) {
    ElevatorFactory factory = null;
    String vendorName = args[0];

    // 인자에 따라 LG 또는 Hyundai 팩토리를 생성
    if(vendorName.equalsIgnoreCase("LG"))	//대소문자 노구분
      factory = new LGElevatorFactory();
    else
      factory = new HyundaiElevatorFactory();

    Door door = factory.createDoor(); // 해당 업체의 Door 생성
    Motor motor = factory.createMotor(); // 해당 업체의 Motor 생성
    motor.setDoor(door);

    door.open();
    motor.move(Direction.UP);
  }
}
~~~

---

**팩토리 메소드 패턴 문제 해결**

첫번째 문제점 : 다른 제조업체의 부품을 변경하려고 하면 Client 코드의 변경이 많아짐

두번째 문제점 : 제조 업체 추가 시 각 부품 팩토리에 추가 및 Client 코드의 변경이 발생

**Saumsung이 추가 된다면?**

~~~java
/* Samsung 부품을 생성하는 팩토리 클래스 */
public class SamsungElevatorFactory extends ElevatorFactory {
  public Motor createMotor() { return new SamsungMotor(); }
  public Door createDoor() { return new SamsungDoor(); }
}
/* Samsung Door 클래스 */
public class SamsungDoor extends Door {
  protected void doClose() { System.out.println("close Samsung Door"); }
  protected void doOpen() { System.out.println("open Samsung Door"); }
}
/* Samsung Motor 클래스 */
public class SamsungDoor extends Door {
  protected void doClose() { System.out.println("close Samsung Motor"); }
  protected void doOpen() { System.out.println("open Samsung Motor"); }
}

/* 주어진 업체의 이름에 따라 부품을 생성하는 Client 클래스 */
public class Client {
  public static void main(String[] args) {
    ElevatorFactory factory = null;
    String vendorName = args[0];

    // 인자에 따라 LG 또는 Samsung 또는 Hyundai 팩토리를 생성
    if(vendorName.equalsIgnoreCase("LG"))
      factory = new LGElevatorFactory();
    else if(vendorName.equalsIgnoreCase("Samsung")) // Samsung만 추가
      factory = new SamsungElevatorFactory();
    else
      factory = new HyundaiElevatorFactory();

    ...
  }
}
~~~

---

**진정한 추상 팩토리 패턴**

- ElevatorFactoryFactory 클래스 : VendorID에 따라 제조 업체의 Factory 생성
- ElevatorFactoryFactory 클래스의 getFactory() : 팩토리 메소드

~~~java
/* 팩토리 클래스에 팩토리 메서드 패턴을 적용 */
public class ElevatorFactoryFactory {
  public static ElevatorFactory getFactory(VendorID vendorID) {
    ElevatorFactory factory = null;

    if(vendorID == VendorID.LG) {
      factory = new LGElevatorFactory.getInstance();
    } else if(vendorID == VendorID.HYUNDAI) {
      factory = new HyundaiElevatorFactory.getInstance();
    } else if(vendorID == VendorID.SAMSUNG) {
      factory = new SamsungElevatorFactory.getInstance();
    }
    
    return factory;
  }
}

~~~

---

제조 업체별 Factory 객체는 각 1개만 있으면 되므로 이때 싱글톤을 사용하여 설계가 가능하다.

~~~java
/* 싱글턴 패턴을 적용한 LG 팩토리 */
public class LGElevatorFactory extends ElevatorFactory {
  public static ElevatorFactory getFectory(VendorID vendorID) {
    private static ElevatorFactory factory;
    private LGElevatorFactory() { } // 생성자를 private로

    public static ElevatorFactory getInstance() {
      return LazyHolder.INSTANCE;
    }
    
    private static class LazyHolder {
        private static final ElevatorFactory INSTANCE = new LGElevatorFactory();
    }

    public Motor createMotor() { return new LGMotor(); }
    public Door createDoor() { return new LGDoor(); }
  }
}
/* 싱글턴 패턴을 적용한 Hyundai 팩토리 */
//동일
/* 싱글턴 패턴을 적용한 Samsung 팩토리 */
//동일
~~~

---

이제 메인에서는 Factory를 바로 생성하지 않고 ElavatorFactoryFactory.getFactory를 통해 Client가 원하는 엘리베이터 제품을 만들 수 있게 된다.

~~~java
//Main
public class Client {
  public static void main(String[] args) {
    String vendorName = args[0];
    VendorID vendorID;

    // 인자에 따라 LG 또는 Samsung 또는 Hyundai 팩토리를 생성
    if(vendorName.equalsIgnoreCase("LG"))
      vendorID = VendorID.LG;
    else if(vendorName.equalsIgnoreCase("Samsung"))
      vendorID = VendorID.SAMSUNG;
    else
      vendorID = VendorID.HYUNDAI;

    ElevatorFactory factory = ElevatorFactoryFactory.getFactory(vendorID);

    ...
  }
}
~~~

<img src="https://user-images.githubusercontent.com/40616436/82829838-331afb00-9eef-11ea-9b8f-e604fda16c25.png" alt="image" style="zoom:50%;" />

https://gmlwjd9405.github.io/2018/08/08/abstract-factory-pattern.html



---

## 브릿지 패턴

브릿지 패턴을 간단하게 풀어서 말하면, 기능 클래스의 계층과 구현 클래스의 계층을 구분하고 연결할 때 마치 다리로 연결 시켜 놓는 것 처럼 보여 브릿지 패턴이라고 한다.

---

**그럼 이제 설계를 통해 살펴보자.**

<img src="https://user-images.githubusercontent.com/40616436/82153283-75b25700-98a1-11ea-89b2-cbcce973dffc.png" alt="image" style="zoom:50%;" />

우리의 실습 예제를 살펴보면,

<img src="https://user-images.githubusercontent.com/40616436/82831877-8b083080-9ef4-11ea-852c-95b67fa982f1.png" alt="image" style="zoom:50%;" />

위와 같은 구조가 될 것이다.

---

**먼저 기능의 클래스 계층을 소스로 살펴보자.**

~~~java
public class MorseCode {

  private MorseCodeFunction function;

  public MorseCode(MorseCodeFunction function) {
    this.function = function;
  }

  public void dot() {
    function.dot();
  }

  public void dash() {
    function.dash();
  }

  public void space() {
    function.space();
  }
}

public class PrintMorseCode extends MorseCode {

  public PrintMorseCode(MorseCodeFunction function) {
    super(function);
  }

  //blue 모스부호 기능
  public PrintMorseCode b() {
    dash();dot();dot();dot();space();
    return this;
  }

  public PrintMorseCode l() {
    dot();dash();dot();dot();space();
    return this;
  }

  public PrintMorseCode u() {
    dot();dot();dash();space();
    return this;
  }

  public PrintMorseCode e() {
    dot();space();
    return this;
  }
}

~~~

blue라는 모스부호의 기능을 추가한 것이다.

---

**구현 클래스의 계층을 살펴보자.**

~~~java
public interface MorseCodeFunction {
  public void dot();
  public void dash();
  public void space();
}

public class DefaultMCF implements MorseCodeFunction{

  @Override
  public void dot() {
    System.out.print("·");
  }

  @Override
  public void dash() {
    System.out.print("-");
  }

  @Override
  public void space() {
    System.out.print(" ");
  }
}
~~~

구현 단계에서 DefaultMCF 클래스는 MorseCodeFunction의 하위 클래스로서, dot, dash, space의 구체적인 부분을 구현하고 있다.

---

**Main을 통해 브릿지 패턴을 사용**

~~~java
public class BridgeMain {
  public static void main(String[] args) {
    PrintMorseCode printMorseCode = new PrintMorseCode(new DefaultMCF());
    printMorseCode.b().l().u().e();
  }
}
//실행 결과 : -··· ·-·· ··- · 
~~~



---

음성 모스 부호 추가 될 시 

~~~java
public class SoundMCF implements MorseCodeFunction{
  @Override
  public void dot() {
    System.out.print("삣");
  }

  @Override
  public void dash() {
    System.out.print("삐~");
  }

  @Override
  public void space() {
    System.out.print(" ");
  }
}

public class BridgeMain {
  public static void main(String[] args) {
    PrintMorseCode printMorseCode = new PrintMorseCode(new FlashMCF());
    printMorseCode.b().l().u().e();
  }
}
//실행 결과
//-*-### #-*-## ##-*- #
~~~

---

**어댑터 패턴 vs 브릿지 패턴**

두 패턴의 가장 큰 차이는 목적이 무엇인가 이다.

- Adapter의 목적 : 두 인터페이스의 불일치 해결
- Bridge의 목적 : 추상화 개념과 구현을 따로 만들고 이들을 연결시켜 주려고 하는 것

---

**어댑터 패턴 vs 전략 패턴**

두 패턴의 가장 큰 차이는 의도가 다른 것이다.

- Bridge : 구현과 기능을 분리하여 계층간의 결합도를 낮춤.
- Strategy : 결합도는 신경쓰지 않고 변경 및 추가되는 부분을 추상화.

---

**브릿지 패턴을 왜 사용하는가?**

Before

<img src="https://user-images.githubusercontent.com/40616436/82916355-e85fb880-9fac-11ea-8d61-0367258ed752.png" alt="image" style="zoom:50%;" />

**변경에 대한 이점**

<img src="https://user-images.githubusercontent.com/40616436/82916512-180ec080-9fad-11ea-99e3-f7b64c4c83cb.png" alt="image" style="zoom:50%;" />

**보안에 대한 이점**

<img src="https://user-images.githubusercontent.com/40616436/82916624-3bd20680-9fad-11ea-8a09-803b172f29ef.png" alt="image" style="zoom:50%;" />

---

before

<img src="https://user-images.githubusercontent.com/40616436/82917112-ec400a80-9fad-11ea-90d8-a8ac2e0469ed.png" alt="image" style="zoom:50%;" />

**추가에 대한 이점**

<img src="https://user-images.githubusercontent.com/40616436/82917363-4ccf4780-9fae-11ea-8a4b-995c1fb9005b.png" alt="image" style="zoom:50%;" />