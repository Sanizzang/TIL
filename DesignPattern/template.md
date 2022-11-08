### 템플릿 패턴?

어떤 기능에 대해 실행되어야 할 각 단계에 대한 각 순서를 정해놓고
각 단계에 대한 세부 구현을 상황에 맞게 다르게 구현해 낼 수 있는 패턴

![](https://velog.velcdn.com/images/sanizzang00/post/3cdc6a39-3102-402c-a59a-41c424b2d87c/image.png)

![](https://velog.velcdn.com/images/sanizzang00/post/f4b0a9f8-01ca-4ca4-9a8d-6e1e27fb6d84/image.png)

템플릿 패턴의 한가지 예

DisplayArticleTemplate: 아직 구현되지 않은 각 단계를 정해진 순서대로 실행해주는 클래스
각 단계에 해당하는 메서드를 순서대로 실행만해주는 클래스 이 클래스는 추상클래스인데
이유는 단계 1,2,3에 대한 method는 이 클래스 내부에 선언만 되어있을뿐 코드로 구현되어
있지 않기 때문이다.

SimpleDisplayArticle과 CaptionDisplayArticle 클래스는 DisplayArticleTemplate 클래스를
상속받아서 각 단계에 대한 method를 구체적으로 구현한다.

Article은 출력할 데이터를 얻을 수 있는 클래스 이다.

Article.java

```java
import java.util.ArrayList;

public class Article{
    private String title;
    private ArrayList<String> content;
    private String footer;

    public Article(String title, ArrayList<String> content, String footer) {
        this.title = title;
        this.content = content;
        this.footer = footer;
    }

    public String getTitle() {
        return title;
    }

    public ArrayList<String> getContent() {
        return content;
    }

    public String getFooter() {
        return this.footer;
    }
}
```

제목, 내용, 밑바닥 글로 구성

DisplayArticleTemplate.java

```java
public abstract class DisplayArticleTemplate {
    protected Article article;

    public DisplayArticleTemplate(Article article) {
        this.article = article;
    }

    public final void display() {
        title();
        content();
        footer();
    }

    protected abstract void title();
    protected abstract void content();
    protected abstract void footer();
}
```

SimpleDisplayArticle.java

```java
import java.util.ArrayList;

public class SimpleDisplayArticle extends DisplayArticleTemplate {
    public SimpleDisplayArticle(Article article) {
        super(article);
    }

    @Override
    protected void title() {
        System.out.println(article.getTitle());
    }

    @Override
    protected void content() {
        ArrayList<String> content = article.getContent();
        int cntLines = content.size();
        for(int i = 0;i < cntLines;i++) {
            System.out.println(content.get(i));
        }
    }

    @Override
    protected void footer() {
        System.out.println(article.getFooter());
    }
}
```

DisplayArticleTemplate.java

```java
public abstract class DisplayArticleTemplate {
    protected Article article;

    public DisplayArticleTemplate(Article article) {
        this.article = article;
    }

    public final void display() {
        title();
        content();
        footer();
    }

    protected abstract void title();
    protected abstract void content();
    protected abstract void footer();
}
```

CaptionDisplayArticle.java

```java
import java.util.ArrayList;

public class CaptionDisplayArticle extends DisplayArticleTemplate {
    public CaptionDisplayArticle(Article article) {
        super(article);
    }

    @Override
    protected void title() {
        System.out.println("TITLE: " + article.getTitle());
    }

    @Override
    protected void content() {
        System.out.println("CONTENT:");
        ArrayList<String> content = article.getContent();
        int cntLines = content.size();
        for(int i = 0;i < cntLines;i++) {
            System.out.println("    " +content.get(i));
        }
    }

    @Override
    protected void footer() {
        System.out.println("FOOTER" + article.getFooter());
    }
}
```

MainEntry.java

```java
import java.util.ArrayList;

public class MainEntry {
     public static void main(String[] args) {
        String title = "디자인패턴";

        ArrayList<String> content = new ArrayList<String>();
        content.add("디자인패턴은 클래스 간의 효율적이고 최적화된 관계를 설계하는 것입니다.");
        content.add("각 패턴을 외우기 보다 이해하는 것이 중요합니다.");
        content.add("다양한 패턴을 접하고 반복적으로 이해할수록");
        content.add("각 디자인패턴에 대한 응용의 폭이 넓어질 것입니다.");

        String footer = "2022.1.9, GIS Developer 김형준";

        Article article = new Article(title, content, footer);

        System.out.println("[CASE-1]");
        DisplayArticleTemplate style1 = new SimpleDisplayArticle(article);
        style.display();

        System.out.println();


        System.out.println("[CASE-2]");
        DisplayArticleTemplate style1 = new CaptionDisplayArticle(article);
        style.display();
     }
}
```
