# Замена параметра вызовом метода (Replace Parameter with Method)

Объект вызывает метод, а затем передает полученный результат в качестве параметра метода. Получатель значения тоже может вызывать этот метод.

_Уберите параметр и заставьте получателя вызывать этот метод._

```java
int basePrice = _quantity * _itemPrice;
discountLevel = getDiscountLevel();
double finalPrice = discountedPrice (basePrice, discountLevel);
```

![](images/arrow.jpg)

```java
int basePrice = _quantity * _itemPrice;
double finalPrice = discountedPrice (basePrice);
```

## Мотивировка

Если метод может получить передаваемое в качестве параметра значение другим способом, он так и должен поступить. Длинные списки параметров трудны для понимания, и их следует по возможности сокращать.

Один из способов сократить список параметров заключается в том, чтобы посмотреть, не может ли рассматриваемый метод получить необходимые параметры другим путем. Если объект вызывает свой метод и вычисление для параметра не обращается к каким либо параметрам вызывающего метода, то должна быть возможность удалить параметр путем превращения вычисления в собственный метод. Это верно и тогда, когда вы вызываете метод другого объекта, в котором есть ссылка на вызывающий объект.

Нельзя удалить параметр, если вычисление зависит от параметра в вызывающем методе, потому что этот параметр может быть различным в каждом вызове (если, конечно, не заменить его методом). Нельзя также удалять параметр, если у получателя нет ссылки на отправителя и вы не хотите предоставить ему ее.

Иногда параметр присутствует в расчете на параметризацию метода в будущем. В этом случае я все равно избавился бы от него. Займитесь параметризацией тогда, когда это вам потребуется; может оказаться, что необходимый параметр вообще не найдется. Исключение из этого правила я бы сделал только тогда, когда результирующие изменения в интерфейсе могут иметь тяжелые последствия для всей программы, например потребуют длительной компиляции или изменения большого объема существующего кода. Если это тревожит вас, оцените, насколько больших усилий потребует такое изменение. Следует также разобраться, нельзя ли сократить зависимости, из за которых это изменение столь затруднительно. Устойчивые интерфейсы – это благо, но не надо консервировать плохие интерфейсы.

## Техника

* При необходимости выделите расчет параметра в метод.
* Замените ссылки на параметр в телах методов ссылками на метод.
* Выполняйте компиляцию и тестирование после каждой замены.
* Примените к параметру «Удаление параметра» ([Remove Parameter](Remove-Parameter.md)).

## Пример

Еще один маловероятный вариант скидки для заказов выглядит так:

```java
public double getPrice() {
    int basePrice = _quantity * _itemPrice;
    int discountLevel;
    if (_quantity > 100) discountLevel = 2;
    else discountLevel = 1;
    double finalPrice = discountedPrice (basePrice, discountLevel);
    return finalPrice;
}

private double discountedPrice (int basePrice, int discountLevel) {
    if (discountLevel == 2) return basePrice * 0.1;
    else return basePrice * 0.05;
}
```

Я могу начать с выделения расчета категории скидки `discountLevel`:

```java
public double getPrice() {
    int basePrice = _quantity * _itemPrice;
    int discountLevel = getDiscountLevel(); // !!!
    double finalPrice = discountedPrice (basePrice, discountLevel);
    return finalPrice;
}

private int getDiscountLevel() {
    if (_quantity > 100) return 2;
    else return 1;
}
```

Затем я заменяю ссылки на параметр в `discountedPrice`:

```java
private double discountedPrice (int basePrice, int discountLevel) {
    if (getDiscountLevel() == 2) return basePrice * 0.1;
    else return basePrice * 0.05;
}
```

После этого можно применить «Удаление параметра» ([Remove Parameter](Remove-Parameter.md)):

```java
public double getPrice() {
    int basePrice = _quantity * _itemPrice;
    int discountLevel = getDiscountLevel();
    double finalPrice = discountedPrice (basePrice);
    return finalPrice;
}
     
private double discountedPrice (int basePrice) {
    if (getDiscountLevel() == 2) return basePrice * 0.1;
    else return basePrice * 0.05;
}
```

Теперь можно избавиться от временной переменной:

```java
public double getPrice() {
    int basePrice = _quantity * _itemPrice;
    double finalPrice = discountedPrice (basePrice);
    return finalPrice;
}
```

После этого настает момент избавиться от другого параметра и его временной переменной. Остается следующее:

```java
public double getPrice() {
    return discountedPrice ();
}

private double discountedPrice () {
    if (getDiscountLevel() == 2) return getBasePrice() * 0.1;
    else return getBasePrice() * 0.05;
}

private double getBasePrice() {
    return _quantity * _itemPrice;
}
```

Поэтому можно также применить к `discountedPrice` «Встраивание метода» ([Inline Method](Inline-Method.md)):

```java
private double getPrice () {
    if (getDiscountLevel() == 2) return getBasePrice() * 0.1;
    else return getBasePrice() * 0.05;
}
```