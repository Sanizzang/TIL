### Prototype Pattern?

- 실행 중에 생성된 객체로 다른 객체를 생성하는 패턴
- Prototype으로 만든 객체의 상태를 변경해도 원본 객체의 상태는 변경되지 않음(Deep Copy; 깊은 복사)

![](https://velog.velcdn.com/images/sanizzang00/post/9160bdc8-59d1-4a81-89e5-9d379ace2bcf/image.png)
Prototype: 객체로부터 또 다른 새로운 객체를 생성해 내는 api를 가지고 있다.
Shape: 도형에 대한 api를 가지고 있다.
Group: Shape 인터페이스의 객체를 여러개 가질 수 있는 클래스, Shape 인터페이스와 호환되는
여러개의 객체를 이용해서 새로운 도형을 구성하기 위한 클래스이다.

Prototype.java

```java
public interface Prototype {
    // 새로운 객체를 복사해서 생성하는 copy메소드
    Object copy();
}
```

Shape.java

```java
public interface Shape {
    // 도형을 표현하는 문자열을 반환
    String draw();

    // 도형을 일정 거리만큼 이동시킴
    void moveOffset(int dx, int dy);
}
```

Point.java

```java
public class Point implements Shape, Prototype {
    private int x;
    private int y;

    public Point setX(int X) {
        this.x = x;
        return this;
    }

    public Point setY(int y) {
        this.y = y;
        return this;
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }

    @Override
    public Object copy() {
        Point newPoint = new Point();

        newPoint.x = this.x;
        newPoint.y = this.y;

        return newPoint;
    }

    @Override
    public String draw() {
        return "(" + x + " " + y + ")";
    }

    @Override
    public void moveOffset(int dx, int dy) {
        this.x += dx;
        this.y += dy;
    }
}
```

Line.java

```java
public class Line implements Shape, Prototype {
    private Point startPt;
    private Point endPt;

    public Line setStartPoint(Point pt) {
        this.startPt = pt;
        return this;
    }

    public Line setEndPoint(Point pt) {
        this.endPt = pt;
        return this;
    }

    public Point getStartPoint() {
        return startPt;
    }

    public Point getEndPoint() {
        return endPt;
    }

    @Override
    public Object copy() {
        Line newLine = new Line();

        newLine.startPt = (Point)startPt.copy();
        newLine.endPt = (Point)endPt.copy();

        return newLine;
    }

    @Override
    public String draw() {
        return "LINE(" + startPt.draw() + "," + endPt.draw() + ")";
    }

    @Override
    public void moveOffset(int dx, int dy) {
        startPt.moveOffset(dx, dy);
        endPt.moveOffset(dx, dy);
    }
}
```

Group.java

```java
import java.util.ArrayList;

public class Group implements Shape, Prototype {
    private ArrayList<Shape> shapeList = new ArrayList<Shape>();

    private String name;

    public Group(String name) {
        this.name = name;
    }

    Group addShape(Shape shape) {
        shapeList.add(shape);
        return this;
    }

    @Override
    public Object copy() {
        Group newGroup = new Group(name);

        Iterator<Shape> iter = shapeList.iterator();
        while(iter.hasNext()) {
            Prototype shape = (Prototype)iter.next();
            newGroup.shapeList.add((Shape)shape.copy());
        }

        return new Group
    }

    @Override
    public String draw() {
        StringBuffer result = new StringBuffer(name);
        result.append("(");

        Iterator<Shape> iter = shapeList.iterator();
        while(iter.hasNext()) {
            Shape shape = iter.next();
            result.append(shape.draw());
            if(iter.hasNext()) {
                result.append(" ");
            }
        }

        result.append(")");
        return result.toString();
    }

    @Override
    public void moveOffset(int dx, int dy) {
        Iterator<Shape> iter = shapeList.iterator();
        while(iter.hasNext()) {
            Shape shape = iter.next();
            shape.moveOffset(dx, dy);
        }
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        Point pt1 = new Point();
        pt1.setX(0),setY(0);

        Point pt2 = new Point();
        pt2.setX(100).setY(0);

        Line line1 = new Line();
        line1.setStartPoint(pt1).setEndPoint(pt2);

        Line lineCopy = (Line)line1.copy();

        Point pt3 = new Point();
        pt3.setX(100).setY(100);

        Point pt4 = new Point();
        pt4.setX(0).setY(100);

        Line line2 = new Line();
        line2.setStartPoint(pt2).setEndPoint(pt3);

        Line line3 = new Line();
        line2.setStartPoint(pt3).setEndPoint(pt4);

        Line line4 = new Line();
        line2.setStartPoint(pt4).setEndPoint(pt1);

        Group rect = new Group("RECT");
        rect.addShape(line1).addShape(line2).addShape(line3).addShape(line4);
        System.out.println(rect.draw());

        Group cloneRect = (Group)rect.copy();
        cloneRect.moveOffset(100, 100);

        System.out.println(cloneRect.draw());
        System.out.println(rect.draw());

    }
}
```
