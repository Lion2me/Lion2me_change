---

layout: post

title: Spring Framework

date: 2021-04-26 21:05:23 +0900

categories: Spring

---

스프링 프레임워크
-----------------

##### 1. 제어의 역전

스프링 프레임워크의 **프레임워크** 라는 단어는 사용자가 해야 할 제어를 위임한다는 의미로 사용됩니다. 객체의 생성부터 생명주기까지 이러한 부분을 특정 객체에 위임하는 것을 IOC(제어의 역전)라고 합니다.

스프링 프레임워크에서 제어의 역전이 일어나는 방식은 객체를 생성할 때 의존성을 주입해주는 방식으로 작동합니다. 원래라면 객체를 생성하고 사용하면서 새로운 객체를 동적할당(생성)하여 사용합니다. 하지만 스프링 프레임워크는 빈(Bean) 개념을 사용하여 더욱 쉽게 의존성을 주입할 수 있습니다.

기존의 구현 방식은 다음과 같습니다.

```java
public class CarService {
    public void drive(){
        CarRepository CarRepository = new Drive();
        CarRepository.driveCar();
    }
}
```

이 소스에서는 CarService는 Drive 클래스에 의존하는 상태가 됩니다. 이 경우에는 다른 함수에서 Drive 클래스로 생성한 객체를 사용할 수 없게 됩니다. 만약 사용하려면 또 다시 생성해야하는 어려움이 있습니다.

그러면 하나의 방법은 있습니다. 바로 클래스 내에서 사용 할 수 있는 객체를 생성하고 생성자에서 해당 클래스를 인자로 받아 의존성을 주입하는 방식입니다.

```java
public class CarService {
    private CarRepository carRepository;

    public CarService(CarRepository carRepository){
        this.carRepository = carRepository;
    }

    public void drive(){
        carRepository.driveCar();
    }
}
```

위의 소스를 보면 처음에 private 접근 지정자를 통해 외부에서 접근 할 수 없는 **빈 객체** 를 만들고 생성자가 실행되기 전 까지 비어있는 모습을 볼 수 있습니다. 이 후 생성자에서 인자로 받은 값을 객체에 넣어주는 방식으로 작동하는 것을 볼 수 있습니다.

여기서 스프링 프레임워크에서는 이런 의존성을 주입해주며 객체의 생성과 소멸을 자동으로 지원해주는 **빈(Bean) 컨테이터** 를 지원합니다.

위에서 본 생성자를 이용하는 경우도 의존성을 주입하는 방식이 될 수 있지만, 생성자는 여러개가 있을 수 있고 사용하기에 따라 다른 작업을 해야 할 수도 있습니다.

빈 컨테이너를 이용하면 다음과 같이 간단한 방식으로 사용할 수 있습니다.

```java
@Service
public class CarService {
    @Autowired
    private CarRepository carRepository;

    public void drive(){
        carRepository.driveCar();
    }
}
```

Service는 해당 클래스가 실제 사용되는 비즈니스 로직임을 알려주는 어노테이션입니다.

여기서 중요한 점은 Autowired 어노테이션인데, 생성자나 setter, 필드값에 사용할 수 있으며 해당하는 타입과 동일한 클래스의 bean을 찾아서 해당 객체에 넣어주게 됩니다.

이렇게 CarService는 CarRepository에 대해 의존성을 가지게 됩니다.

매번 생성되는 동적할당 방식이 아닌 싱글톤 방식으로 빈 컨테이너에 생성되기 때문에 1개의 객체만이 생성되는 것을 확신할 수 있고, 동시에 메모리 낭비도 줄어들게 됩니다. 또한 전역 인스턴스이기 때문에 다른 클래스의 인스턴스에서도 사용하기 쉽습니다.

##### 2. AOP ( 관점 지향 프로그래밍 )

AOP를 공부하면서 알게 된 개념으로는 크게

##### 참고

-	https://dpdpwl.tistory.com/140
-	https://devlog-wjdrbs96.tistory.com/166
