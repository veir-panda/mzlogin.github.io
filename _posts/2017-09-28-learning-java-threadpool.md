## 需求
在一个项目中需要调用Python写的服务进行更快速快速的计算，因为Python在数据处理方面比Java快很多。但是，要计算的数据量太大，有几千条数据，每条数据都要调用一次Python,那效率就很低，这时候，就要考虑多线程了。

可以多开几个线程来分别处理这些数据，但是数据怎么分配到各个线程，线程的数量又是多少合适，这又是一个问题，这里使用线程池来解决。

## 难点
1. 应该开几个线程才合适？（少了达不到目的，多了反而会因为线程之间的切换降低效率）；
2. 这些数据应该怎么分配给各个线程（平均分？那就要遍历每条数据，每多少条数据又要建立一个引用，传给不同的线程来处理，但这比较麻烦）
3. 每条数据计算完都会有一个计算结果，怎么在多线程中返回数据？（Thread类和Runnable接口的run方法返回类型都是void）
4. 每条数据的计算不可能都成功，因为Python是用Socket写的一个监听器来监听某个端口，然后用Java写个Socket来发起请求，当线程数量过多，与Python请求量过大，会使Python不能及时响应，然后Java端就会因为超时而抛出异常。

## 解决
1. 针对前两个问题，可以使用线程池来解决。Java中有两个常用的new一个线程池的方法：

```java
//new 一个线程池
ExecutorService exec = Executors.newCachedThreadPool();//线程数量根据需要创建
ExecutorService exec = Executors.newFixedThreadPool(THREAD_POOL_SIZE);//固定的线程数量
```
创建好线程池之后，就可以直接遍历所有需要计算的数据，对每条数据调用下面的方法
```java
exec.submit(Callable<T>);//调用线程池中的空闲的线程来计算，参数为实现一个Callable接口的类
```
上面的参数是一个实现Callable接口的类，而这就是解决第三个问题的办法：Callable中的核心方法call是有返回类型的，返回类型是一个泛型，实现该接口时传入的类型就是线程返回的结果类型；那怎么来接收返回数据呢？


Future接口就是用来接收Callable返回的数据的，他还可以对线程的执行状态进行查询、取消等操作。这里我们只需要调用下面的方法即可取得线程返回的数据

```java
future.get();//获取线程返回的数据
```
2. 针对最后一个问题，我想到的解决办法是**建立失败重连机制**:new一个failedArrayList，存放每个线程里某条数据因为检查到Python不能及时响应（返回为空）而重新将该条数据暂存到这里，以备下一次重新计算。这里要设置一个允许失败的次数times，如果第一次调用完检查到failedArrayList不为空，那就重新进行第二次计算，这时计算的就不是所有的数据了，而是failedArrayList  copy出来的数据（**注意：这里不能只是声明一个变量来引用，因为每次计算前都会清空failedArrayList**）;

只是这样又会有另一个特别严重的问题：failedArrayList需要在不同的线程中操作，要保证在某一个线程需要添加一条计算失败的数据到failedArrayList时数据的一致性（也就是线程安全），好在Java有一个方法：Collections.synchronizedList(List <T>)，这个方法可以保证failedArrayList的线程安全。

好了，接下来的事情就好办多了。下面附上关键部分的源码

## 源码

```java
/*ExecutorService exec = Executors.newCachedThreadPool();//这里是创建任务数量的线程数*/
ExecutorService exec = Executors.newFixedThreadPool(THREAD_POOL_SIZE);//创建指定大小的线程数
ArrayList<Future<Map<String, Object>>> result = new ArrayList<>();

retList = new ArrayList<>();
long t1 = System.currentTimeMillis();//记录开始计算前的时间

List<Map<String, Object>> failedList = Collections.synchronizedList(new ArrayList<>());//对failedList加上线程安全
List<Map<String, Object>> currentList = dataList;//datalist就是要计算的数据
int times = 0;//这里为了后面方便打印出是第几次，为从0开始，下面的RETY_TIMES=5
while(times++ < RETY_TIMES) {
	failedList.clear();//先清空原有的failedList
	//遍历计算
	for (Map<String, Object> stationMap : currentList) {
		//CorrelationByPythonMultithreading就是一个实现了Callable接口的类，在里面重写了call方法，返回Map<String, Object>类型
		result.add(exec.submit(new CorrelationByPythonMultithreading(failedList, stationMap, stationName, lonx, laty, val1, val2)));
	}
	//获取计算结果
	for(Future<Map<String, Object>> future : result){
		try {
			Map<String, Object> tmp = future.get();//获取call方法返回的结果
			if (tmp != null) {
				retList.add(tmp);//计算成功时，将计算结果添加到retList中
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
	}
	System.out.println("第"+times+"次计算完成");
	if (failedList.size() > 0) {
		currentList = new ArrayList<>(failedList);//将失败的list copy到要计算的list中
		System.out.println("本次计算失败个数："+failedList.size()+" ,正在重试……");
		continue;
	}
	else {
		break;
	}
}
if (failedList.size() > 0) {
	System.out.println("调用Python计算失败，服务器负载过高，本次还有 "+failedList.size()+"个点未计算");
	retData.put("retcode", -1);
	retData.put("retmsg", "调用Python计算失败");
	return retData;
}
long t2 = System.currentTimeMillis();
System.out.println("计算"+retList.size()+"个点耗时："+(t2-t1));
System.out.println("最大线程数："+((ThreadPoolExecutor)exec).getLargestPoolSize());;
exec.shutdownNow();//最后，一定不要忘了关掉线程池！！！
```