# Удаление управляющего флага (Remove Control Flag)

Имеется переменная, действующая как управляющий флаг для ряда булевых выражений.

_Используйте вместо нее break или return._

## Мотивировка

Встретившись с серией условных выражений, часто можно обнаружить управляющий флаг, с помощью которого определяется окончание просмотра:

```
set done to false
while not done
    if (condition)
        do something
        set done to true
    next step of loop
```

От таких управляющих флагов больше неприятностей, чем пользы. Их присутствие диктуется правилами структурного программирования, согласно которым в процедурах должна быть одна точка входа и одна точка выхода. Я согласен с тем, что должна быть одна точка входа (чего требуют современные языки программирования), но требование одной точки выхода приводит к сильно запутанным условным операторам, в коде которых есть такие неудобные флаги. Для того чтобы выбраться из сложного условного оператора, в языках есть команды `break` и `continue`. Часто бывает удивительно, чего можно достичь, избавившись от управляющего флага. Действительное назначение условного оператора становится гораздо понятнее.

## Техника

Очевидный способ справиться с управляющими флагами предоставляют имеющиеся в Java операторы `break` и `continue`.

* Определите значение управляющего флага, при котором происходит выход из логического оператора.
* Замените присваивания значения для выхода операторами `break` или `continue`.
* Выполняйте компиляцию и тестирование после каждой замены.

Другой подход, применимый также в языках без операторов `break` и `continue`, состоит в следующем:

* Выделите логику в метод.
* Определите значение управляющего флага, при котором происходит выход из логического оператора.
* Замените присваивания значения для выхода оператором `return`.
* Выполняйте компиляцию и тестирование после каждой замены.

Даже в языках, где есть `break` или `continue`, я обычно предпочитаю применять выделение и `return`. Оператор `return` четко сигнализирует, что никакой код в методе больше не выполняется. При наличии кода такого вида часто в любом случае надо выделять этот фрагмент.

Следите за тем, не несет ли управляющий флаг также информации о результате. Если это так, то управляющий флаг все равно необходим, либо можно возвращать это значение, если вы выделили метод.

## Пример: замена простого управляющего флага оператором 

Следующая функция проверяет, не содержится ли в списке лиц кто либо из парочки подозрительных, имена которых жестко закодированы:

```java
void checkSecurity(String[] people) {
    boolean found = false;
    for (int i = 0; i < people.length; i++) {
        if (! found) {
            if (people[i].equals ("Don")) {
                sendAlert();
                found = true;
            }
            
            if (people[i].equals ("John")) {
                sendAlert();
                found = true;
            }
        }
    } 
}
```

В таких случаях заметить управляющий флаг легко. Это фрагмент, в котором переменной `found` присваивается значение `true`. Ввожу операторы `break` по одному:

```java
void checkSecurity(String[] people) {
    boolean found = false;
    for (int i = 0; i < people.length; i++) {
        if (! found) {
            if (people[i].equals ("Don")) {
                sendAlert();
                break; // !!!
            }
            if (people[i].equals ("John")) {
                sendAlert();
                found = true;
            }
        }
    }
}
```

пока не будут заменены все присваивания:

```java
void checkSecurity(String[] people) {
    boolean found = false;
    for (int i = 0; i < people.length; i++) {
        if (! found) {
            if (people[i].equals ("Don")) {
                sendAlert();
                break;
            }
        
            if (people[i].equals ("John")) {
                sendAlert();
                break; // !!!
            }
        }
    }
}
```

После этого можно убрать все ссылки на управляющий флаг:

```java
void checkSecurity(String[] people) {
    for (int i = 0; i < people.length; i++) {
        if (people[i].equals ("Don")) {
            sendAlert();
            break;
        }
        
        if (people[i].equals ("John")) {
            sendAlert();
            break;
        }
    }
}
```

## Пример: использование оператора return, возвращающего значение управляющего флага

Другой стиль данного рефакторинга использует оператор `return`. Проиллюстрирую это вариантом, в котором управляющий флаг выступает в качестве возвращаемого значения:

```java
void checkSecurity(String[] people) {
    String found = "";
    for (int i = 0; i < people.length; i++) {
        if (found.equals("")) {
            if (people[i].equals ("Don")){
                sendAlert();
                found = "Don";
            }
            
            if (people[i].equals ("John")){
                sendAlert();
                found = "John";
            }
        }
    }
    someLaterCode(found);
}
```

Здесь `found` служит двум целям: задает результат и действует в качестве управляющего флага. Если я вижу такой код, то предпочитаю выделить фрагмент, определяющий `found`, в отдельный метод:

```java
void checkSecurity(String[] people) {
    String found = foundMiscreant(people);
    someLaterCode(found);
}

String foundMiscreant(String[] people) {
    String found = "";
    for (int i = 0; i < people.length; i++) {
        if (found.equals("")) {
            if (people[i].equals ("Don")){
                sendAlert();
                found = "Don";
            }
            
            if (people[i].equals ("John")){
                sendAlert();
                found = "John";
            }
        }
    }
    
    return found;
}
```

Теперь я могу заменять управляющий флаг оператором `return`:

```java
String foundMiscreant(String[] people){
    String found = "";
    for (int i = 0; i < people.length; i++) {
        if (found.equals("")) {
            if (people[i].equals ("Don")){
                sendAlert();
                return "Don"; // !!!
            }
            
            if (people[i].equals ("John")){
                sendAlert();
                found = "John";
            }
        }
    }
    
    return found;
}
```

пока не избавлюсь от управляющего флага:

```java
String foundMiscreant(String[] people){
    for (int i = 0; i < people.length; i++) {
        if (people[i].equals ("Don")){
            sendAlert();
            return "Don";
        }
        
        if (people[i].equals ("John")){
            sendAlert();
            return "John";
        }    
    }
    
    return "";
}
```

Применение `return` возможно и тогда, когда нет еобходимости возвращать значение – просто используйте `return` без аргумента.

Конечно, здесь возникают проблемы, связанные с побочными эффектами функции. Поэтому надо применять «Разделение запроса и модификатора» ([Separate Query from Modifier](/Separate-Query-from-Modifier.md)). Данный пример будет продолжен в соответствующем разделе.