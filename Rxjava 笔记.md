

# Rxjava 笔记

## 一）基本概念

- **Observable**： 表示数据源， Observable会发出一定数量的元素， 发送可能会成功， 也可能在这个过程中出现状况而失败。 从电影院的场景可以知道， 同一时间Observable可以有多个订阅者 。

- **Observer或Subscriber**： 表示订阅者， 通过监听Observable来消费Observable所发送的元素。  

- **Methods**： 表示一系列操作API， 对下发数据进行加工整合。  

- **onNext**： 在一个元素被Observable发送出去的时候， 通过该方法可以调用每一个订阅者。

- **onComplete**： 在Observable成功发送完所有数据后， 会调用这个方法来收尾。

- **onError**： 当Observable发送数据的过程中出现错误的状况时， 会调用这个方法结束发送并返回一个error事件。  ： 在一个元素被Observable发送出去的时候， 通过该方法可以调用每一个订阅者。

- **onComplete**： 在Observable成功发送完所有数据后， 会调用这个方法来收尾。

- **onError**： 当Observable发送数据的过程中出现错误的状况时， 会调用这个方法结束发送并返回一个error事件。  

  ---
  
  rxjava1：五大元素 -**Observable**，**Observer**，**Subscription，OnSubscribe，Subscriber**
  
  
  
  **Observable**：抽象类
  
  ​                        -----被观察者，创建可观察序列（create），通过subscribe注册一个观察者
  
  **Subscription**：一个接口，拥有unsubscribe和 isUnsubscribed两个方法。
  
  ​                        ------订阅，描述观察者和被观察者间的关系，可以取消获取当前订阅
  
  **Observer**：一个接口，几个On**方法。
  
  ​                      ------用于接收数据，作为观察者，作为Observable的subscribe方法参数
  
  **Subscriber**：抽象类，实现Observer和Subscription两个接口
  
  ​                      -------只有自己阻止自己
  
  **OnSubscribe**：一个接口，实现Action1-》call方法
  
  ```
  //create(OnSubscribe f),  返回new Observable(hook,onCreate(f)),hook返回f
  //返回的是传入的subscribe
  Subscription tSubscription = Observable.create(new Observable.OnSubscribe<String>() {
                  @Override
                  public void call(Subscriber<? super String> subscriber) {
                      if (!subscriber.isUnsubscribed()) {
                              subscriber.onNext("test");
                              subscriber.onCompleted();
                          }
                      }
                      //实际是subscribe方法调用了call方法
                  }).subscribe(new Observer<String>() {
                      @Override
                      public void onCompleted() {
                          System.out.println("onCompleted");
                      }
  
                      @Override
                      public void onError(Throwable e) {
  
                      }
  
                      @Override
                      public void onNext(String s) {
                          System.out.println("onNext:" + s);
                      }
                  });
  ```
  
  ---

rxjava2：基本元素 -- **Observable，Observer，Disposable，OnSubscribe，Emitter**

**Observable**：抽象类 实现ObservableSource接口 -->

```
public interface ObservableSource<T> {

    /**
     * Subscribes the given Observer to this ObservableSource instance.
     * @param observer the Observer, not null
     * @throws NullPointerException if {@code observer} is null
     */
    void subscribe(@NonNull Observer<? super T> observer);
}
```
**Observer**：接口，比rxjava1 多了onSubscribe(Disposable d)方法 onNext ,onError,onComplete



**Disposable**:接口，两个方法dispose() ,isDisposed()



**OnSubscribe**：接口，subscribe(ObservableEmitter e)

**Emitter**:接口，onNext，onError， onComplete


```

Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> e) throws Exception {
        if (!e.isDisposed()) {
            e.onNext("1");
            e.onNext("2");
            e.onNext("3");
            e.onNext("4");
            e.onNext("5");
            e.onNext("6");
            e.onNext("7");
            e.onNext("8");
            e.onNext("9");
            e.onNext("10");
            e.onComplete();
        }
    }
}).subscribe(new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {
        System.out.println("onSubscribe");
    }

    @Override
    public void onNext(String value) {
        System.out.println("onNext:" + value);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {
        System.out.println("onCompleted");
    }
});
```

create方法

```
public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
    ObjectHelper.requireNonNull(source, "source is null");
    //封装Function
    return RxJavaPlugins.onAssembly(new ObservableCreate<T>(source));
}
```

## 二）观察着模式

![image-20200113150722135](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200113150722135.png)