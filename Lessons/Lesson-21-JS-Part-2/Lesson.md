# JAVASCRIPT

## Функции
Наряду с объектами функции также являются ключевыми компонентами языка JavaScript. Базовые функции очень просты:

```
function add(x, y) {
    var total = x + y;
    return total;
}
```

В этом примере показано практически всё, что нужно знать о функциях. Функции в JavaScript могут принимать ноль или более параметров. Тело функции может содержать любые выражения и определять свои собственные переменные, которые будут для этой функции локальными. Инструкция return используется для возврата значения и остановки выполнения функции. Если инструкции return в функции нет (или есть, но не указано возвращаемое значение), то JavaScript возвратит undefined.

Можно вызвать функцию, вообще не передавая ей параметры. В таком случае будет считаться, что их значения равны undefined:

```
add(); // NaN
// Нельзя проводить сложение undefined и undefined
```

Можно передать больше аргументов, чем ожидает функция:

```
add(2, 3, 4); // 5
// используются только первые два аргумента, "4" игнорируется
```

Это может показаться бессмысленным, но на самом деле функции могут получить доступ к "лишним" аргументам с помощью псевдомассива arguments, в нём содержатся значения всех аргументов, переданных функции. Давайте напишем функцию, которая принимает неограниченное количество аргументов:

```
function add() {
    var sum = 0;
    for (var i = 0, j = arguments.length; i < j; i++) {
        sum += arguments[i];
    }
    return sum;
}

add(2, 3, 4, 5); // 14
```

Или создадим функцию для вычисления среднего значения:

```
function avg() {
    var sum = 0;
    for (var i = 0, j = arguments.length; i < j; i++) {
        sum += arguments[i];
    }
    return sum / arguments.length;
}
avg(2, 3, 4, 5); // 3.5
```

На тот случай, если вы хотите передать существующий массив объектов, не переписывая функцию заново, то в JavaScript есть возможность вызывать функцию с произвольным массивом аргументов. Для этого используется метод apply():

```
avg.apply(null, [2, 3, 4, 5]); // 3.5
```

Вторым аргументом метода apply() передаётся массив, который будет передан функции в качестве аргументов. О первом аргументе мы поговорим позже. Наличие у функций методов также говорит о том, что на самом деле они являются объектами.

В JavaScript можно создавать анонимные функции:

```
var avg = function() {
    var sum = 0;
    for (var i = 0, j = arguments.length; i < j; i++) {
        sum += arguments[i];
    }
    return sum / arguments.length;
}
```

Данная запись семантически равнозначна записи function avg(). Это даёт возможность использовать разные интересные трюки. Вот посмотрите, как можно "спрятать" локальные переменные в функции:

```
var a = 1;
var b = 2;
(function() {
    var b = 3;
    a += b;
})();
a; // 4
b; // 2
```

В JavaScript есть возможность рекурсивного вызова функции. Это может оказаться полезным при работе с иерархическими (древовидными) структурами данных (например такие, которые встречаются при работе с DOM).

```
function countChars(elm) {
    if (elm.nodeType == 3) { // TEXT_NODE
        return elm.nodeValue.length;
    }
    var count = 0;
    for (var i = 0, child; child = elm.childNodes[i]; i++) {
        count += countChars(child);
    }
    return count;
}
```

Тут мы сталкиваемся с проблемой: как вызвать функцию рекурсивно, если у неё нет имени? Для этого в JavaScript есть именованные функциональные выражения [IIFEs (Immediately Invoked Function Expressions)](https://developer.mozilla.org/en-US/docs/Glossary/IIFE). Вот пример использования именованной самовызывающейся функции:

```
var charsInBody = (function counter(elm) {
    if (elm.nodeType == 3) { // TEXT_NODE
        return elm.nodeValue.length;
    }
    var count = 0;
    for (var i = 0, child; child = elm.childNodes[i]; i++) {
        count += counter(child);
    }
    return count;
})(document.body);
```

Имя функции в примере доступно только внутри самой функции. Это улучшает оптимизацию и читаемость кода.

## Собственные объекты
В классическом Объектно-Ориентированном Программировании (ООП) объекты — это модели данных и методов, которые этими данными оперируют. JavaScript - это язык, основанный на прототипах, и в его определении нет понятия классов, таких, как в языках C++ или С#. Вместо классов JavaScript использует функции. Давайте представим объект с личными данными, содержащий поля с именем и фамилией. Есть два типа отображения имён: "Имя Фамилия" или "Фамилия, Имя". С помощью объектов и функций можно сделать следующее:

```
function makePerson(first, last) {
    return {
        first: first,
        last: last,
        fullName: function() {
            return this.first + ' ' + this.last;
        },
        fullNameReversed: function() {
            return this.last + ', ' + this.first;
        }
    }
}
s = makePerson("Simon", "Willison")
s.fullName(); // Simon Willison
s.fullNameReversed(); // Willison, Simon
```

А вот кое-что новенькое: ключевое слово *'this'*. Когда *'this'* используется внутри функции, оно ссылается на текущий объект. Значение ключевого слова зависит от способа вызова функции. Если вызвать функцию с обращением к объекту через точку или квадратные скобки, то *'this'* получится равным данному объекту. В ином случае *'this'* будет ссылаться на глобальный объект. Это часто приводит к ошибкам. Например:

```
s = makePerson("Simon", "Willison")
var fullName = s.fullName;
fullName(); // undefined undefined
```

Используя особенность ключевого слова 'this', можно улучшить код функции makePerson:

```
function Person(first, last) {
    this.first = first;
    this.last = last;
    this.fullName = function() {
        return this.first + ' ' + this.last;
    };
    this.fullNameReversed = function() {
        return this.last + ', ' + this.first;
    };
}
var s = new Person("Simon", "Willison");
```

В примере мы использовали новое ключевое слово: 'new'. Оно тесно связано с 'this'. Данное ключевое слово создаёт новый пустой объект, а потом вызывает указанную функцию, а this получает ссылку на этот новый объект. Функции, которые предназначены для вызова с 'new' называются конструкторами. Существует соглашение, согласно которому все функции-конструкторы записываются с заглавной буквы.

Мы доработали наш код в предыдущем примере, но всё равно остался один неприятный момент с самостоятельным вызовом  fullName().

Каждый раз, когда с помощью конструктора создаётся новый объект, мы заново создаём и две новые функции. Гораздо удобнее создать эти функции отдельно и дать доступ к ним конструктору:

```
function personFullName() {
    return this.first + ' ' + this.last;
}
function personFullNameReversed() {
    return this.last + ', ' + this.first;
}
function Person(first, last) {
    this.first = first;
    this.last = last;
    this.fullName = personFullName;
    this.fullNameReversed = personFullNameReversed;
}
```

Уже лучше: мы создали функции-методы только один раз, а при новом вызове функции-конструктора просто ссылаемся на них. Можно сделать ещё лучше? Конечно:

```
function Person(first, last) {
    this.first = first;
    this.last = last;
}
Person.prototype.fullName = function() {
    return this.first + ' ' + this.last;
}
Person.prototype.fullNameReversed = function() {
    return this.last + ', ' + this.first;
}
```

Person.prototype это объект, доступ к которому есть у всех экземпляров класса Person. Он создаёт особую цепочку прототипов. Каждый раз, когда вы пытаетесь получить доступ к несуществующему свойству объекта Person, JavaScript проверяет, существует ли свойство в Person.prototype. В результате все, что передано в Person.prototype, становится доступным и всем экземплярам этого конструктора через this объект.

Это очень мощный инструмент. JavaScript позволяет изменять прототипы в любое время, это значит, что можно добавлять новые методы к существующим объектам во время выполнения программы:

```
s = new Person("Simon", "Willison");
s.firstNameCaps();
// TypeError on line 1: s.firstNameCaps is not a function

Person.prototype.firstNameCaps = function() {
    return this.first.toUpperCase()
}
s.firstNameCaps(); // "SIMON"
```

Занимательно то, что добавлять свойства в прототип можно и для встроенных объектов JavaScript. Давайте добавим новый метод reversed классу String, этот метод будет возвращать строку задом наперёд:

```
var s = "Simon";
s.reversed(); // TypeError on line 1: s.reversed is not a function

String.prototype.reversed = function reversed() {
    var r = "";
    for (var i = this.length - 1; i >= 0; i--) {
        r += this[i];
    }
    return r;
}
s.reversed(); // "nomiS"
```

Данный метод будет работать даже на литералах строки!

```
"This can now be reversed".reversed();
```

Помните, мы вызывали avg.apply() с первым аргументом равным null? Теперь мы можем сделать так: первым аргументом, переданным методу apply() будет объект, который примет значение 'this'. Вот к примеру упрощённая реализация 'new'

```
function trivialNew(constructor, ...args) {
    var o = {}; // Создаём новый объект
    constructor.apply(o, args);
    return o;
}
```

## Вложенные функции
Объявлять новые функции можно и внутри других функций. Мы использовали этот приём чуть выше, создавая функцию makePerson(). Главная особенность вложенных функций в том, что они получают доступ к переменным, объявленным в их функции-родителе:

```
function parentFunc() {
  var a = 1;

  function nestedFunc() {
    var b = 4; // parentFunc can't use this
    return a + b;
  }
  return nestedFunc(); // 5
}
```

Это очень полезное свойство, которое делает сопровождение кода более удобным. Если ваша функция в своей работе использует другие функции, которые больше нигде не используются, то можно просто вложить вспомогательные функции в основную. Это сократит количество функций в глобальном объекте, что довольно неплохо.

## Замыкания (Closures)
Мы подошли к одному из самых мощных и непонятных инструментов JavaScript. Давайте разберёмся.

```
function makeAdder(a) {
    return function(b) {
        return a + b;
    };
}

var x = makeAdder(5);
var y = makeAdder(20);
x(6); // ?
y(7); // ?
```

Функция makeAdder создаёт новую функцию, которая прибавляет полученное значение к значению, которые было получено при создании функции.

Такой же фокус мы наблюдали в предыдущем примере, когда внутренние функции получали доступ к переменным той функции, в которой были объявлены. Только в нашем примере основная функция возвращает вложенную. Поначалу может показаться, что локальные переменные при этом перестанут существовать. Но они продолжают существовать — иначе код попросту не сработал бы. Вдобавок ко всему у нас есть две разные "копии" функции makeAdder, присвоенные разным переменным (одна копия, в которой а - это 5, а во второй а - это 20). Вот что имеем в результате вызова:

```
x(6); // возвратит 11
y(7); // возвратит 27
```

И вот что произошло: когда JavaScript выполняет функцию, создаётся объект *'scope'*, который содержит в себе все локальные переменные, объявленные внутри этой функции. Он инициализируется любым значением, переданным функции в качестве параметра. *'Scope'* подобен глобальному объекту, который содержит все глобальные переменные и функции, кроме нескольких важных отличий: при каждом вызове функции создаётся новый объект *'scope'* и, в отличие от глобального, к объекту *'scope'* нельзя получить прямой доступ из вашего кода. И нет способа пройтись по свойствам данного объекта.

Так что при вызове функции makeAdder создаётся новый объект *'scope'* с единственным свойством: a, которому присваивается значение, переданное функции в качестве аргумента.  Потом makeAdder возвращает новую анонимную функцию. В любом другом случае 'сборщик мусора' удалил бы объект scope, но возвращаемая функция ссылается на этот объект. В итоге объект scope не удаляется до тех пор, пока существует хотя бы одна ссылка на него.

Все объекты scope соединяются в цепочку областей видимости, которая похожа на цепочку прототипов в объектной системе JavaScript.
