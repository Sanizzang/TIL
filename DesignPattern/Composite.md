### Composite Pattern?

컴포지트 패턴이란 단일체와 집합체를 하나의 동일한 개념으로 처리하기 위한 패턴이다.

![](https://velog.velcdn.com/images/sanizzang00/post/4031e06f-b37d-45f9-bdd1-ee389f8c5e5c/image.png)

집합체로써 상자가 있다. 상자를 집합체라고 하는 이유는 상자 안에는 여러 개의 구슬을 담을 수 있기 때문이다.
여러가지 상황을 단 하나의 개념으로 처리해서 상황을 단순하게 만들어주는 패턴이 컴포지트 패턴이다.

![](https://velog.velcdn.com/images/sanizzang00/post/342f6057-9651-4701-9a62-1b426b7b441f/image.png)

```java
public abstract class Unit {
    private String name;

    public Unit(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return name + "(" + getSize() + ")";
    }

    public abstract int getSize();
}
```

File.java

```java
public class File extends Unit {
    private int size;

    public File(String name, int size) {
        super(name);
        this.size = size;
    }

    @Override
    public int getSize() {
        return size;
    }
}
```

Folder.java

```java
public class Folder extends Unit {
    private LinkedList<Unit> unitList = new LinkedList<Unit>();

    public Folder(String name) {
        super(name);
    }

    @Override
    public int getSize() {
        int size = 0;
        Iterator<Unit> it = unitList.iterator();

        while(it.hasNext()) {
            Unit unit = it.next();
            size += unit.getIsze();
        }

        return size;
    }

    public boolean add(Unit unit) {
        unitList.add(unit);
        return true;
    }

    private void list(String indent, Unit unit) {
        if(unit instanceof File) {
            System.out.println(indent + unit);
        } else {
            Folder dir = (Folder)unit;
            Iterator<Unit> it = dir.unitList.iterator();
            System.out.println(indent + "+ " + unit);
            while(it.hasNext()) {
                list(indent + "    ", it.next());
            }
        }
    }

    public void list() {
        list("", this);
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        Folder root = new Folder("root");
        root.add(new File("a.txt", 1000));
        root.add(new File("b.txt", 2000));

        Folder sub1 = new Folder("sub1");
        root.add(sub1);
        sub1.add(new File("sa.txt", 100));
        sub1.add(new Fiel("sb.txt", 4000));

        Folder sub2 = new Folder("sub2");
        root.add(sub2);
        sub2.add(new File("SA.txt", 250));
        sub2.add(new Fiel("SB.txt", 340));

        root.list();
    }
}
```
