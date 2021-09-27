# 코루틴 사용해보기
해당 예제는 프로젝트들을 사용해 보았으나 [코틀린 공식문서](https://kotlinlang.org/docs/coroutines-guide.html#table-of-contents)에 있는 코루틴 튜토리얼을 실습해보면서 파트를 작성하는게 보다 나을것이여서 예제위주로 작성하였습니다.
***
 ## :wrench: 프로젝트 사용 사례
 
 + [코루틴 깃허브 레파지토리 앱](https://github.com/tnvnfdla1214/github_repository)
 + ToDo(https://github.com/tnvnfdla1214/ToDo)
 ***
 ## :lollipop: 예제
 본예제 실습 환경을 안드로이드 스튜디오 Junit4에서 작성되었습니다.
 해당 예제들은 코루틴 깃허브 레파지토리 앱의 테스트 레파지토리에 있으니 확인 가능합니다.
 
 ***
 
 ### Basic
 - Coroutine Builder
   - launch
   - runBlocking
 - Scope
   - CoroutineScope
   - GlobalScope
 - Suspend funtion
   - suspend
   - delay()
   - join()
 - Structured concurrency
 ```Kotlin
    fun main1()  {
        GlobalScope.launch { // coroutineScope
            delay(1000L)
            println("World!")
        }
        println("Hello")
        Thread.sleep(2000L)
    }
```
 ```Kotlin
  Hello
  World!
```
쓰레드로 바꾸어도 결과는 똑같이 나온다.
 ```Kotlin
    fun main3(){
        Thread {
            Thread.sleep(1000L) //sleep은 쓰레드에서만 동작한다.
            println("World!")
        }
        println("Hello")
    }
```
 ```Kotlin
    fun main4()  {
        GlobalScope.launch { // 자신이 호출한 쓰레드를 블록킹하지 않는다.
            delay(1000L) //코루틴 안에서만 가능한 function 이다.
            println("World!")
        }
        println("Hello")
        runBlocking { //코루틴 빌더 중 일종이며 자신이 호출한 쓰레드를 블록킹한다.
            delay(2000L)
        }
    }
```
위의 예제를 간단히 하면 아래와 같다. (main의 문이 모두 코루틴의 방식으로 따르기때문에아래와 같은 방식으로 감싸준다.)
 ```Kotlin
    fun main2() = runBlocking {
        launch { // 코루틴 빌더
            delay(1000L) //suspend funtion
            println("World!")
        }
        println("Hello")
        delay(2000L)
    }
```
위와 같은 예제는 만약 launch가 2초이상을 넘을시 실행되지 않으므로 좋은 코드가 아니다. 따라서 위의 실행을 기다리기 위해서는 job을 사용해 결과값을 기다려준다.launch에서 반환하는 값은 job을 기억하자
 ```Kotlin
    fun main5() = runBlocking {
        val job = launch {
            delay(3000L)
            println("World!")
        }
        println("Hello")
        job.join()
    }
```
또 구조적으로 작성시 모든 job을 매번 작성해주는 번거러움이 발생할 수 있다. 그러므로 runblocking을 launch를 해줌으로서 구조적으로 join을 하지 않더라고 launch의 값을 반한할 수 있게 한다.
 ```Kotlin
    fun main6() = runBlocking {
         this.launch {
            delay(1000L)
            println("World!!!!")
        }
        launch { //runblacking으로 감싸져 있기 때문에 this도 필요없음
            delay(1000L)
            println("World!!!!!!!!")
        }
        println("Hello")
    }
```
 ```Kotlin
Hello
World!!!!
World!!!!!!!!
```
만약 코루틴으로 동작하는 함수를 만들고 싶을때 suspend 를 붙여줌으로서 만들 수 있다.
 ```Kotlin
     fun main6() = runBlocking {
        launch {
            myWorld()
        }
        println("Hello")
    }
    suspend fun myWorld(){
        delay(1000L)
        println("World")
    }
```
다음은 코루틴이 가벼운 것을 증명하는 예제이다.
```Kotlin
    fun main7() = runBlocking {
        repeat(100_000) { // 100000개의 코루틴 점찍기
            launch {
                delay(5000L)
                print(".")
            }
        }
    }
```
아래의 예시는 Thread에서 하기때문에 버벅이는 현상이 발생한다.
```Kotlin
   fun main8() = runBlocking {
        repeat(100_000) { // 100000개의 코루틴 점찍기
            Thread {
                Thread.sleep(5000L)
                print(".")
            }
        }
    }
```
다음 예시는 메인 함수가 끝나도 코루틴이 살아 있는지 확인하는 예시이다. 만약 살아 있다면 코루틴을 전부 찍겠지만 결과는 코루틴은 프로세스가 끝날시 끝나게 된다.
```Kotlin
   fun main9() = runBlocking {
       GlobalScope.launch {
           repeat(1000){ i ->
               println("I'm sleeping $i ...")
               delay(500L)
           }
       }
        delay(1300L)
    }
```
```Kotlin
   I'm sleeping 0 ...
   I'm sleeping 1 ...
   I'm sleeping 2 ...
```
***

 ### Cancellation and Timeouts
 - Job
    - cancel()
 - Cancellation and Timeouts
    - suspend functions 을 호출하기
    - 명시적으로 isActive상태를 체크하기
 - Timeout
    - withTimeout
    - withTimeoutOrNull
 ```Kotlin
   fun main1() = runBlocking {
       val job = launch { //1000번을 돌면서 0.5초마다 출력
           repeat(1000) { i ->
               println("job: I'm sleeping $i ...")
               delay(500L)
           }
       }
       delay(1300L)
       println("main : I'm tired of waiting!")
       job.cancel()
       job.join()
       println("main : now Ican quit.")
    }
```
```Kotlin
   job: I'm sleeping 0 ...
   job: I'm sleeping 1 ...
   job: I'm sleeping 2 ...
   main : I'm tired of waiting!
   main : now Ican quit.
```
다음 예제는 코루틴이 취소에 협조적으로 작성되어야하는 예시이다.

목표는 0,1,2까지만 하고 싶지만 취소되지 않고 다 실행된다. 이유는 delay라는 suspend functions이 불러와지지 않았기 때문이다.


```Kotlin
   fun main2() = runBlocking {
        val startTime = System.currentTimeMillis()
        val job = launch(Dispatchers.Default) {
            var nextPrintTime = startTime
            var i = 0
            while (i < 5) { 
                if (System.currentTimeMillis() >= nextPrintTime) {
                    println("job: I'm sleeping ${i++} ...")
                    nextPrintTime += 500L
                }
            }
        }
        delay(1300L) // delay a bit
        println("main: I'm tired of waiting!")
        job.cancelAndJoin() // cancel과 join이 순차적으로 실행되는 함수
        println("main: Now I can quit.")
    }
```
```Kotlin
   job: I'm sleeping 0 ...
   job: I'm sleeping 1 ...
   job: I'm sleeping 2 ...
   job: I'm sleeping 3 ...
   main: I'm tired of waiting!
   job: I'm sleeping 4 ...
   main: Now I can quit.
```
해당 코드는 아래와 같이 변경해주어야 한다. (delay(1L) 이나 yield() 추가)

아래 코드가 취소가 가능한 이유는 yield()문을 지나가면서 내부적으로 Exception을 터트리면서 취소가 된다.
```Kotlin
   fun main3() = runBlocking {
        val startTime = System.currentTimeMillis()
        val job = launch(Dispatchers.Default) {
            var nextPrintTime = startTime
            var i = 0
            while (i < 5) {
                if (System.currentTimeMillis() >= nextPrintTime) {
                    //delay(1L)
                    yield()
                    println("job: I'm sleeping ${i++} ...")
                    nextPrintTime += 500L
                }
            }
        }
        delay(1300L) // delay a bit
        println("main: I'm tired of waiting!")
        job.cancelAndJoin() // cancel과 join이 순차적으로 실행되는 함수
        println("main: Now I can quit.")
    }
```
```Kotlin
    fun main3() = runBlocking {
        val startTime = System.currentTimeMillis()
        val job = launch(Dispatchers.Default) {
            try{
                var nextPrintTime = startTime
                var i = 0
                while (i < 5) {
                    if (System.currentTimeMillis() >= nextPrintTime) {
                        delay(1L)
                        println("job: I'm sleeping ${i++} ...")
                        nextPrintTime += 500L
                    }
                }
            }catch (e: Exception){
                println("Exception $e")
            }

        }
        delay(1300L) // delay a bit
        println("main: I'm tired of waiting!")
        job.cancelAndJoin() // cancel과 join이 순차적으로 실행되는 함수
        println("main: Now I can quit.")
    }
``` 
 ```Kotlin
job: I'm sleeping 0 ...
job: I'm sleeping 1 ...
job: I'm sleeping 2 ...
main: I'm tired of waiting!
Exception kotlinx.coroutines.JobCancellationException: StandaloneCoroutine was cancelled; job="coroutine#2":StandaloneCoroutine{Cancelling}@48a581ea
main: Now I can quit.
```
 다음은 상태를 주기적으로 체크해주는 예시이다.
 
 isActive의 정체는 CoroutineScope의 확장 프로퍼티이다. 내부를 보면 코루틴의 Job이 실제 종료되었는지 확인하는 방법이다.
```Kotlin
  fun main4() = runBlocking {
        val startTime = System.currentTimeMillis()
        val job = launch(Dispatchers.Default) {
            var nextPrintTime = startTime
            var i = 0
            while (isActive) { // cancellable computation loop
                // print a message twice a second
                if (System.currentTimeMillis() >= nextPrintTime) {
                    println("job: I'm sleeping ${i++} ...")
                    nextPrintTime += 500L
                }
            }
        }
        delay(1300L) // delay a bit
        println("main: I'm tired of waiting!")
        job.cancelAndJoin() // cancels the job and waits for its completion
        println("main: Now I can quit.")
    }
```

***

### Composing Suspending Functions

+ Async to sequential
   + Sequential by default
   + the Dream Code on Android
+ async
   + concurrent using async
   + Lazily started async
+ Structured concurrency
   + Async-style functions (strongly discouraged)
   + Structured concurrency with async

다음 예시는 헤비한 두 함수를 비동기로 처리할때 순차적으로 실행 하고싶을때 어떻게 할 수 있는지 보여주는 예시이다.

결론은 그냥 코드짜듯이 아래에 작성하기만 해도 순차적으로 작성됨을 알 수 있다.
```Kotlin
    fun main1() = runBlocking {
        val time = measureTimeMillis {
            val one = doSomethingUsefulOne()
            val two = doSomethingUsefulTwo()
            println("The answer is ${one + two}")
        }
        println("Completed in $time ms")
    }

    suspend fun doSomethingUsefulOne(): Int {
        println("doSomethingUsefulOne")
        delay(4000L) // 해당자리는 서버와 같은 해비한 연산을 하는 자리
        println("onefinish")
        return 13
    }

    suspend fun doSomethingUsefulTwo(): Int {
        println("doSomethingUsefulTwo")
        delay(1000L)
        println("Twofinish")
        return 29
    }
```
```Kotlin
doSomethingUsefulOne
onefinish
doSomethingUsefulTwo
Twofinish
The answer is 42
Completed in 5052 ms
```
하지만 만약 독립적으로 실행순서와 상관없이 실행하고 싶다면 async를 감싸 독립적인 코루틴을 만들어준다.

하지만 값을 프린트 할때에는 one.await()와 같은 함수로 기다렸다가 값을 출력한다.

async는 job을 반환하는데 이떄 job의 함수를 이용할 수 있기에 자유자재로 동시하는것과 순차적인것이 둘 다 가능해진다.
```Kotlin
    fun main1() = runBlocking {
        val time = measureTimeMillis {
            val one = async{doSomethingUsefulOne()}
            val two = async{doSomethingUsefulTwo()}
            println("The answer is ${one.await() + two.await()}")
        }
        println("Completed in $time ms")
    }

    suspend fun doSomethingUsefulOne(): Int {
        println("doSomethingUsefulOne")
        delay(4000L) // 해당자리는 서버와 같은 해비한 연산을 하는 자리
        println("onefinish")
        return 13
    }

    suspend fun doSomethingUsefulTwo(): Int {
        println("doSomethingUsefulTwo")
        delay(1000L)
        println("Twofinish")
        return 29
    }
```
```Kotlin
doSomethingUsefulOne
doSomethingUsefulTwo
Twofinish
onefinish
The answer is 42
Completed in 4065 ms
```
다음은 start함수와 Lazy함수의 사용이다.

Lazy함수는 시작하는것을 기다려주고 start함수를 만나면 코루틴을 시작한다.

```Kotlin
    fun main1() = runBlocking {
        val time = measureTimeMillis {
            val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
            val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
            one.start() // one을 시작한다.
            two.start() // two를 시작한다.
            println("The answer is ${one.await() + two.await()}")
        }
        println("Completed in $time ms")
    }

    suspend fun doSomethingUsefulOne(): Int {
        delay(1000L) // pretend we are doing something useful here
        return 13
    }

    suspend fun doSomethingUsefulTwo(): Int {
        delay(1000L) // pretend we are doing something useful here, too
        return 29
    }
```
```Kotlin
The answer is 42
Completed in 1071 ms
```
위와 같이 start를 만나 동시에 시작하므로 1초가 걸리는것을 볼 수 있다.

하지만 만약 start가 없다면 2초가 걸리는 것 을 알 수있다.

왜냐하면 awit문은 함수의 결과가 끝날때까지 기다리는 함수이기 때문이다.

```Kotlin
The answer is 42
Completed in 2082 ms
```
다음 예시는 함수로 자체적으로 만들어줬을때의 문제를 보여주는 예시이다.

아래 예시에서는 try안에서 코루틴함수인 one,two가 실행 되고 나서 exception이 터진다면 멈춰야 하는데 계속 실행되는것을 보여준다.

```Kotlin
    fun main2() {
        try {
            val time = measureTimeMillis {
                val one = SomethingUsefulOneAsync();
                val two = SomethingUsefulTwosync();
                println("my execeptions")
                throw Exception("my execeptions")

                runBlocking {
                    println("The answer is ${one.await()+two.await()}")
                }
            }
            println("Completed in $time ms")
        }catch (e : Exception){
        }
        runBlocking {
            delay(100000)
        }

    }
    fun SomethingUsefulOneAsync() = GlobalScope.async {
        println("start, SomethingUsefulOneAsync")
        val res=doSomethingUsefulOne()
        println("end, SomethingUsefulOneAsync")
        res
    }
    fun SomethingUsefulTwosync() = GlobalScope.async {
        println("start, SomethingUsefulTwosync")
        val res=doSomethingUsefulOne()
        println("end, SomethingUsefulTwosync")
        res
    }
    suspend fun doSomethingUsefulOne(): Int {
        delay(1000L) // pretend we are doing something useful here
        return 13
    }

    suspend fun doSomethingUsefulTwo(): Int {
        delay(1000L) // pretend we are doing something useful here, too
        return 29
    }
```
```Kotlin
my execeptions
start, SomethingUsefulTwosync
start, SomethingUsefulOneAsync
end, SomethingUsefulTwosync
end, SomethingUsefulOneAsync
```
이에 대한 해결책은 다음과 같다.
```Kotlin
    fun main3() = runBlocking{
        val time = measureTimeMillis {
            println("The answer is ${concurrentSum()}")
        }
        println("Completed in $time ms")
    }
    suspend fun concurrentSum() : Int = coroutineScope {
        val one = async{ doSomethingUsefulOne() }
        val two = async{ doSomethingUsefulTwo() }
        one.await()+two.await()
    }
    suspend fun doSomethingUsefulOne(): Int {
        delay(1000L) // pretend we are doing something useful here
        return 13
    }

    suspend fun doSomethingUsefulTwo(): Int {
        delay(1000L) // pretend we are doing something useful here, too
        return 29
    }
```

***
### Coroutines under ths hood
이번 차트에서는 취소와 중단, 자바에서는 가능하지 않지만 코틀린에서는 가능한가 등 코루틴의 동작원리를 간단히 알아보려한다.

[KotlinConf 2017](https://www.youtube.com/watch?v=YrrUCSi72E8)

에서 보면 코루틴은 마법같이 느껴지지만 마법은 없다라고 알려준다.

간단히 말해서 Continuation Passing Style(CPS)로 만들어지게 되는데 이는 Continuations 이란 변수가 추가되면서 매번 모든 코드가 스위치에 case를 돌듯 확인을 하면서 가는 방식이다.
Continuations은 Calback 을 넘겨주는 함수이다.

### Coroutine Context and Dispatchers
각각에는 디스패처를 변경해줄 수 있는데 결과값과 같이 실행하고 있는 Thread가 다 다르다.
```Kotlin
    fun main1() = runBlocking<Unit> {
        launch { // context of the parent, main runBlocking coroutine
            println("main runBlocking      : I'm working in thread ${Thread.currentThread().name}")
        }
        launch(Dispatchers.Unconfined) { // not confined -- will work with main thread
            println("Unconfined            : I'm working in thread ${Thread.currentThread().name}")
        }
        launch(Dispatchers.Default) { // will get dispatched to DefaultDispatcher
            println("Default               : I'm working in thread ${Thread.currentThread().name}")
        }
        launch(newSingleThreadContext("MyOwnThread")) { // will get its own new thread
            println("newSingleThreadContext: I'm working in thread ${Thread.currentThread().name}")
        }
    }
```
```Kotlin
Unconfined            : I'm working in thread main @coroutine#3
Default               : I'm working in thread DefaultDispatcher-worker-1 @coroutine#4
newSingleThreadContext: I'm working in thread MyOwnThread @coroutine#5
main runBlocking      : I'm working in thread main @coroutine#2
```
다음은 코루틴 스코프의 예시이다.

예를들면 Activity에서 코루틴을 사용하다가 Activity가 종료됨에 따라 같이 종료 하고 싶을때 매번 종료하기보다는 Context를 연결하여 실행시켜준다.

이처럼 안드로이드에서는 코루틴 스코트를 이용하여 생명주기에 맞추어 개발하면 된다.

```Kotlin
class Coroutine_Context_and_Dispatchers {
    private val mainScope = MainScope()

    fun destroy() {
        mainScope.cancel()
    }
    fun doSomething() {
        // launch ten coroutines for a demo, each working for a different time
        repeat(10) { i ->
            mainScope.launch {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, ... etc
                println("Coroutine $i is done")
            }
        }
    }
} // class Activity ends
fun main() = runBlocking {
    val activity = Coroutine_Context_and_Dispatchers()
    activity.doSomething() // run test function
    println("Launched coroutines")
    delay(500L) // delay for half a second
    println("Destroying activity!")
    activity.destroy() // cancels all coroutines
    delay(3000) // visually confirm that they don't work
}
```
