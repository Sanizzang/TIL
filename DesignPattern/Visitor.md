### Visitor

- 데이터 구조와 데이터 처리를 분리해주는 패턴
- 데이터 처리 방식을 기존의 소스 코드 변경 없이 새로운 클래스 추가만으로 확장할 수 있음
- 데이터 구조는 Composite Pattern을 사용해 표현함

![](https://velog.velcdn.com/images/sanizzang00/post/93178bb3-0f51-4603-a05d-ffbe507ea127/image.png)

Unit.java

```java
public interface Unit {
    void accept(Visitor visitor);
}
```

Item.java

```java
public class Item implements Unit {
    private int value;

    public Item(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}
```

ItemList.java

```java
public class ItemList implements Unit {
    private ArrayList<Unit> list = new ArrayList<Unit>();

    public void add(Unit unit) {
        list.add(unit);
    }

    @Override
    public void accept(Visitor visitor) {
        Iterator<Unit> iter = list.iterator();

        while(iter.hasNext()) {
            Unit unit = iter.next();
            visitor.visit(unit);
        }
    }
}
```

Visitor.java

```java
public interface Visitor {
    void visit(Unit unit);
}
```

SumVisitor.java

데이터를 구성하는 값들의 합을 계산하는 기능을 제공

```java
public class SumVisitor implements Visitor {
    private int sum = 0;

    public int getValue() {
        return sum;
    }

    @Override
    public void visit(Unit unit) {
        if(unit instanceof Item) {
            sum += ((Item)unit).getValue();
        } else {
            unit.accept(this);
        }
    }
}
```

AvgVisitor.java

```java
public class AvgVisitor implements Visitor {
    private int sum = 0;
    private int count = 0;

    public double getValue() {
        return sum / count;
    }

    @Override
    public void visit(Unit unit) {
        if(unit instanceof Item) {
            sum += ((Item)unit).getValue();
            count++;
        } else {
            unit.accept(this);
        }
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        ItemList list1 = new ItemList();
        list1.add(new Item(10));
        list1.add(new Item(20));
        list1.add(new Item(40));

        ItemList list2 = new ItemList();
        list2.add(new Item(30));
        list2.add(new Item(40));
        list1.add(list2);

        ItemList list3 = new ItemList();
        list3.add(new Item(25));
        list2.add(list3);

        SumVisitor sum = new Sumvisitor();
        list1.accept(sum);
        System.out.println("합계: " + sum.getValue());

        AvgVisitor avg = new AvgVisitor();
        list1.accept(avg);
        System.out.println("평균: " + avg.getValue());
    }
}
```
