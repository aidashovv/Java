[Java](README.md)

# Многопоточноть
  - [Что такое многопоточность?](#что-такое-многопоточность)
  - [Как открыть новый поток?](#как-открыть-новый-поток)
  - [Ключевое слово volatile](#ключевое-словое-volatile)
  - [Ключевое слово synchronized](#ключевое-словое-synchronized-без-явного-указания)

## Что такое _многопоточность_?
__Многопоточность__ - механизм, который помогает выполнять код || с другим. 

Если __процессор 1 ядерный__ или Java посчитает, что __распределять потоки на ядра нет смысла__, то в Java многопоточность представлена в виде __виртуальной ||-ти__, то есть задания не выполняются ||, а просто процессор быстро переключается между потоками. Многопоточнось в Java можно сравнить с чтением n кол-ва книг одновременно. __В остальных случаях__, Java старается расспределить задачи между потоками.

__Основная цель__: реализация специального функционала (обработка сложных задач в фоновом режиме, веб-сервис).

[к оглавлению](#Многопоточноть)

## Как открыть новый поток?
__I способ__:
```java
public class App {
    public static void main( String[] args )
    {
        MyThread myThread1 = new MyThread(); // запускаем 1 поток
        myThread1.start();

        MyThread myThread2 = new MyThread(); // запускаем 2 поток
        myThread2.start();
    }
}

class MyThread extends Thread { // Thread - класс, который лежит в пакете java.lang и доступен без import
    public void run() { // переопрделяем run (не определен в Thread) и описываем тот код, который хотиим выполнить
        for (int i = 0; i < 100; i++) {
            System.out.println( "Hello World from new thread!" + i );
        }
    }
}
```

__II способ__:
```java
public class App {
    public static void main( String[] args )
    {
        Thread thread = new Thread(new Runner());
        thread.start();
    }
}

class Runner implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println( "Hello World from new thread!" + i );
        }
    }
}
```
`В данных случаях вывод - спонтанный, так как нет никакой синхронизации и каждый из потоков борется за процессорное время.`

[к оглавлению](#Многопоточноть)

## Ключевое слово volatile
Применяется в том случае, если один из потоков записывает в переменную (main), а другой(ие) читает(ют) из переменной (myThread), чтобы избежать возможной проблемы с `Cache Coherency`.
```java
public class App
{
    public static void main( String[] args )
    {
        MyThread MyThread = new MyThread();
        MyThread.start();

        Scanner scanner = new Scanner(System.in);
        scanner.nextLine();

        MyThread.offThread();
    }
}

class MyThread extends Thread {
    private volatile boolean running = true; // volatile говорит о том, что 'running' не кэшируется в кэш ядра

    @Override
    public void run() {
        while (running) {
            System.out.println("Hello World");
            try {
                Thread.sleep(500); // закрывает поток на 0.5 секунд
            } catch (InterruptedException e) { // для sleep
                e.printStackTrace();
            }
        }
    }

    public void offThread() {
        running = false;
    }
}
```

Проблема __Cache Coherency__:

У каждого ядра процессора есть свой кэш (быстрая и небольшая память). Там находятся данные, которые нужны ядру здесь и сейчас.

Если вернуться к коду выше, то там была переменная `running`, которая должна была измениться для прекращения потока. Но кэш одного из ядер процессора мог загрузить себе исходное значение `= true` и другой поток не смог бы никак повлиять на него.

В это и заключается проблема "`Когенертности (Свопадения) Кэшей`".

![ ](images/Multithreading/cache_coherency.png)

[к оглавлению](#Многопоточноть)

## Ключевое слово synchronized без явного указания
Применяется в том случае, если один из потоков читает из переменной, а другие записывают в переменную. Если точнее, то решает проблему `Race Contidion`.

Без явного указания мы синхронизируемся на объекте `this` (тут оно = new App()).

Ниже, вывод 20.000 будет не стабилен, так как потоки не синхронизированы. Не атомарность операции `++` меняет значение k, потому что после смены потока может случится перезапись k в меньшую сторону, (к примеру, 101), когда другой потом на прошлой шаге записал туда значение явно большее (к примеру, 107), потому что он получил больше процессорного времени.
```java
public class App
{
    private int k;
    public static void main( String[] args ) throws InterruptedException { // exception для .join в doWork()
        App app = new App();
        app.doWork();
    }

    public void doWork() throws InterruptedException { // exception для .join
        Thread thread1 = new Thread(new Runnable() { // 1-й поток, который делвет +1 к переменной k 
            public void run() {
                for (int i = 0; i < 10000; i++) {
                    k++; // k = k + 1 - не атомарная (делимая на части) операция
                }
            }
        });

        Thread thread2 = new Thread(new Runnable() { // 2-й поток, который делвет +1 к переменной k 
            public void run() {
                for (int i = 0; i < 10000; i++) {
                    k++; // k = k + 1 - не атомарная (делимая на части) операция
                }
            }
        });

        thread1.start();
        thread2.start();

        thread1.join(); // .join говорит main о том, что ждем окончания 1-ого потока
        thread2.join(); // ждем окончания 2-ого потока

        System.out.println(k); // и только потом выводи в консоль значения переменной
    }
}
```

Исправленный код c использованием synchronized, который предотвращает одновременное исполнение инкремента. Пока один поток пользуется методом, другой ждет разрешения:
```java
public class App
{
    private int k;
    public static void main( String[] args ) throws InterruptedException {
        App app = new App();
        app.doWork(); 
    }

    // выделили отдельный метод
    // так как synchronized применяется исключительно с методами
    // в отличие от volatile
    public synchronized void increment() {
        k++;
    }

    public void doWork() throws InterruptedException {
        Thread thread1 = new Thread(new Runnable() {
            public void run() {
                for (int i = 0; i < 10000; i++) {
                    increment();
                }
            }
        });

        Thread thread2 = new Thread(new Runnable() {
            public void run() {
                for (int i = 0; i < 10000; i++) {
                    increment();
                }
            }
        });

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println(k);
    }
}
```

__Как устроено synchronized?__ 

В Java к каждому созданному объекту присвается `сущность` (monitor lock - дает понять Java, что в данный момент поток взаимодействует с объектом или помогает получить доступ к полям/методам объекта), которая `в один момент времени может быть только у одного потока`. И synchronized как раз таки использует данную особенность Java-объектов.

[к оглавлению](#Многопоточноть)


