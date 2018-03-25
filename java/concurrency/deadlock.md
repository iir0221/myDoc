# 产生条件
* 多个sharedResource
* 线程在持有某个sharedResource的锁的同时，还将获取其他sharedResource的锁
* 获取sharedResource锁的顺序不固定

# example
```java
import java.util.concurrent.CopyOnWriteArrayList;

/**
 * Created by xinyuan.zhang on 1/28/18.
 */
public class EaterThread extends Thread{


    private String name;

    private final Tool lefthand;
    private final Tool righthand;


    public EaterThread(String name,Tool lefthand,Tool righthand) {
        this.name = name;
        this.lefthand = lefthand;
        this.righthand = righthand;
    }

    public void run() {
        while (true) {
            eat();
        }
    }


    public void eat() {

        //单独获取勺子和叉子
        synchronized (lefthand) {
            System.out.println(name+" takes up "+lefthand+" (left).");
            synchronized (righthand) {
                System.out.println(name+ " takes up "+righthand+" (right).");
                System.out.println(name+" is eating now!");
                System.out.println(name+" puts down "+righthand+" (right).");
            }

            System.out.println(name+" puts down "+lefthand+" (left).");
        }
    }
}
```
```java
public class Tool {

    private final String name;

    public Tool(String name) {
        this.name = name;
    }

    public String toString() {
        return "[ " +name+" ]";
    }
}
```
```java
public class Main {

    public static void main(String[] args) {
        System.out.println("Testing start!");

        Tool spoon =new Tool("Spoon");
        Tool fork = new Tool("Fork");

        // 两个Thread以不同的顺序获取餐具
        new EaterThread("Alice",spoon,fork).start();
        new EaterThread("Bob",fork,spoon).start();

        Collections.synchronizedList(new ArrayList<>());
    }
}
```