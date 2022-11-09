### Facade Pattern

- 어떤 기능을 처리하기 위해 여러 객체들 사이의 복잡한 메서드 사용을 감춰서 단순화 시켜주는 패턴

![](https://velog.velcdn.com/images/sanizzang00/post/f495949f-706e-4c22-a7d4-2cb08fedcd20/image.png)

Row.java

```java
public class Row {
    private String name;
    private String birthday;
    private String email;

    public Row(String name, String birthday, String email) {
        this.name = name;
        this.birthday = birthday;
        this.email = email;
    }

    public String getName() {
        return name;
    }

    public String getBirthday() {
        return birthday;
    }

    public String getEmail() {
        return email;
    }
}
```

DBMS.java

```java
import java.util.HashMap;

public class DBMS {
    private HashMap<String, Row> db = new HashMap<String, Row>();

    public DBMS() {
        db.put("jane", new Row("Jane", "1990-02-14", "jane09@geosee.co.kr"));
        db.put("robert", new Row("Robert", "1979-11-05", "nice@googl.com"));
        db.put("dorosh", new Row("Dorosh", "1985-08-21", "doshdo@nave.net"));
    }

    public Row query(String name) {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        return db.get(name.toLowerCase());
    }
}
```

Cache.java

```java
import java.util.HashMap;

public class Cache {
    private HashMap<String, Row> cache = new HashMap<String, Row>();

    public void put(Row row) {
        cache.put(row.getName(), row);
    }

    public Row get(String name) {
        return cache.get(name);
    }
}
```

DBMS에서 조회된 데이터를 임시로 담아두는 클래스
DBMS를 통해서 조회하면 속도가 느리므로 이 캐쉬를 사용해서 속도를 향상시킬 수 있다.

Message.java

```java
public class Message {
    private Row row;

    public Message(Row row) {
        this.row = row;
    }

    public String makeName() {
        return "Name: \"" + row.getName() + "\"";
    }

    public String makeBirthDay() {
        return "Birthday: " + row.getBirthDay();
    }

    public String makeEmail() {
        return "Email: " + row.getEmail();
    }
}
```

Row클래스 객체를 가공하는 클래스

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        DBMS dbms = new DBMS();
        Cache cache = new Cache();

        String name ="Dorosh";

        Row row = cache.get(name);
        if(row == null) {
            row = dbms.query(name);
            if(row != null) {
                cache.put(row);
            }
        }

        if(row != null) {
            Message message = new Message(row);

            System.out.println(message.makeName());
            System.out.println(message.makeBirthday());
            System.out.println(message.makeEmail());
        } else {
            System.out.println(name + " is not exists.");
        }
    }
}
```

![](https://velog.velcdn.com/images/sanizzang00/post/3f033a82-3380-430a-9f75-687636fffbdd/image.png)

Facade.java

```java
public class Facade {
    public DBMS dbms = new DBMS();
    private Cache cache = new Cache();

    public void run(String name){
        Row row = cache.get(name);
        if(row == null) {
            row = dbms.query(name);
            if(row != null) {
                cache.put(row);
            }
        }

        if(row != null) {
            Message message = new Message(row);

            System.out.println(message.makeName());
            System.out.println(message.makeBirthday());
            System.out.println(message.makeEmail());
        } else {
            System.out.println(name + " is not exists.");
        }
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        Facade facade = new Facade();

        String name ="Dorosh";

        facade.run(name);
    }
}
```
