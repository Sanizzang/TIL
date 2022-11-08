### Singleton 패턴이란?

하나의 클래스 타입에 대해서 오직 하나의 객체만이 생성되도록 보장해주는 패턴
![](https://velog.velcdn.com/images/sanizzang00/post/be8bccb6-1d08-4151-bac8-b989d002bb46/image.png)

![](https://velog.velcdn.com/images/sanizzang00/post/cae42ca7-c184-46b3-a405-a679db22b9b4/image.png)

King.java

```java
public class king {
    // 생성자를 private으로 설정하여 나 자신 외에 다른 곳에서 생성자를 접근하지 못하도록 함
    private King() {

    }

    // 클래스로 접근하기 위해서 static으로 선언
    private static King self = null;

    // 멀티쓰레드에서 이 메소드를 호출할 때 문제가 없도록 synchronized(동기화) 선언
    // 싱글쓰레드에서는 동기화가 필요 없음
    public synchronized static King getInstance() {
        if (self == null) {
            self = new King();
        }
        return self;
    }

    public void say() {
        System.out.println("I am only one..");
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        King king = King.getInstance();

        king.say();
    }
}
```
