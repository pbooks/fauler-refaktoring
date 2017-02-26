# Замена параметра явными методами (Replace Parameter with Explicit Methods)

Есть метод, выполняющий разный код в зависимости от значения параметра перечислимого типа.

_Создайте отдельный метод для каждого значения параметра._

```java
void setValue (String name, int value) {
    if (name.equals("height")) {
        _height = value;
        return;
    }
    if (name.equals("width")) {
        _width = value;
        return;
    }
    
    Assert.shouldNeverReachHere();
}
```

![](images/arrow.jpg)

```java
void setHeight(int arg) {
    _height = arg;
}
void setWidth (int arg) {
    _width = arg;
}
```

## Мотивировка

«Замена параметра явными методами» ([Replace Parameter with Explicit Methods](Replace-Parameter-with-Explicit-Methods.md)) является рефакторингом, обратным по отношению к «Параметризации метода» ([Parameterize Method](Parameterize-Method.md)). Типичная ситуация для ее применения возникает, когда есть параметр с дискретными значениями, которые проверяются в условном операторе, и в зависимости от результатов проверки выполняется разный код. Вызывающий должен решить, что ему надо сделать, и установить для параметра соответствующее значение, поэтому можно создать различные методы и избавиться от условного оператора. При этом удается избежать условного поведения и получить контроль на этапе компиляции. Кроме того, интерфейс становится более прозрачным. Если используется параметр, то программисту, применяющему метод, приходится не только рассматривать имеющиеся в классе методы, но и определять для параметра правильное значение. Последнее часто плохо документировано.

Прозрачность ясного интерфейса может быть достаточным результатом, даже если проверка на этапе компиляции не приносит пользы. Код `Switch.beOn()` значительно яснее, чем `Switch.setState(true)`, даже если все действие заключается в установке внутреннего логического поля.

Не стоит применять «Замену параметра явными методами» ([Replace Parameter with Explicit Methods](Replace-Parameter-with-Explicit-Methods.md)), если значения параметра могут изменяться в значительной мере. В такой ситуации, когда переданный параметр просто присваивается полю, применяйте простой метод установки значения. Если требуется условное поведение, лучше применить «Замену условного оператора полиморфизмом» ([Replace Conditional with Polymorphism](Replace-Conditional-with-Polymorphism.md)).

## Техника

* Создайте явный метод для каждого значения параметра.
* Для каждой ветви условного оператора вызовите соответствующий новый метод.
* Выполняйте компиляцию и тестирование после изменения каждой ветви.
* Замените каждый вызов условного метода обращением к соответствующему новому методу.
* Выполните компиляцию и тестирование.
* После изменения всех вызовов удалите условный метод.

## Пример

Я предпочитаю создавать подкласс класса служащего, исходя из переданного параметра, часто в результате «Замены конструктора фабричным методом» ([Replace Constructor with Factory Method](Replace-Constructor-with-Factory-Method.md)):

```java
static final int ENGINEER = 0;
static final int SALESMAN = 1;
static final int MANAGER = 2;

static Employee create(int type) {
    switch (type) {
        case ENGINEER:
            return new Engineer();
        case SALESMAN:
            return new Salesman();
        case MANAGER:
            return new Manager();
        default:
            throw new IllegalArgumentException("Incorrect type code value");
    }
}
```

Поскольку это фабричный метод, я не могу воспользоваться «Заменой условного оператора полиморфизмом» ([Replace Conditional with Polymorphism](Replace-Conditional-with-Polymorphism.md)), т. к. я еще не создал объект. Я предполагаю, что новых классов будет немного, поэтому есть смысл в явных интерфейсах. Сначала создаю новые методы:

```java
static Employee createEngineer() {
    return new Engineer();
}

static Employee createSalesman() {
    return new Salesman();
}

static Employee createManager() {
    return new Manager();
}
```

Поочередно заменяю ветви операторов вызовами явных методов:

```java
static Employee create(int type) {
    switch (type) {
        case ENGINEER:
            return Employee.createEngineer(); // !!!
        case SALESMAN:
            return new Salesman();
        case MANAGER:
            return new Manager();
        default:
            throw new IllegalArgumentException("Incorrect type code value");
    }
}
```

Выполняю компиляцию и тестирование после изменения каждой ветви, пока не заменю их все:

```java
static Employee create(int type) {
    switch (type) {
        case ENGINEER:
            return Employee.createEngineer();
        case SALESMAN:
            return Employee.createSalesman();
        case MANAGER:
            return Employee.createManager();
        default:
            throw new IllegalArgumentException("Incorrect type code value");
    }
}
```

Теперь я перехожу к местам вызова старого метода `create`. Код вида

```java
Employee kent = Employee.create(ENGINEER);
```

я заменяю таким:

```java
Employee kent = Employee.createEngineer();
```

Проделав это со всеми местами вызова `create`, я могу удалить метод `create`. Возможно, удастся также избавиться от констант.

