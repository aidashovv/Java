[Java](README.md)

# Spring Core
  - [Что такое фреймворк?](#что-такое-фреймворк)
  - [Что такое Inversion of control?](#что-такое-inversion-of-control)
  - [Как реализовать Inversion of control?](#как-реализовать-inversion-of-control)
  - [Bean lifecycle](#bean-lifecycle)
  - [Методы bean'ов](#методы-beanов)
  - [Bean scope](#bean-scope)
  - [Конфигурация с помощью аннотаций](#конфигурация-с-помощью-аннотаций)


## Что такое фреймворк?
Это набор библиотек со своими правилами, к которым мы подстраиваемся. `Spring Framework` можно изменять под себя.

[к оглавлению](#springcore)

## Что такое Inversion of control?
__Inversion of control__ (Инверсия управлений) - это идея разработки ПО, при которой управление объектами или частями программы передается в контейнер или платформу.

__Aplicaition Context__ - интерфейс над IoC.

IoC даёт следующие преимущества:
- разделение выполнения задачи и ее реализации
- упрощение переключения между различными реализациями
- большая модульность программы
- упрощение тестирования программы за счет изоляции компонента или имитации его зависимостей, а также разрешения компонентам взаимодействовать через контракты

[к оглавлению](#springcore)

## Как реализовать Inversion of control?
+ Поиск завивимостей (Это рудимент, который useless).
+ Внедрение зависимостей (Dependecy Injection) - использовать этот вариант и он стоит по default'у.

```java
@Component(scope="?") // из java-объекта MyRepository создает Spring Bean (базово создается singleton'ом)
public class MyRepository {...}
```

Все сервисы превращать в Spring Bean - имеет смысл.

[к оглавлению](#springcore)

@RequiredArgs необходим для Spring + во всех переменных ставим final
Любой Bean, который есть в Bean

## Bean lifecycle
1) Запускается Spring контейнер (Spring Application Context).
2) Создание bean'a.
3) В bean внедряются зависимости (Dependecy Injection).
4) Вызывается указанный init-method (если указывали), где указывается та логика, которая должна выполниться при создании bean'а.
5) Bean готов к использованию и передан пользователю.
6) При завершении Spring приложения вызывается указанный destroy-method (если указывали), где указывается та логика, которая должна выполниться при удалении bean'а.

[к оглавлению](#springcore)

## Методы bean'ов
+ __init-method__ - метод, который запускается при инициализации bean'a. Отвечает за инициализацию ресурсов, обращение к внешним файлам, запуск БД.
+ __destroy-method__ - метод, который запускается при удалении bean'а. Отвечает за очищение ресурсов, закрытие потоков ввода/вывода, закрытие доступа к БД.
+ __factory-method__ - метод, который можно определить, если объекты класса создаются фабричным методом.

__Тонкости init и destroy методов:__
+ Модификатор доступа любой
+ Тип возвращаемого значения любой, но чаще всего `void`
+ Эти методы `не принимают` на вход какие-либо аргументы.

[к оглавлению](#springcore)

## Bean scope
+ Singleton (стоит по default'у) - используется, когда наш bean stateless.
+ Prototype - каждый раз создаёт новый объект при вывозе `context.getBean()`. Также у bean'ов с таким scope'ом не вызывается destroy-method.

[к оглавлению](#springcore)

## Конфигурация с помощью аннотаций
При такой конфигурации Spring __сканирует__ все классы приложения, __находит__ все классы с аннотацией @Component и __автоматически создает__ bean'ы из этих классов.


__@Component__ - помечает будущий bean. Также можно указать __id__ для создаваемого bean'а или Spring автоматически присвоит ему id с названием класса в camel case.

__@Autowired__ - позволяет делигировать Spring'у внедрение зависимостей. Можно использовать на полях (даже при отсутствие сеттера и конструктора с помощью `Java Reflection API`), сеттерах и конструкторах.

__@Qualifier__ - позволяет уточнить @Autowired, какой bean нам конкретно нужен:
```java
@Autowired
@Qualifier("rockMusic") \\ id того bean'а, который мы хотим внедрить
private Music music; \\ Music - интерфейс

\\ при уточнении в конструкторе нужно пользоваться
\\ данным синтаксисом
@Autowired
public MusicPlayer(@Qualifier("rockMusic") Music music1,
                   @Qualifier("popMusic") Music music2) {
    this.music1 = music1;
    this.music2 = music2;
}
```

__Возможные ошибки при DI с помощью @Autowired:__
- Если не находится ни одного bean'а
- Если несколько bean'ов подходят - неоднозначность (проблема решается @Qualifier)

__@Scope__ -

[к оглавлению](#springcore)

Аннотации - для своих классов

Java config - для всего остальное

Groovy - если хотим что-то вынести без перекомпиляции

1 этап:
После обхода всех конфигураций Spring создал BeanDefinitionMap (name BeanDefinition'а и метаинформация), в которой лежат BeanDefinition для каждого bean'а. В BeanDefinition хранится вся метаинформация по bean'у.

Spring управляет руками приложения => помечать bean'ами сущности нет смысла. Нужно для репо, сервисов и т.д.

2 этап: Дополнение BeanDefinition'ов.
3 этап: FactoryBean - регистрация фабрик для создания конкретных bean'ов.

4 этап: Создание экзампляров bean'ов.
BeanFactory занимается созданием экземпляров bean'ов.

5 этап: BeanPostProcessor - обрабатывает bean'ы и обязан их же вернуть. И появляется ApplicationContext.


