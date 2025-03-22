[Java](README.md)

# Hibernate
  - [Что такое _Hibernate_?](#что-такое-hibernate)
  - [Какие основные сущности в Hibernate?](#какие-основные-сущности-в-hibernate)
  - [Как правильно настроить файл _properties_?](#как-правильно-настроить-файл-properties)
  - [Какие аннотации необходимы для работы с Hibernate?](#какие-аннотации-необходимы-для-работы-с-hibernate)
  - [Полезные аннтоации](#полезные-аннотации)

## Что такое Hibernate?
__Hibernate__ - ORM (Object-Relational Mapping), которая автоматизирует отображение Java объектов в строки в таблице реаляционной БД. 

`Под капотом у него JDBC API`

[к оглавлению](#Hibernate)

## Какие основные сущности в Hibernate?
+ __Session__ - объект для взаимодействия в Hibernate. Когда мы хотим произвести какие-либо действия с БД через  Hibernate - получаем _сессию_. А объект __Session__ получаем из объекта __SessionFactory__ (паттерн Фабрика):
```java
SessionFactory sessionFactory = configuration.BuildSessionFactory();
Session session = sessionFactory.getCurrentSession();
``` 

+ __Transaction__ - единица работы с БД.
```java
session.beginTransaction(); // Начали транзакцию
...
...
session.save(person); // работа с БД
session.getTransaction().commit(); // Завершили транзакцию
// В случае неудачной транзакции, можем сделать rollback.
```

__Зачем нам помещать несколько операций в 1 транзакцию?__
Это нужно для поддержания согласованности в таблицах. 

__Например__: Мы хотим запретить доступ к таблице, пока БД обрабатывает наш запрос на изменение таблицы. Или мы хотим ограничить доступ дргуих сессий, потоков на обработку той сущности, которая в текущий момент изменяется.

Данная проблема называется __Race Condition__ и для ее решения можно синхронизировать потоки (обычно с помощью блокировок). Будут использоваться __Тransactional Isolation Levels__ (уровни изоляции транзакций).

[к оглавлению](#Hibernate)

## Как правильно настроить файл _properties_?
```java
// --- Data Source configuration ---

// hibernate.driver_class - класс jdbc, который мы указываем для jdbc driver
hibernate.driver_class=org.postgresql.Driver
hibernate.connection.url=jdbc:postgresql://localhost:5432/postgres
hibernate.connection.username=postgres
hibernate.connection.password=0000

// --- Hibernate configuration ---

// Указываем диалект (определенную реаляционную БД)
hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
// ??? (пока что)
hibernate.current_session_context_class=thread
```

[к оглавлению](#Hibernate)

## Какие аннотации необходимы для работы с Hibernate?
+ @Entity - помечает те классы, которые связаны с БД.
Все классы, помеченные @Entity, обязаны иметь __пустой конструктор__ и содержать хотя бы 1 поле, помеченное __@Id__.
+ @Table - указание таблицы, которая предназначена для определенной сущности.
+ @Column - позволяет сопоставить каждому полю класса соотвествующий столбец в таблице.
+ @Id - указывает на поле, которое является первичным ключом в таблице.
 
```java
import javax.persistence.*;

@Entity
@Table(name = "friends")
public class Friend {

    @Id
    @Column(name = "friend_login")
    private String friendLogin;

    public Friend() {}

    public Friend(String inputFriendLogin) {
        friendLogin = inputFriendLogin;
    }

    ...
}
```

[к оглавлению](#Hibernate)

## Полезные аннотации

[к оглавлению](#Hibernate)
