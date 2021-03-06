## Консолидация дублирующихся условных фрагментов (Consolidate Duplicate Conditional Fragments)

Один и тот же фрагмент кода присутствует во всех ветвях условного выражения.

_Переместите его за пределы выражения._

```java
if (isSpecialDeal()) {
    total = price * 0.95;
    send();
} else {
    total = price * 0.98;
    send();
}
```

![alt tag](/images/arrow.jpg)

```java
if (isSpecialDeal())
    total = price * 0.95;
else
    total = price * 0.98;
send();
```

## Мотивировка

Иногда обнаруживается, что во всех ветвях условного оператора выполняется один и тот же фрагмент кода. В таком случае следует переместить этот код за пределы условного оператора. В результате становится яснее, что меняется, а что остается постоянным.

## Техника

* Выявите код, который выполняется одинаковым образом вне зависимости от значения условия.
* Если общий код находится в начале, поместите его перед условным оператором.
* Если общий код находится в конце, поместите его после условного оператора.
* Если общий код находится в середине, посмотрите, модифицирует ли что нибудь код, находящийся до него или после него. Если да, то общий код можно переместить до конца вперед или назад. После этого можно его переместить, как это описано для кода, находящегося в конце или в начале.
* Если код состоит из нескольких предложений, надо выделить его в метод.

## Пример

Данная ситуация обнаруживается, например, в следующем коде:

```java
if (isSpecialDeal()) {
    total = price * 0.95;
    send();
} else {
    total = price * 0.98;
    send();
}
```

Поскольку метод `send` выполняется в любом случае, следует вынести его из условного оператора:

```java
if (isSpecialDeal())
    total = price * 0.95;
else
    total = price * 0.98;
send();
```

То же самое может возникать с исключительными ситуациями. Если код повторяется после оператора, вызывающего исключительную ситуацию, в блоке `try` и во всех блоках `catch`, можно переместить его в блок `final`.