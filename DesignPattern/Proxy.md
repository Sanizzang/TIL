### Proxy Pattern?

- Proxy의 뜻은 "대리인"
- 어떤 작업의 실행을 대리인을 통해 실행하도록 하는 패턴

![](https://velog.velcdn.com/images/sanizzang00/post/a947749d-b191-48f1-8e2b-5683d7c061a4/image.png)
ScreenDisplay: 어떤 문자열을 화면에 출력해주는 역할
BufferDisplay: Proxy 즉 대리인에 해당 어떤 문자열을 출력할때 ScreenDisplay를 바로 사용해도 되지만 BufferDisplay 객체를 대신 사용해도 된다.

Display.java

```java
public interface Display {
    void print(String content);
}
```

ScreenDisplay.java

```java
public class ScreenDisplay implements Display{

    // content 인자로 받은 문자열을 화면에 출력하기 위해서는 준비작업이 필요하고 상당한 시간이 소요된다고 가정
    @Override
    public void print(String content) {
        try {
            Thread.sleep(500); // 출력을 위한 준비 작업
        } catch(InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(content);
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        Display display = new ScreenDisplay();

        display.print("1231231231231232");
        display.print("1231231231231232");
        display.print("1231231231231232");
        display.print("1231231231231232");
        display.print("1231231231231232");
        display.print("1231231231231232");
        // 위의 코드는 content를 출력하기 위한 작업이 오래걸려 시간이 오래걸린다.
        // -> 출력할 content를 한번에 모아 출력하는 것이 좋다. (BufferDisplay 사용)


    }
}
```

BufferDisplay

```java
public class BufferDisplay implements Display {
    private ArrayList<String> buffer = new ArrayList<String>();
    private ScreenDisplay screen;
    private int bufferSize;

    public BufferDisplay(int bufferSize) {
        this.bufferSize = bufferSize;
    }

    @Override
    public void print(String content) {
        buffer.add(content);
        if(buffer.size() == bufferSize) {
            flush();
        }
    }

    public void flush() {
        if(screen == null) screen = new ScreenDisplay();
        String lines = String.join("\n", buffer);
        screen.print(lines);
        buffer.clear();
    }
}
```
