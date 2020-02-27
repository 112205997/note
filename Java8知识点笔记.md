# Java8知识点笔记

## 一）分支/合并

ForkJoinPool-》ExecutorService接口的实现

RecursiveTask<R> -> 唯一抽象方法 compute，在这个方法中定义子任务拆分逻辑。业务逻辑如下

![image-20200116092441243](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200116092441243.png)

递归拆分逻辑如下

![image-20200116092601207](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200116092601207.png)

代码例子

![image-20200116092700640](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200116092700640.png)

![image-20200116093027138](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200116093027138.png)

最佳实践

- 对一个任务调用join方法会阻塞调用方，直到该任务做出结果。因此，有必要在两个子
  任务的计算都开始之后再调用它。否则，你得到的版本会比原始的顺序算法更慢更复杂，
  因为每个子任务都必须等待另一个子任务完成才能启动。
- 不应该在RecursiveTask内部使用ForkJoinPool的invoke方法。相反，你应该始终直
  接调用compute或fork方法，只有顺序代码才应该用invoke来启动并行计算。
- 对子任务调用fork方法可以把它排进ForkJoinPool。同时对左边和右边的子任务调用
  它似乎很自然，但这样做的效率要比直接对其中一个调用compute低。这样做你可以为
  其中一个子任务重用同一线程，从而避免在线程池中多分配一个任务造成的开销。
-  调试使用分支/合并框架的并行计算可能有点棘手。特别是你平常都在你喜欢的IDE里面
  看栈跟踪（stack trace）来找问题，但放在分支合并计算上就不行了，因为调用compute
  的线程并不是概念上的调用方，后者是调用fork的那个。
- 和并行流一样，你不应理所当然地认为在多核处理器上使用分支/合并框架就比顺序计
  算快。我们已经说过，一个任务可以分解成多个独立的子任务，才能让性能在并行化时
  有所提升。所有这些子任务的运行时间都应该比分出新任务所花的时间长；一个惯用方
  法是把输入/输出放在一个子任务里，计算放在另一个里，这样计算就可以和输入/输出
  同时进行。此外，在比较同一算法的顺序和并行版本的性能时还有别的因素要考虑。就
  像任何其他Java代码一样，分支/合并框架需要“预热”或者说要执行几遍才会被JIT编
  译器优化。这就是为什么在测量性能之前跑几遍程序很重要，我们的测试框架就是这么
  做的。同时还要知道，编译器内置的优化可能会为顺序版本带来一些优势（例如执行死
  码分析——删去从未被使用的计算）  

Java8中使用Spliterator拆分流

## 二）CompletableFuture组合式异步编程

### 1.Future+Callable

![image-20200117084751394](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117084751394.png)
基于该模式的一个异步接口实现

![image-20200117131324811](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117131324811.png)

![image-20200117131438965](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117131438965.png)



![image-20200117084925918](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117084925918.png)



缺点：难以处理多个异步计算合并，异步计算完成主动通知需要借助回调，异常的处理也复杂。

### 2.使用CompletableFutre

使用工厂方法supplyAsync构建异步接口，该方法接受一个Supplier作为参数，返回CompletableFutre对象。生产方法交给ForkJionPool池执行，也提供了错误管理机制。

代码等同11-6.

![image-20200117132108775](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117132108775.png)

#### 2.1调用多个阻塞接口时的异步优化

假设从一组不同的shop中查询一个产品的价格。

![image-20200117132741907](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117132741907.png)

使用CompletableFuture改进，每个查询都成为异步的执行，join类似于Futrue的get方法，但不会抛出异常。

注意这里使用两个流，如果在一个流中处理，join会把并行变为阻塞同步的顺序执行。

这种方法在使用线程数量跟并行流一样，取决于Runtime.getRunTime().availableProcessors()的返回值，因此性能差异不大，但通过定制执行器可以提升。

![image-20200117133138937](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117133138937.png)

定制执行器改进版

![image-20200117134015710](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117134015710.png)

更改核心代码

![image-20200117134131302](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117134131302.png)

#### 2.2 对多个异步任务进行流水操作

假设从不同商户查询到产品价格，再调用折扣服务计算实际价格。

![image-20200117135343105](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117135343105.png)

方案1：使用并行流，缺点是商户数量上升时扩张性不好。

方案2：使用异步方式

![image-20200117135428417](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117135428417.png)

![image-20200117135626464](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117135626464.png)

**thenApply**：这是一个同步操作，用在不延时的数据转换。

**thenCompose**：允许对两个异步操作进行流水线，第一个操作完成时，将结果作为参数传给第二个操作。这个不会阻塞。

**thenComposeAsync**：将任务提交到另一个线程池

**thenCombine**：对两个结果不相干的CompleteFuture对象的结果整合起来，也不需要有前后执行顺序。它接受BiFunction作为参数。

**thenCombineAsync**：合并操作会提交到线程池中，由另一个任务以异步方式执行

**thenAccept**（同样也有异步版本）：接受CompletableFuture执行完的返回值做参数。返回CompletableFuture<Void>,所以不能做更多操作，只能等待结束。

场景：查询价格，同时查询汇率。当两者结束后，合并两个结果，用汇率计算出美元的计价。这里只是乘法计算，所以可以用非异步的方式合并计算。

1）Java7的实现

![image-20200117170223302](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117170223302.png)
2）Java8的实现

![image-20200117165904960](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117165904960.png)

![image-20200117170045303](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117170045303.png)

#### 2.3 响应CompletableFuture的completion事件

调用get或join方法会造成阻塞，直到CompletableFuture完成才能继续，如果需要尽快将商品价格返回，而不是等待全部计算完成。

![image-20200117172532468](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117172532468.png)





![image-20200117175221874](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117175221874.png)

**allOf**工厂方法接收一个CompletableFuture构成的数组，数组中所有的CompletableFuture执行完成后，返回一个CompletableFuture(Void)对象，因此可以用join。

**anyOf**可以是对象数组中任何一个执行完毕就不在等待。

![image-20200117175836827](C:\Users\wei jinwei\AppData\Roaming\Typora\typora-user-images\image-20200117175836827.png)