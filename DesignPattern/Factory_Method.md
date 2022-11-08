### Factory Method Pattern

- 객체 생성을 위한 패턴
- 객체 생성에 필요한 과정을 템플릿처럼 정해 놓고 각 과정을 다양하게 구현이 가능함
- 구체적으로 생성할 클래스를 유연하게 정할 수 있음
- 객체 생성에 대한 인터페이스 구현의 분리

![](https://velog.velcdn.com/images/sanizzang00/post/9c8fc506-9c19-46d8-88d7-d2ab98752775/image.png)

필요한 시점에 칼이나 방패 활을 생성해주는데 각 아이템은 생성할 수 있는 개수에 제약이 있다.
아이템을 생성할 때 생성할 수 있는 개수를 초과하게 되면 더 이상 아이템을 생산할 수 없도록 한다.

Item.java

```java
public interface Item {
    void use();
}
```

Sword.java

```java
public class Sword implements Item {
    @Override
    public void use() {
        System.out.println("칼로 샥 베었다.");
    }
}
```

Shield.java

```java
public class Shield implements Item {
    @Override
    public void use() {
        System.out.println("방패로 공격을 막았다!");
    }
}
```

Bow.java

```java
public class Bow implements Item {
    @Override
    public void use() {
        System.out.println("화살로 멀리서 쐈다");
    }
}
```

Factory.java

```java
public abstract class Factory {
    // 생성하고자 하는 Item을 문자열로 받는다.
    public Item create(String name) {
        // 생성이 가능한지 확인
        boolean bCreatable = this.isCreateable(name);
        if(bCreatable) {
            Item item = this.createItem(name);
            postprocessItem(name);
            return item;
        }

        return null;
    }

    public abstract boolean isCreatable(String name);
    public abstract Item createItem(String name);
    public abstract void postprocessItem(String name);
}
```

ItemFactory.java

```java
public class ItemFactory extends Factory {
    // 각 아이템에 대한 최대의 생성 개수와 현재 생성된 아이템의 개수를 저장하기 위한 용도
    private class ItemData {
        int maxCount;
        int currentCount;
        ItemData(int maxCount) {
            this.maxCount = maxCount;
        }
    }

    private HashMap<String, ItemData> repository;

    public ItemFactory() {
        repository = new HashMap<String, ItemData>();

        repository.put("sword", new ItemData(3));
        repository.put("shield", new ItemData(2));
        repository.put("bow", new ItemData(1));
    }

    @Override
    public boolean isCreatable(String name) {
        ItemData itemData = repository.get(name);

        if(itemData == null) {
            System.out.println(name + "은 알 수 없는 아이템입니다.");
            return false;
        }

        if(itemData.currentCount >= itemData.maxCount) {
            System.out.println(name + "은 품절 아이템입니다.");
            return false;
        }

        return true;
    }

    @Override
    public Item createItem(String name) {
        Item item = null;

        if(name == "sword") item = new Sword();
        if(name == "shield") item = new Shield();
        if(name == "bow") item = new Bow();

        return item;
    }

    @Override
    public void postprocessItem(String name) {
        ItemData itemData = repository.get(name);
        if(itemData != null) itemData.currentCount++;
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        Factory factory = new ItemFactory();

        Item item1 = factory.create("sword");
        if(item1 != null) item1.use();

        Item item2 = factory.create("sword");
        if(item2 != null) item2.use();

        Item item3 = factory.create("sword");
        if(item3 != null) item3.use();

        Item item4 = factory.create("sword");
        if(item4 != null) item4.use();

        Item item5 = factory.create("sword");
        if(item5 != null) item5.use();
    }
}
```
