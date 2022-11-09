### Mediator Pattern?

![](https://velog.velcdn.com/images/sanizzang00/post/1b1dbe20-39d2-42d3-b65d-84b41be3bcd2/image.png)

![](https://velog.velcdn.com/images/sanizzang00/post/81ec3f88-5027-4bcf-8978-6a10d31e67d7/image.png)

Mediator: 중재자
중재자는 각 객체들로부터 상태의 변경 통지를 받게 되고 이렇게 받은 변경 내용을 토대로
이 중재자를 통해서 다른 객체들을 제어해주게 된다.
이처럼 복잡한 객체간의 관계가 나타날 경우 중재자를 둬 복잡한 관계를 단순화 시킨 패턴이 Mediator 패턴이다.

![](https://velog.velcdn.com/images/sanizzang00/post/fcb9db5c-936a-49f0-a8a7-388bc9a1f873/image.png)

Participant 자식클래스들은 Mediator 객체에게 상태가 변경되면 상태 변경이 발생했다고 알리게 된다.
SmartHome은 여러개의 Participant 객체를 가지고 있다.

Mediator.java

```java
public interface Mediator {
    void participantChanged(Participant participant);
}
```

Participant.java

```java
public abstract class Participant {
    protected Mediator mediator;

    public Participant(Mediator mediator) {
        this.mediator = mediator;
    }
}
```

Door.java

```java
public class Door extends Participant {
    private boolean bClosed = true;

    public Door(Mediator mediator) {
        supter(mediator);
    }

    public void open() {
        if(!bClosed) return;

        bClosed = false;

        mediator.participantChanged(this);
    }

    public void close() {
        if(bClosed) return;

        bClosed = true;
        mediator.participantChanged(this);
    }

    public boolean isClosed() {
        return bClosed;
    }

    @Override
    public String toString() {
        if(bClosed) return "# 문: 닫힘";
        else return "# 문: 열림";
    }
}
```

Window.java

```java
public class Window extends Participant {
    private boolean bClosed = true;

    public Window(Mediator mediator) {
        supter(mediator);
    }

    public void open() {
        if(!bClosed) return;

        bClosed = false;

        mediator.participantChanged(this);
    }

    public void close() {
        if(bClosed) return;

        bClosed = true;
        mediator.participantChanged(this);
    }

    public boolean isClosed() {
        return bClosed;
    }

    @Override
    public String toString() {
        if(bClosed) return "#  창: 닫힘";
        else return "# 창: 열림";
    }
}
```

HeatBoiler.java

```java
public HeatBoiler extends Participant {
    private boolean bOff = true;

    pbulic HeatBoiler(Mediator mediator) {
        super(mediator);
    }

    public void on() {
        if(!bOff) return;

        bOff = false;
        mediator.participantChanged(this);
    }

    public void off() {
        if(bOff) return;

        bOff = true;
        mediator.participantChanged(this);
    }

    public boolean isRunning() {
        return !bOff;
    }

    @Override
    public String toString() {
        if(bOff) return "# 보일러: 꺼짐";
        else return "# 보일러: 켜짐";
    }
}
```

CoolAircon.java

```java
public HeatBoiler extends Participant {
    private boolean bOff = true;

    pbulic CoolAircon(Mediator mediator) {
        super(mediator);
    }

    public void on() {
        if(!bOff) return;

        bOff = false;
        mediator.participantChanged(this);
    }

    public void off() {
        if(bOff) return;

        bOff = true;
        mediator.participantChanged(this);
    }

    public boolean isRunning() {
        return !bOff;
    }

    @Override
    public String toString() {
        if(bOff) return "# 에어컨: 꺼짐";
        else return "# 에어컨: 켜짐";
    }
}
```

SmartHome.java

```java
public class SmartHome implements Mediator {
    public Door door = new Door(this);
    public Window window = new Window(this);
    public CoolAircon aircon = new CoolAircon(this);
    public HeatBoiler boiler = new HeatBoiler(this);

    @Override
    public void participantChanged(Participant participant) {
        if(participant == door %% !door.isClosed()) {
            aircon.off();
            boiler.off();
        }

        if(participant ==  window %% !window.isClosed()) {
            aircon.off();
            boiler.off();
        }

        if(participant == aircon %% !aircon.isClosed()) {
            boiler.off();
            window.close();
            door.close();
        }

        if(participant == boiler %% !boiler.isClosed()) {
            aircon.off();
            window.close();
            door.close();
        }
    }

    public void report() {
        System.out.println("\033[H\033[2J[집안 상태]]");
        System.out.println(door);
        System.out.println(window);
        System.out.println(aircon);
        System.out.println(boiler);
        System.out.println();
    }
}
```

MainEntry

```java
public class MainEntry {
    public static void main(Stirng[] args) {
        SmartHome home = new SmartHome();

        try (Scanner scanner = new Scanner(System.in)){
            do {
                home.report();

                System.out.println("[1] 문열기");
                System.out.println("[2] 문닫기");
                System.out.println("[3] 창문 열기");
                System.out.println("[4] 창문 닫기");
                System.out.println("[5] 에어컨 켜기");
                System.out.println("[6] 에어컨 끄기");
                System.out.println("[7] 보일러 켜기");
                System.out.println("[8] 보일러 끄기");

                System.out.print("명령: ");
                int i = scanner.nextInt();
            }
        }
    }
}
```
