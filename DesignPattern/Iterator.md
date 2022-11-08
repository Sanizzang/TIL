### Container / Aggregator

- Array
- Linked List
- Tree
- Graph
- Table(DBMS)
  ...

동일한 형태의 데이터 항목을 여러개 가지고 있는 것을 Container 혹은 Aggregator라고 한다.

(https://velog.velcdn.com/images/sanizzang00/post/fe0ac988-7d21-480b-9e3a-b756b9b0f625/image.png)

Aggregator.java

```java
public interface Aggregator {
    Interator iterator();
}
```

Iterator.java

```java
public interface Iterator {
    // Aggregator의 다음 구성 데이터를 얻을 수 있도록 함.(얻을 수 있다면 True 반환)
    boolean next();
    // 구성 데이터를 하나 얻어 반환 (구성데이터에 대한 타입은 정해지지 않아야 하므로 Object여야 한다.)
    Object current();
}
```

Item.java

```java
public class Item {
    private String name;
    private int cost;

    public Item(String name, int cost) {
        this.name = name;
        this.cost = cost;
    }

    @Override
    public String toString() {
        return "(" + name + ", " + cost + ")";
    }
}
```

Array.java

```java
public class Array implements Aggregator {
    private Item[] items;

    public Array(Item[] items) {
        this.items = items;
    }

    public Item getItem(int index) {
        return items[index];
    }

    public int getCount() {
        return items.length;
    }

    @Override
    public Iterator iterator() {
        return new ArrayIterator(this);
    }
}
```

ArrayIterator.java

```java
public class ArrayIterator implements Iterator {
    private Array array;
    private int index;

    public ArrayIterator(Array array) {
        this.array = array;
        this.index = -1;
    }

    @Override
    public boolean next() {
        index++;
        return index < array.getCount();
    }

    @Override
    public Object current() {
        return array.getItem(index);
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        Item[] items = {
            new Item("CPU", 1000),
            new Item("KeyBoard", 2000),
            new Item("Mouse", 3000),
            new Item("HDD", 50)
        };

        Array array = new Array(items);
        Iterator it = array.iterator();

        while(it.next()) {
            Item item = (Item)it.current();
            System.out.println(item);
        }
    }
}
```

Iterator 패턴의 핵심은 배열이나 링크드리스트, 트리 등과 같은 다양한 형태의 컨테이너 즉
Aggregator의 구성 데이터를 참조할 수 있는 표준화된 공통 api를 제공할 수 있다는 점이다.
이렇게 되면 개발자는 다양한 데이터 구조를 파악하지 않아도 표준화된 한개의 api 만으로도
다양한 구조의 Aggregator의 구성 데이터를 참조할 수 있게 된다.
