![](https://velog.velcdn.com/images/sanizzang00/post/c8c60226-30ce-4138-b2d0-73bbfd0d5015/image.png)

### Adapter 패턴?

어떤 클래스를 사용해야 하는데 이 클래스에 대한 코드를 변경할 수 없는 상황에서도
Adapter 패턴을 적용해서 사용할 수 있도록 해준다.

![](https://velog.velcdn.com/images/sanizzang00/post/6f6dbb65-f3a7-47fa-92f0-437d5c556e13/image.png)

Tiger 클래스를 변경할 수 없는 상황이라고 가정
Tiger 클래스를 변경하지 않아도 TigerAdapter 클래스를 통해서 Animal 클래스 처럼 사용할 수 있게 된다.

Animal.java

```java
public abstract class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public abstract void sound();
}
```

Dog.java

```java
public class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }

    @Override
    public void sound() {
        System.out.println(name + " Barking");
    }
}

```

Cat.java

```java
public class Cat extends Animal {
    public Cat(String name) {
        super(name);
    }

    @Override
    public void sound() {
        System.out.println(name + " Meow");
    }
}
```

User.java

```java
import java.util.ArrayList;

public class User {
    public static void main(String[] args) {
        ArrayList<Animal> animals = new ArrayList<Animal>();
        animals.add(new Dog("댕이"));
        animals.add(new Cat("솜털이"));
        animals.add(new Cat("츄츄"));
        //animals.add(new Tiger("타이온"));
        animals.add(new TigerAdapter("타이온"));

        animals.forEach(animal -> {
            animal.sound();
        });
    }
}
```

Tiger.java

```java
public class Tiger {
    private String name;

    void setName(String name) {
        this.name = name;
    }

    String getName() {
        return name;
    }

    void roar() {
        System.out.println("growl");
    }
}
```

TigerAdapter.java

```java
public class TigerAdapter extends Animal {
    private Tiger tiger;

    public TigerAdapter(String name) {
        super(name);

        tiger = new Tiger();
        tiger.setName(name);
    }

    @Override
    public void sound() {
        System.out.print(tiger.getName() + " ");
        tiger.roar();
    }
}
```
