### Decorator Pattern?

- Decorator의 뜻은 "장식하는 사람"
- 기능을 마치 장식처럼 계속 추가할 수 있는 패턴
- 기능을 실행 중에 동적으로 변경 또는 확장 할 수 있는 패턴

![](https://velog.velcdn.com/images/sanizzang00/post/a0056baa-5d1c-44c8-abea-4965ffb9ede6/image.png)

Strings: 문자열을 여러개 가지고 있는 클래스, 장식할 대상이 되는 내용물에 해당
Decorator: Strings에 대한 장식을 나타내는 클래스
Item: Strings, Decorator의 다른 개념을 하나의 개념으로 다룰 수 있는 장치

Strings는 아무런 장식이 없는 원래의 내용물이고 이 내용물에 여러가지 장식을 추가하게 된다.
원래의 내용과 장식이 Item 클래스를 상속 받게 함으로써 내용물과 장식에 대한 구분이 사라지게 되고
둘을 마치 동일한 개념으로 다룰 수 있게 된다.

![](https://velog.velcdn.com/images/sanizzang00/post/dc454449-a441-4e2b-9903-8f0e8c9b7c4c/image.png)

Item.java

```java
public abstract class Item {
    // 문자라인 길이 반환
    public abstract int getLinesCount();
    // 문자열의 길이가 가장 긴 문자열 길이 반환
    public abstract int getMaxLength();
    // 지정된 문자의 길이 반환
    public abstract int getLength(int index);
    // 지정된 문자 반환
    public abstract String getString(int index);

    public void print() {
        int cntLines = getLinesCount();
        for(int i = 0;i < cntLines;i++) {
            String string = getString(i);
            System.out.println(string);
        }
    }
}
```

Strings.java

```java
public class Strings extends Item {
    private ArrayList<String> strings = new ArrayList<String>();

    @Override
    public int getLinesCount() {
        return strings.size();
    }

    @Override
    public int getMaxLength() {
        Iterator<String> iter = strings.iterator();
        int maxWidth = 0;

        while(iter.hasNext()) {
            String string = iter.next();
            int width = string.length();

            if(width > maxWidth) maxWidth = width;
        }

        return maxWidth;
    }

    @Override
    public int getLength(int index) {
        String string = strings.get(index);
        return string.length();
    }

    @Override
    public String getString(int index) {
        String string = strings.get(index);
        return string;
    }
}
```

Decorator.java

```java
public abstract class Decorator extends Item {
    protected Item item;

    public Decorator(Item item) {
        this.item = item;
    }
}
```

장식을 원래 내용물에 대해서도 할 수 있고 장식에 대해서도 장식을 할 수 있다.

SideDecorator.java

```java
public class SideDecorator extends Decorator {
    private Character ch;

    public SideDecorator(Item item, Character ch) {
        super(item);
        this.ch = ch;
    }

    @Override
    public int getLinesCount() {
        return item.getLinesCount();
    }

    @Override
    public int getMaxLength() {
        return item.getMaxLength() + 2;
    }

    @Override
    public int getLength(int index) {
        return item.getLength(index) + 2;
    }

    @Override
    public String getString(int index) {
        return ch + item.getString(index) + ch;
    }
}
```

BoxDecorator.java

```java
public class BoxDecorator extends Decorator {
    public BoxDecorator(Item item) {
        super(item);
    }

    @Override
    public int getLinesCount() {
        return item.getLinesCount() + 2;
    }

    @Override
    public int getMaxLength() {
        return item.getMaxLength() + 2;
    }

    @Override
    public int getLength(int index) {
        return item.getLength(index) + 2;
    }

    @Override
    public String getString(int index) {
        int maxWidth = this.getMaxLength();
        if(index == 0 || index == getLindesCount() - 1) {
            StringBuilder sb = new StringBuiler();
            sb.append('+');
            for(int i = 0;i < maxWidth - 2;i++) {
                sb.append('-');
            }
            sb.append('+');
            return sb.toString();
        } else {
            return '|' + item.getString(index - 1) + makeTailString(maxWidth - getLength(index - 1));
        }
    }

    private String makeTailString(int count) {
        StringBuilder sb = new StringBuilder();
        for(int i = 0;i < count;i++) {
            sb.append(' ');
        }
        sb.append('|');

        return sb.toString();
    }
}
```

LineNumberDecorator.java

```java
public class LineNumberDecorator extends Decorator {
    public LineNumberDecorator(Item item) {
        super(item);
    }

    @Override
    public int getLinesCount() {
        return item.getLinesCount();
    }

    @Override
    public int getMaxLength() {
        return item.getMaxLength() + 4;
    }

    @Override
    public int getLength(int index) {
        return item.getLength(index) + 4;
    }

    @Override
    public String getString(int index) {
        return String.format("%02d", index) + ": " + item.getString(index);
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        Strings strings = new Strings();

        strings.add("Hello~");
        strings.add("My Name is Kim Hyoung-Jun.");
        strings.add("I am a GIS Developer.");
        strings.add("Design-Pattern is powerful tool.");

        //strings.print();

        Item decorator = new SideDecorator(strings, '\"');
        decorator = new LineNumberDecorator(decorator);
        decorator = new BoxDecorator(decorator);

        decorator.print();
    }
}
```
