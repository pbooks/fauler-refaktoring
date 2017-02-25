# Замена вложенных условных операторов граничным оператором (Replace Nested Conditional with Guard Clauses)

Метод использует условное поведение, из которого неясен нормальный путь выполнения.

_Используйте граничные условия для всех особых случаев._

```java
double getPayAmount() {
    double result;
    if (_isDead) result = deadAmount();
    else {
        if (_isSeparated) result = separatedAmount();
        else {
            if (_isRetired) result = retiredAmount();
            else result = normalPayAmount();
        }
    }
    
    return result;
}
```

![alt tag](/images/arrow.jpg)

```java
double getPayAmount() {
    if (_isDead) return deadAmount();
    if (_isSeparated) return separatedAmount();
    if (_isRetired) return retiredAmount();
    return normalPayAmount();
}
```

## Мотивировка

Часто оказывается, что условные выражения имеют один из двух видов. В первом виде это проверка, при которой любой выбранный ход событий является частью нормального поведения. Вторая форма представляет собой ситуацию, в которой один результат условного оператора указывает на нормальное поведение, а другой – на необычные условия.

Эти виды условных операторов несут в себе разный смысл, и этот смысл должен быть виден в коде. Если обе части представляют собой нормальное поведение, используйте условие с ветвями `if` и `else`. Если условие является необычным, проверьте условие и выполните `return`, если условие истинно. Такого рода проверка часто называется _граничным оператором_ (_guard clause_ [Beck]).

Главный смысл «Замены вложенных условных операторов граничным оператором» ([Replace Nested Conditional with Guard Clauses](/Replace-Nested-Conditional-with-Guard-Clauses.md)) состоит в придании выразительности. При использовании конструкции `if-then-else` ветви `if` и ветви `else` придается равный вес. Это говорит читателю, что обе ветви обладают равной вероятностью и важностью. Напротив, защитный оператор говорит: «Это случается редко, и если все таки произошло, надо сделать то-то и то-то и выйти».

Я часто прибегаю к «Замене вложенных условных операторов граничным оператором» ([Replace Nested Conditional with Guard Clauses](/Replace-Nested-Conditional-with-Guard-Clauses.md)), когда работаю с программистом, которого учили, что в методе должны быть только одна точка входа и одна точка выхода. Одна точка входа обеспечивается современными языками, а в правиле одной точки выхода на самом деле пользы нет. Ясность – главный принцип: если метод понятнее, когда в нем одна точка выхода, сделайте ее единственной, в противном случае не стремитесь к этому.

## Техника

* Для каждой проверки вставьте граничный оператор.

_Граничный оператор осуществляет возврат или возбуждает исключительную ситуацию._

* Выполняйте компиляцию и тестирование после каждой замены проверки граничным оператором.

_Если все граничные операторы возвращают одинаковый результат, примените «Консолидацию условных выражений» ([Consolidate Conditional Expression](/Consolidate-Conditional-Expression.md))._

## Пример

Представьте себе работу системы начисления зарплаты с особыми правилами для служащих, которые умерли, проживают раздельно или вышли на пенсию. Такие случаи необычны, но могут встретиться.

Допустим, я вижу такой код:

```java
double getPayAmount() {
    double result;
    if (_isDead) result = deadAmount();
    else {
        if (_isSeparated) result = separatedAmount();
        else {
            if (_isRetired) result = retiredAmount();
            else result = normalPayAmount();
        }
    }
    
    return result;
}
```

В этом коде проверки маскируют выполнение обычных действий, поэтому при использовании граничных операторов код станет яснее. Буду вводить их по одному и начну сверху:

```java
double getPayAmount() {
    double result;
    if (_isDead) return deadAmount(); // !!!
    if (_isSeparated) result = separatedAmount();
    else {
        if (_isRetired) result = retiredAmount();
        else result = normalPayAmount();
    }
     
    return result;
}
```

Продолжаю делать замены по одной на каждом шагу:

```java
double getPayAmount() {
    double result;
    if (_isDead) return deadAmount();
    if (_isSeparated) return separatedAmount(); // !!!
    if (_isRetired) result = retiredAmount();
    else result = normalPayAmount();
    return result;
}
```

и затем

```java
double getPayAmount() {
    double result;
    if (_isDead) return deadAmount();
    if (_isSeparated) return separatedAmount();
    if (_isRetired) return retiredAmount(); // !!!
    result = normalPayAmount();
    return result;
}
```

После этих изменений временная переменная `result` не оправдывает своего существования, поэтому я ее убиваю:

```java
double getPayAmount() {
    if (_isDead) return deadAmount();
    if (_isSeparated) return separatedAmount();
    if (_isRetired) return retiredAmount();
    return normalPayAmount();
}
```

Вложенный условный код часто пишут программисты, которых учили, что в методе должна быть только одна точка выхода. Я считаю, что это слишком упрощенное правило. Если метод больше не представляет для меня интереса, я указываю на это путем выхода из него. Заставляя читателя рассматривать пустой блок `else`, вы только создаете преграды на пути понимания кода.

## Пример: обращение условий

Рецензируя рукопись этой книги, Джошуа Кериевски (Joshua Kerievsky) отметил, что часто «Замена вложенных условных операторов граничными операторами» ([Replace Nested Conditional with Guard Clauses](/Replace-Nested-Conditional-with-Guard-Clauses.md)) осуществляется путем обращения условных выражений. Он любезно предоставил пример, что позволило мне не подвергать свое воображение дальнейшему испытанию:

```java
public double getAdjustedCapital() {
    double result = 0.0;
    if (_capital > 0.0) {
        if (_intRate > 0.0 && _duration > 0.0) {
            result = (_income / _duration) * ADJ_FACTOR;
        }
    }
    return result;
}
```
Я снова буду выполнять замену поочередно, но на этот раз при вставке граничного оператора условие обращается:

```java
public double getAdjustedCapital() {
    double result = 0.0;
    if (_capital <= 0.0) return result; // !!!
    if (_intRate > 0.0 && _duration > 0.0) {
        result = (_income / _duration) * ADJ_FACTOR;
    }
    return result;
}
```

Поскольку следующий условный оператор немного сложнее, я буду обращать его в два этапа. Сначала я ввожу отрицание:

```java
public double getAdjustedCapital() {
    double result = 0.0;
    if (_capital <= 0.0) return result;
    if (!(_intRate > 0.0 && _duration > 0.0)) return result; // !!!
    result = (_income / _duration) * ADJ_FACTOR;
    return result;
}
```

Когда в условном операторе сохраняются такие отрицания, у меня мозги съезжают набекрень, поэтому я упрощаю его следующим образом:

```java
public double getAdjustedCapital() {
    double result = 0.0;
    if (_capital <= 0.0) return result;
    if (_intRate <= 0.0 || _duration <= 0.0) return result; // !!!
    result = (_income / _duration) * ADJ_FACTOR;
    return result;
}
```

В таких ситуациях я стараюсь помещать явные значения в операторы возврата из граничных операторов. Благодаря этому можно легко видеть результат, получаемый при срабатывании защиты (я бы также попробовал здесь применить «Замену магического числа символической константой» ([Replace Magic Number with Symbolic Constant]('/Replace-Magic-Number-with-Symbolic-Constant.md)).

```java
public double getAdjustedCapital() {
    double result = 0.0;
    if (_capital <= 0.0) return 0.0; // !!!
    if (_intRate <= 0.0 || _duration <= 0.0) return 0.0; // !!!
    result = (_income / _duration) * ADJ_FACTOR;
    return result;
}
```

После этого можно также удалить временную переменную:

```java
public double getAdjustedCapital() {
    if (_capital <= 0.0) return 0.0;
    if (_intRate <= 0.0 || _duration <= 0.0) return 0.0;
    return (_income / _duration) * ADJ_FACTOR;
}
```