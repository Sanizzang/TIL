### Command Pattern?

- 하나의 명령(기능)을 객체화한 패턴
- 객체는 전달할 수 있고 보관할 수 있음, 즉 명령(기능)을 전달하고 보관할 수 있게 됨
- 배치 실행, Undo/Redo, 우선순위가 높음 명령을 먼저 실행하기 등이 가능해 짐

![](https://velog.velcdn.com/images/sanizzang00/post/8fbce2c0-bdf7-4fad-9a79-baab0613265d/image.png)

Command.java

```java
public interface Command {
    public void run();
}
```

ClearCommand.java

```java
public class ClearCommand implements Command {

    @Override
    public void run() {
        System.out.print("\033[H\033[2J");
        System.out.flush();
    }
}
```

PrintCommand.java

```java
public class PrintCommand implements Command {
    private String content;

    public PrintCommand(String content) {
        this.content = content;
    }

    @Override
    public void run() {
        System.out.print(content);
    }
}
```

MoveCommand.java

```java
public class MoveCommand implements Command {
    private int x;
    private int y;

    public MoveCommand(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public void run() {
        System.out.print(String.format("%c[%d;%df", 0x1B, y,x));
    }
}
```

ColorCommand.java

```java
public class ColorCommand implements Command {
    public enum Color {
        BLACK("u001B[30m"), RED("u001B[31m"),
        GREEN("u001B[32m"), YELLOW("u001B[33m"),
        BLUE("u001B[34m"), PURPLE("u001B[35m"),
        CYAN("u001B[36m"), WHITE("u001B[37m");

        // 제어문자를 저장할 필드
        final private String code;

        public Color(String code) {
            this.code = code;
        }

        public String getCode() {
            return code;
        }
    };

    private Color color;

    public ColorCommand(Color color) {
        this.color = color;
    }

    @Override
    public void run() {
        System.out.print(color.getCode());
    }
}
```

CommandGroup.java

```java
// CommandGroup은 자신 이외에 어떤 종류의 명령이 있는지 조차 모른다.
public class CommandGroup implements Command {
    private ArrayList<Command> commands = new ArrayList<Command>();

    public void add(Command command) {
        commands.add(command);
    }

    // 필드에 저장하고 있는 Command 객체들을 하나씩 실행하는 코드
    @Override
    public void run() {
        int cntCommands = commands.size();
        for(int = 0;i < cntCommands;i++) {
            Command command = commands.get(i);
            command.run();
        }
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        CommandGroup cmdGroup = new CommandGroup();

        Command clearCmd = new ClearCommand();
        cmdGroup.add(clearCmd);

        Command yellowColorCmd = new ColorCommand(ColorCommand.Color.YELLOW);
        cmdGroup.add(yellowColorCmd);

        Command moveCmd = new MoveCommand(10, 1);
        cmdGroup.add(moveCmd);

        Command printCmd = new PrintCommand("안녕하세요!");
        cmdGroup.add(printCmd);

        Command moveCmd2 = new MoveCommand(15, 5);
        cmdGroup.add(moveCmd2);

        cmdGroup.add(printCmd);

        cmdGroup.run();
    }
}
```
