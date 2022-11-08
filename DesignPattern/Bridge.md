![](https://velog.velcdn.com/images/sanizzang00/post/1bd98526-b2c2-4ef9-b9e6-9144eaba5014/image.png)

기존 시스템에 새로운 기능을 추가해도 기능 계층을 통해 기존에 작성된 코드의 변경을 최소화시킬 수 있고,
또한 기존의 기능에 대해서도 구현 계층을 통해서 확장을 용이하게 해준다.

기능계층에 포함되는 것은 새로운 기능을 위한 메서드를 추가할 수 있는 클래스들
구현계층에 포함되는 것은 이미 정해진 인터페이스에 대한 구현 클래스들이다.
또한 이 기능 계층과 구현계층을 연결하기 위해서 Bridge 즉 다리를 놓아 연결한다는 의미해서
Bridge 패턴이라고 한다.

![](https://velog.velcdn.com/images/sanizzang00/post/89a11568-eebf-424f-b773-a8938202c272/image.png)

Draft와 Publication이 기능계층에 포함된 것들이고,
Display, SimpleDisplay, CaptionDisplay가 구현 계층에 포함된 것들이다.
Draft와 Display가 연결되어 있는 모습인데 마치 Bridge 즉 다리가 놓여져 있는 모습이다.

Draft.java

```java
public class Draft {
    private String title;
    private String author;
    private String[] content;

    public String getTitle() { return title; }
    public String getAuthor() { return author; }
    public String[] getContent() { return content; }

    public Draft(String title, String author, String[] content) {
        this.title = title;
        this.author = author;
        this.content = content;
    }

    public void print(Display display) {
        display.title(this);
        display.author(this);
        display.content(this);
    }
}
```

어떤 원고에 대한 제목, 저자, 내용에 대한 데이터를 담고있는 클래스이다.
그리고 이러한 데이터를 화면에 표시해준다. 표시해 주는 방식에 대한 구현은
Display 인터페이스에 맞기게 된다.

Display.java

```java
public interface Display {
    void title(Draft draft);
    void author(Draft draft);
    void content(Draft draft);
}
```

Draft 객체를 받아서 제목, 저자, 내용을 출력하는 메소드이다.

SimpleDisplay

```java
public class SimpleDisplay implements Display {

    @Override
    public void title(Draft draft) {
        System.out.println(draft.getTitle());
    }

    @Override
    public void author(Draft draft) {
        System.out.println(draft.getAuthor());
    }

    @Override
    public void content(Draft draft) {
        var content = draft.getContent();
        for(var i = 0; i< content.length; i++) {
            System.our.println(content[i]);
        }
    }
}
```

CaptionDisplay.java

```java
public class CaptionDisplay implements Display {

    @Override
    public void title(Draft draft) {
        System.out.println("제목: " + draft.getTitle());
    }

    @Override
    public void author(Draft draft) {
        System.out.println("저자: " + draft.getAuthor());
    }

    @Override
    public void content(Draft draft) {
        System.out.println("내용: ");
        var content = draft.getContent();
        for(var i = 0; i< content.length; i++) {
            System.our.println("    " + content[i]);
        }
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        var title = "복원된 지구";
        var author = "김형준";
        String[] content = {
            "플라스틱 사용의 감소와",
            "바다 생물 어획량 감소로 인하여",
            "지구는 복원되었다."
        };

        Draft draft = new Draft(title, author, content);

        Display display1 = new SimpleDisplay();
        // draft.print(display1);

        Display display2 = new CaptionDisplay();
        draft.print(display2);

        // 새로운 기능에 대해서 기존의 클래스에 대한 어떠한 변경도 없이 추가할 수 있다.
        var publisher = "북악출판"
        var cost = 100;

        Publication publication  = new Publication(title, author, content, publisher, cost);

        publication.print(display2);
    }
}
```

Publication.java

```java
public class Publication extends Draft {
    private String publisher;
    private int cost;

    public Publication(String title, String author, String[] content, String publisher, int cost) {
        super(title, author, content);
        this.publisher = pulisher;
        this.cost = cost;
    }

    private void printPublicationInfo() {
        System.out.println("#" + pulbisher + "$" + cost);
    }

    public void print(Display display) {
        super.print(display);
        printPublicationInfo();
    }
}
```
