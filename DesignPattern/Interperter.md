### Interpreter

- 문법에 맞춰 작성된 스크립트를 해석
- 해석된 구문을 정해진 규칙대로 실행하는 패턴

![](https://velog.velcdn.com/images/sanizzang00/post/09c921d4-99dc-4ea8-af2e-29fe987108cc/image.png)

![](https://velog.velcdn.com/images/sanizzang00/post/23e19943-204b-42f1-81ca-225308aa6ff5/image.png)

Context: 스크립트에서 구문을 뽑아내는 기능을 담당
Expression: 스크립트를 구성하는 각 구문을 처리하는 인터페이스
BeginExpression: BEGIN 구문을 처리하는 클래스
CommandListExpression: 여러개의 CommandExpression 클래스 객체를 가질 수 있는 클래스
CommandExpression: 실제 실행할 수 있는 명령에 대한 구문

Context.java

스크립트를 구성하는 구문을 하나씩 추출하는 기능을 제공

```java
public class Context {
    private StringTokenizer tokenizer;
    private String currentKeyword;

    public Context(String script) {
        tokenizer = new StringTokenizer(script);
        readNextKeyword();
    }

    public String readNextKeyword() {
        if(tokenizer.hasMoreTokens()) {
            currentKeyword = tokenizer.nextToken();
        } else {
            currnetKeyword = null;
        }

        return currentKeyword;
    }

    public String getCurrentKeyword() {
        return currentKeyword;
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        String script = "BEGIN FRONT LOOP 2 BACK RIGHT END BACK END";

        Context context = new Context(script);

        while(true) {
            String keyword = context.getCurrentKeyword();
            if(keyword == null) break;

            System.out.println(keyword);

            context.readNextKeyword();
        }
    }
}
```

Expression.java

```java
public interface Expression {
    boolean parse(Context context);
    boolean run();
}
```

BeginExpression.java

```java
public class BeginExpression implements Expression {
    private CommandListExpression expression;
}
```
