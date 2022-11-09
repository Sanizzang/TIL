### Chain of reponsibility Pattern?

- 책임이란 무언가를 처리하는 기능(클래스)
- 여러 개의 책임들을 동적으로 연결해서 순차적으로 실행하는 패턴
- 기능을 클래스 별로 분리하여 구현하도록 유도하므로 클래스의 코드가 최적화됨

![](https://velog.velcdn.com/images/sanizzang00/post/9499cd6e-c916-44b7-bf63-2120ac0722c3/image.png)

![](https://velog.velcdn.com/images/sanizzang00/post/093ce378-476e-4860-8170-0b72c1862ed5/image.png)

Handler.java

```java
public abstract class Handler {
    protected Handler nextHandler = null;

    public Handler setNext(Handler handler) {
        this.nextHandler = handler;
        return handler;
    }

    protected abstract void process(String url);

    public void run(String url) {
        process(url);
        if(nextHandler != null) nextHandler.run(url);
    }
}
```

ProtocolHandler.java

```java
public class ProtocolHandler extends Handler {

    @Override
    protected void process(String url) {
        int index = url.indexOf("://");
        if(index != -1) {
            System.out.println("PROTOCOL: " + url.substring(0, index));
        } else {
            System.out.println("NO PROTOCOL");
        }
    }
}
```

DomainHandler.java

```java
public class DomainHandler extends Handler {
    @Override
    protected void process(String url) {
        int startIndex = url.indexOf("://");
        int lastIndex = url.lastIndexOf(":");

        System.out.print("DOMAIN: ");
        if(startIndex == -1) {
            if(lastIndex == -1) {
                System.out.println(url);
            } else {
                System.out.println(url.substring(0, lastIndex));
            }
        } else if(startIndex != lastIndex) {
            System.out.println(url.substring(startIndex + 3, lastIndex));
        } else if(startIndex == lastIndex) {
            System.out.println(url.substring(startIndex + 3));
        } else{
            System.out.println("NONE");
        }
    }
}
```

PortHandler

```java
public class PortHandler extends Handler {

    @Override
    protected void process(String url) {
        int index = url.lastIndexOf(":");
        if(index != -1) {
            String strPort = url.substring(index+1);
            try {
                int port = Integer.parseInt(strPort);
                System.out.println("PORT: " + port);
                return;
            } catch(NumberFormatException e) {
                e.printStackTrace();
            }
        }

        System.out.println("NO PORT");
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        Handler handler1 = new ProtocolHandler();
        Handler handler2 = new PortHandler();
        Handler handler3 = new DomainHandler();

        handler1.setNext(handler2).setNext(handler3);

        String url = "http://www.youtube.com:1007";
        System.out.println("INPUT: " + url);

        handler1.run(url);
    }
}
```
