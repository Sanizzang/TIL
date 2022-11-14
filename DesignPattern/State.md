### State Pattern?

- 상태를 객체화 한 패턴
- 조건절(if)을 효과적으로 줄여주는 패턴

![](https://velog.velcdn.com/images/sanizzang00/post/22903a87-93da-46ba-9de5-04c8caef0f4c/image.png)

Player.java

```java
public class Player {
    private int speed;

    public void setSpeed(int speed) {
        this.speed = speed;
    }

    public int getSpeed() {
        return speed;
    }

    private State state = new StandUpState(this);

    public void setState(State state) {
        this.state =  state;
    }

    public State getState() {
        return state;
    }

    public void talk(String msg) {
        System.out.println("플레이어의 말: \"" + msg + "\"");
    }
}
```

State.java

```java
public abstract class State {
    protected Player player;

    public State(Player player) {
        this.player = player;
    }

    public abstract void standUp();
    public abstract void sitDown();
    public abstract void walk();
    public abstract void run();
    public abstract String getDescription();
}
```

StandUpState.java

```java
public class StandUpState extends State {

    public StandUpState(Player player) {
        super(player);
    }

    @Override
    public void standUp() {
        player.talk("언제 움직일꺼야?");
    }

    @Override
    public void sitDown() {
        player.setState(new SitDownState(player));
        player.talk("앉으니깐 편하고 좋습니다.");
    }

    @Override
    public void walk() {
        player.setSpeed(5);
        player.setState(new WalkState(player));
        player.talk("걷기는 제2의 생각하기다..");
    }

    @Override
    public void run() {
        player.setSpeed(10);
        player.setState(new RunState(player));
        player.talk("갑자기 뛴다!");
    }

    @Override
    public String getDescription() {
        return "제자리에 서 있음";
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        Player player = new Player();
        Scanner scanner = new Scanner(System.in);

        while(true) {
            System.out.println("플레이어의 상태: " + player.getState().getDescription() + " / 속도:" + player.getSpeed() + "km/h");

            System.out.print("[1]제자리 서기 [2]앉기 [3]걷기 [4]뛰기 [5]종료: ");

            String input = scanner.next();
            System.out.prinln(;

            if(input.equals("1"))) player.getState().standUp();
            else if(input.equals("2")) player.getState().sitDown();
            else if(input.equals("3")) player.getState().walk();
            else if(input.equals("4")) player.getState().run();
            else if(input.equals("5")) break;
        }

    }
}
```
