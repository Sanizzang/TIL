### Builder Pattern?

- 복잡한 구성의 객체를 효과적으로 생성하는 패턴
- 알려진 Builder 패턴은 2가지임
  - 생성시 지정해야 할 인자가 많을 때 사용하는 패턴
  - 객체 생성 시 여러 단계를 순차적으로 거칠 때, 이 단계의 순서를 결정해 두고
    각 단계를 다양하게 구현할 수 있도록 하는 패턴

#### 생성시 지정해야 할 인자가 많을 때 사용하는 패턴

![](https://velog.velcdn.com/images/sanizzang00/post/a5d27a80-fcb9-424a-809e-75ef6ca21ed3/image.png)

Car: 실제로 생성하고자 하는 클래스
CarBuilder: Car 객체를 생성해주는 Builder 클래스

Car 클래스를 생성할 때 생성자의 인자에 칼을 구성하는 스펙 항목들을 지정받게 된다.

Car.java

```java
public class Car {
    private String engine; // 엔진
    private boolean airbag; // 에어백 여부
    private String color; // 차량 색상
    private boolean cameraSensor; // 카메라센서 유무
    private boolean AEB; // 자동급제동장치 유무

    public Car(String enging, boolean airbag, String color, boolean cameraSensor, boolean AEB) {
        this.engine = engine;
        this.airbag = airbag;
        this.color = color;
        this.cameraSensor = cameraSensor;
        this.AEB = AEB;
    }
}
```

CarBuilder.java

```java
public class CarBuilder {
    private String engine; // 엔진
    private boolean airbag; // 에어백 여부
    private String color; // 차량 색상
    private boolean cameraSensor; // 카메라센서 유무
    private boolean AEB; // 자동급제동장치 유무

    public CarBuilder setEngine(String engine) {
        this.engine = engine;
        // this를 반환하는 이유는 Method Chain을 지원하기 위함이다.
        return this;
    }

    public CarBuilder setAirbag(boolean airbag) {
        this.airbag = airbag;
        return this;
    }

    public CarBuilder setColor(String color) {
        this.color = color;
        return this;
    }

    public CarBuilder setCameraSensor(boolean cameraSensor) {
        this.cameraSensor = cameraSensor;
        return this;
    }

    public CarBuilder setAEB(boolean AEB) {
        this.AEB = AEB;
        return this;
    }

    public Car build() {
        return new Car(engin, airbag, airbag, color, cameraSensor, AEB);
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        Car car1 = new Car("V7", true, "Black", true, false);

        // Method Chainning
        CarBuilder car2 = new CarBuilder()
            .setAEB(false)
            //.setAirbag(false)
            .setCameraSensor(true)
            .setColor("White")
            .setEngine("V9");
            //.build();

        Random random = new Random();
        Car car2 = builder
            .setAribag(random.nextInt(2) == 0) // 50%의 확률로 에어백 장착 결정
            .build();
    }
}
```

#### 객체 생성 시 여러 단계를 순차적으로 거칠 때, 이 단계의 순서를 결정해 두고 각단계를 다양하게 구현할 수 있도록하는 패턴

![](https://velog.velcdn.com/images/sanizzang00/post/8a677ccd-e82c-43fc-8e0f-c88cbce364e7/image.png)

Director: 빌더에서 제공하는 메소드를 정해진 순서대로 정확하게 호출해야 할 책임을 갖는다.

Data.java

```java
public class Data {
    private String name;
    private int age;

    public Data(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```

Builder.java

```java
public abstract class Builder {
    protected Data data;

    public Builder(Data data) {
        this.data = data;
    }

    // 어떤 순서로 또 어떤 식으로 이 메소드들을 사용하는지는 Director 클래스가 결정한다.
    public abstract String head();
    public abstract String body();
    public abstract String foot();
}
```

Director.java

```java
public class Director{
    private Builder builder;

    public Director(Builder builder) {
        this.builder = builder;
    }

    public String build() {
        StringBuilder sb = new StringBuilder();

        sb.append(builder.head());
        sb.append(builder.body());
        sb.append(builder.foot());

        return sb.toString();
    }
}
```

PlainTextBuilder

```java
public class PlainTextBuilder extends Builder {
    public PlainTextBuiler(Data data) {
        super(data);
    }

    @Override
    public String head() {
        return "";
    }

    @Override
    public String body() {
        StringBUilder sb = new StringBuilder();

        sb.append("Name: ");
        sb.append(data.getName());
        sb.append(", Age: ");
        sb.append(data.getAge());

        return sb.toString();
    }

    @Override
    public String foot() {
        return "";
    }
}
```

JSONTextBuilder

```java
public class JSONTextBuilder extends Builder {
    public JSONTextBuilder(Data data) {
        super(data);
    }

    @Override
    public String head() {
        return "{";
    }

    @Override
    public String body() {
        StringBUilder sb = new StringBuilder();

        sb.append("\"Name\": ");
        sb.append("\"" + data.getName() + "\"");
        sb.append(", \"Age\": ");
        sb.append(data.getAge());

        return sb.toString();
    }

    @Override
    public String foot() {
        return "}";
    }
}
```

XMLBuilder

```java
public class XMLBuilder extends Builder {
    public XMLBuilder(Data data) {
        super(data);
    }

    @Override
    public String head() {
        StringBuilder sb = new StringBuilder();

        sb.append("<?xml version=\"1.0\" encoding=\"utf-8\"?>");
        sb.append("<DATA>");

        return sb.toString();
    }

    @Override
    public String body() {
        StringBUilder sb = new StringBuilder();

        sb.append("<Name>");
        sb.append(data.getName());
        sb.append("</Name>");
        sb.append("<Age>");
        sb.append(data.getAge());
        sb.append("</Age>");

        return sb.toString();
    }

    @Override
    public String foot() {
        return "</DATA>";
    }
}
```

MainEntry.java

```java
public class MainEntry{
    public static void main(String[] args) {
        Data data = new Data("Jane", 25);

        Builder builder = new PlainTextBuilder(data);
        Director director = new Director(builder);
        String result = director.build();
        System.out.println(result);

        builder = new JSONBuilder(data);
        Director director = new Director(builder);
        String result = director.build();
        System.out.println(result);

        builder = new XMLBuilder(data);
        Director director = new Director(builder);
        String result = director.build();
        System.out.println(result);
    }
}
```
