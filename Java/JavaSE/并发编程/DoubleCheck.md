``` java
public class doubleCheck {
    private volatile static doubleCheck instance;

    private static doubleCheck getInstance(){
        if (instance == null) {
            synchronized (doubleCheck.class){
                if(instance == null){
                    instance = new doubleCheck();
                }
            }
        }
        return instance;
    }
}

```