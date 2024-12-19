```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```



#### 运行时异常与编译时异常

运行时异常：

NullPointerException

ArrayIndexOutOfBoundsException

ClassCastException

编译时异常：

IOException

FileNotFoundException

ClassNotFoundException

InterruptedException
