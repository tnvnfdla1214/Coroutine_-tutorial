# 코루틴 사용해보기
해당 예제는 프로젝트들을 사용해 보았으나 [코틀린 공식문서](https://kotlinlang.org/docs/coroutines-guide.html#table-of-contents)에 있는 코루틴 튜토리얼을 실습해보면서 파트를 작성하는게 보다 나을것이여서 예제위주로 작성하였습니다.
***
 ## :wrench: 프로젝트 사용 사례
<img src="https://user-images.githubusercontent.com/48902047/134470411-a786e31b-a658-455a-af1b-8854f22d7369.jpg" width="20%" height="20%"></img>
<img src="https://user-images.githubusercontent.com/48902047/134470456-bf0f37e3-87a8-4d3f-b94d-14d6147691c3.jpg" width="20%" height="20%"></img>
<img src="https://user-images.githubusercontent.com/48902047/134470695-77c7be06-b2bf-4e84-971f-013b63e42d82.jpg" width="20%" height="20%"></img>
<img src="https://user-images.githubusercontent.com/48902047/134470599-1e025c6d-c5ab-4825-be2b-58c838b36465.jpg" width="20%" height="20%"></img>

 + [코루틴 깃허브 레파지토리 앱](https://github.com/tnvnfdla1214/github_repository)
 ***
 ## :lollipop: 예제
 본예제 실습 환경을 안드로이드 스튜디오 Junit4에서 작성되었습니다.
 해당 예제들은 코루틴 깃허브 레파지토리 앱의 테스트 레파지토리에 있으니 확인 가능합니다.
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
