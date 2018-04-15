# Exkurs ES2015+

## Das „neue“ JavaScript

ES2015 ist kurz gesagt eine modernisierte, aktuelle Version von JavaScript mit vielen neuen Funktionen und Syntax-Erleichterungen. Auf die wichtigsten möchte ich hier kurz eingehen, da ich sie in den nachfolgenden Beispielen verwenden werde. ES2015 hieß ursprünglich einmal ES6 und wird auch in einigen Blogs und Artikeln immer noch so bezeichnet. Stößt du also beim Lesen von Artikeln zu React auf den Begriff ES6 ist damit ES2015 gemeint. Ich schreibe hier meist von ES2015+ und meine damit Änderungen die seit 2015 in JavaScript eingeflossen sind.

{% hint style="info" %}
Das **ES** in ES2015 und ES6 steht für **ECMAScript**. Die ECMA International ist die Organisation, die hinter der Standardisierung der **ECMA-262** Spezifikation steht, auf der JavaScript basiert. Seit 2015 werden jährlich neue Versionen der Spezifikation veröffentlicht die aus historischen Gründen erst eine fortlaufende Versionsnummer beginnend ab Version 1 hatten, dann jedoch für mehr Klarheit die Jahreszahl ihrer Veröffentlichung angenommen haben. So wird ES6 heute offiziell als **ES2015** bezeichnet, ES7 als **ES2016**, usw.
{% endhint %}

Wenn du bereits Erfahrung mit ES2015 und den nachfolgenden Versionen hast kannst du dieses Kapitel überspringen. 

Wer mit React arbeitet nutzt in vermutlich 99% der Fälle auch Babel als Transpiler um sein JSX entsprechend in `createElement()`-Aufrufe zu transpilieren. Doch Babel transpiliert nicht nur JSX in ausführbares JavaScript, sondern hieß ursprünglich mal **6to5** und hat genau das gemacht: mit ES6-Syntax geschriebenes JavaScript in ES5 transpiliert, so dass neuere, zukünftige Features und Syntax-Erweiterungen auch in älteren Browsern ohne Unterstützung für „das neue“ JavaScript genutzt werden konnten.

## Variablen-Deklarationen mit let und const

Gab es bisher nur `var` um eine Variable zu deklarieren in JavaScript, kommen in ES2015 zwei neue Schlüsselwörter dazu mit denen Variablen deklariert werden können: `let` und `const`. Eine Variablendeklaration mit `var` wird dadurch in fast allen Fällen überflüssig, meist sind `let` oder `const` die sauberere Wahl. Doch wo ist der Unterschied? 

Anders als `var` existieren mit `let` oder `const` deklarierte Variablen **nur innerhalb des Scopes in dem sie deklariert wurden!** Ein solcher Scope kann eine Funktion sein wie sie bisher auch schon bei `var` einen neuen Scope erstellt hat aber auch Schleifen oder gar `if` Statements!

**Grobe Merkregel:** überall dort wo man eine öffnende geschweifte Klammer findet, wird auch ein neuer Scope geöffnet. Konsequenterweise schließt die schließende Klammer diesen Scope wieder. Dadurch sind Variablen deutlich eingeschränkter und gekapselter, was für gewöhnlich eine gute Sache ist. 

Möchte man den Wert einer Variable nochmal überschreiben, beispielsweise in einer Schleife, ist die Variable dafür mit `let` zu deklarieren. Möchte man die Referenz der Variable unveränderbar halten, sollte `const` benutzt werden.

Doch Vorsicht: anders als bei anderen Sprachen bedeutet `const` nicht, dass der komplette Inhalt der Variable konstant bleibt. Bei Objekten oder Arrays kann deren Inhalt auch bei mit `const` deklarierten Variablen noch verändert werden. Es kann lediglich das Referenzobjekt auf welche die Variable zeigt nicht mehr verändert werden.

### Beispiele

#### Der Unterschied zwischen `let`/`const` und `var`

Erst einmal zur Demonstration ein kurzes Beispiel wie sich die Variablendeklaration von `let` und `const` von denen mit `var` unterscheiden und was es bedeutet, dass erstere nur in dem Scope sichtbar sind, in dem sie definiert wurden:

```javascript
for (var i = 0; i < 10; i++) { }
console.log(i);
```

**Ausgabe:**

{% hint style="info" %}
10
{% endhint %}

Nun einmal dasselbe Beispiel mit `let`

```javascript
for (let j = 0; j < 10; j++) { }
console.log(j);
```

**Ausgabe:**

{% hint style="danger" %}
Uncaught ReferenceError: `j` is not defined
{% endhint %}

Während auf die Variable `var i`, einmal definiert, auch außerhalb der `for`-Schleife zugegriffen werden kann, existiert die Variable `let j` nur innerhalb des Scopes in dem sie definiert wurde. Und das ist in diesem Fall innerhalb die `for`-Schleife, die einen neuen Scope erzeugt.

Dies ist ein kleiner Baustein der uns später dabei helfen wird unsere Komponenten gekapselt und ohne ungewünschte Seiteneffekte zu erstellen.

#### Unterschiede zwischen `let` und `const`

Folgender Code ist valide und funktioniert, solange die Variable mittels `let` \(oder `var`\) deklariert wurde:

```javascript
let myNumber = 1234;
myNumber = 5678;
console.log(myNumber);
```

**Ausgabe:**

{% hint style="info" %}
5678
{% endhint %}

Der gleiche Code nochmal, nun allerdings mit `const`:

```javascript
const myNumber = 1234;
myNumber = 5678;
console.log(myNumber);
```

**Ausgabe:**

{% hint style="danger" %}
Uncaught TypeError: Assignment to constant variable.
{% endhint %}

Wir versuchen hier also eine durch `const` deklarierte Variable direkt zu überschreiben und werden dabei vom JavaScript-Interpreter zurecht in die Schranken gewiesen. Doch was, wenn wir stattdessen nur eine Eigenschaft _innerhalb_ eines mittels `const` deklarierten Objekts verändern wollen?

```javascript
const myObject = {
  a: 1
};
myObject.b = 2;
console.log(myObject);
```

**Ausgabe:**

{% hint style="info" %}
`{a: 1, b: 2}`
{% endhint %}

In diesem Fall gibt es keinerlei Probleme, da wir nicht die Referenz verändern, auf die die `myObject` Variable verweisen soll, sondern das Objekt, auf das verwiesen wird. Dies funktioniert ebenso mit Arrays, die verändert werden können, solange nicht der Wert der Variable selbst geändert wird!

**Erlaubt:**

```javascript
const myArray = [];
myArray.push(1);
myArray.push(2);
console.log(myArray);
```

**Ausgabe:**

{% hint style="info" %}
`[1, 2]`
{% endhint %}

**Nicht erlaubt, da wir die Variable direkt überschreiben würden:**

```text
const myArray = [];
myArray = Array.concat(1, 2);
```

{% hint style="danger" %}
Uncaught TypeError: Assignment to constant variable.
{% endhint %}

Möchten wir `myArray` also überschreibbar halten, müssen wir stattdessen `let` verwenden oder uns damit begnügen dass zwar der Inhalt des mittels `const` deklarierten Arrays veränderbar ist, nicht jedoch die Variable selbst.

## Arrow Functions

**Arrow Functions** sind eine weitere **deutliche** Vereinfachung die uns ES2015 gebracht hat. Bisher funktionierte eine Funktionsdeklaration so: man schrieb das Keyword `function`, optional gefolgt von einem Funktionsnamen, Klammern, in der die Funktionsargumente beschrieben wurden, sowie dem **Function Body**, also dem eigentlichen Inhalt der Funktion:

```javascript
function(arg1, arg2) {}
```

**Arrow Functions** vereinfachen uns das ungemein, indem sie erst einmal das `function` Keyword überflüssig machen:

```javascript
(arg1, arg2) => {}
```

Haben wir zudem nur einen Parameter, sind sogar die Klammern bei den Argumenten optional. Aus unserer Funktion

```javascript
function(arg) {}
```

Würde also die folgende **Arrow Function** werden:

```javascript
arg => {}
```

Jap, das ist eine gültige Funktion in ES2015!

Und es wird noch wilder. Soll unsere Funktion lediglich einen Ausdruck zurückzugeben als `return`-Wert, sind auch noch die Klammern optional. Vergleichen wir einmal eine Funktion die eine Zahl als einziges Argument entgegennimmt, diese verdoppelt und als `return`-Wert wieder aus der Funktion zurück gibt. Einmal in ES5:

```javascript
function double(number) {
  return number * 2;
}
```

… und als ES2015 **Arrow Function**:

```javascript
const double = number => number * 2;
```

In beiden Fällen liefert uns die eben deklarierte Funktion beim Aufruf von bspw. `double(5)` als Ergebnis `10` zurück!

Aber es gibt noch einen weiteren gewichtigen Vorteil, der bei der Arbeit mit React sehr nützlich sein wird: Arrow Functions haben keinen eigenen Constructor, können also nicht als Instanz in der Form `new MyArrowFunction()` erstellt werden, und binden auch kein eigenes `this` sondern erben `this` aus ihrem **Parent Scope**. Insbesondere Letzteres wird noch sehr hilfreich werden.

Auch das klingt fürchterlich kompliziert, lässt sich aber anhand eines einfachen Beispiels auch recht schnell erklären. Nehmen wir an wir definieren einen Button der die aktuelle Zeit in ein `div` schreiben soll, sobald ich ihn anklicke. Eine typische Funktion in ES5 könnte wie folgt aussehen:

```javascript
function TimeButton() {
  var button = document.getElementById('btn');
  var self = this;
  this.showTime = function() {
    document.getElementById('time').innerHTML = new Date();
  }
  button.addEventListener('click', function() {
    self.showTime();
  });
}
```

Da die als **Event Listener** angegebene Funktion keinen Zugriff auf ihren **Parent Scope**, also den **TimeButton** hat, speichern wir hier hilfsweise `this` in der Variable `self`. Kein unübliches Muster in ES5. Alternativ könnte man auch den Scope der Funktion explizit an `this` binden und dem **Event Listener** beibringen in welchem Scope sein Code ausgeführt werden soll:

```javascript
function TimeButton() {
  var button = document.getElementById('btn');
  this.showTime = function() {
    document.getElementById('time').innerHTML = new Date();
  }
  button.addEventListener('click', function() {
    this.showTime();
  }.bind(this));
}
```

Hier spart man sich zumindest die zusätzliche Variable `self`. Auch das ist möglich, aber nicht besonders elegant. 

An dieser Stelle kommt nun die **Arrow Function** ins Spiel, die, wie eben erwähnt, `this` aus ihrem **Parent Scope** erhält, also in diesem Fall aus unserer `TimeButton`-Instanz:

```javascript
function TimeButton() {
  var button = document.getElementById('btn');
  this.showTime = function() {
    document.getElementById('time').innerHTML = new Date();
  }
  button.addEventListener('click', () {
    this.showTime();
  });
}
```

Und schon haben wir im **Event Listener** Zugriff auf `this` des überliegenden Scopes! 

Keine `var self = this` Akrobatik mehr und auch kein `.bind(this)`. Wir können innerhalb des Event Listeners so arbeiten als befänden wir uns noch immer im `TimeButton` Scope! Das ist später insbesondere bei der Arbeit mit umfangreichen React-Komponenten mit vielen eigenen Class Properties und Methods hilfreich, da es Verwirrungen vorbeugt und nicht immer wieder einen neuen Scope erzeugt.

## Classes

Mit ES2015 fanden auch Klassen Einzug in JavaScript. Klassen kennt man eher aus objektorientierten Sprachen wie Java, in JavaScript gab es sie so explizit bisher allerdings noch nicht. Zwar war es auch schon vorher möglich durch die Verwendung von Funktionsinstanzen objektorientiert zu arbeiten und durch die `prototype`-Eigenschaft einer Funktion eigene Methoden und Eigenschaften zu definieren, dies war verglichen mit echten objektorientierten Sprachen jedoch sehr mühsam.

Dies ändert sich mit ES2015, wo es nun erstmals auch Klassen gibt, die mittels `class` Keyword definiert werden. Das ist für uns insofern interessant, da React, obwohl es viele Prinzipien der funktionalen Programmierung \(**Functional Programming**\) verfolgt, gleichzeitig auch zum wesentlichen Teil auf ES2015 Klassen setzt bei der Erstellung von Komponenten, in diesem Fall speziell von **Class Components**.

Eine Klasse besteht dabei aus einem Namen, kann \(optional\) einen **Constructor** haben der bei der Erstellung einer Klassen-Instanz aufgerufen wird und beliebig viele Klassen-Methoden besitzen.

```javascript
class Customer {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
  
  getFullName() {
    return this.firstName + ' ' + this.lastName;
  }
}

const firstCustomer = new Customer('Max', 'Mustermann');
console.log(firstCustomer.getFullName());
```

 **Ausgabe:**

{% hint style="info" %}
Max Mustermann
{% endhint %}

Auch das Erweitern bestehender Klassen mittels `extends` ist dabei möglich:

```javascript
class Customer extends Person {}
```

Oder eben:

```javascript
class MyComponent extends React.Component {}
```

Auch eine `super()`-Funktion kennt die ES2015 Klasse, um damit den **Constructor** ihrer **Parent Class** aufzurufen. Im Falle von React ist dies immer notwendig wenn ich in meiner eigenen Klasse eine `constructor`-Methode definiere. Diese muss dann dann `super()` aufrufen und ihre `props` an den Constructor der `React.Component` Klasse weiterzugeben:

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
  }
}
```

Tätet ihr das nicht, wäre `this.props` innerhalb eurer Komponente `undefined` und ihr könntet nicht auf die Props eurer Komponente zugreifen. Grundsätzlich sollte die Verwendung eines Constructors aber in den allermeisten Fällen nicht nötig sein, denn React stellt eigene sog. **Lifecycle-Methoden** bereit, die der Verwendung des Constructors vorzuziehen sind.

## Rest und Spread Operators

Eine weitere deutliche Vereinfachung ist die Einführung der der sog. Rest und Spread Operators für Objekte und Arrays. Streng genommen handelt es sich dabei bei der Verwendung in Kombination mit Objekten noch gar nicht um ES2015 Features, da diese sich noch in der Diskussion befinden und noch gar nicht endgültig in die ECMAScript Spezifikation aufgenommen wurden. Dies ändert sich erst mit ES2018. Eingeführt wurden Rest und Spread in ES2015 erstmals für Arrays. Durch die Verwendung von Babel ist die Nutzung auch mit Objekten aber heute bereits möglich und für gewöhnlich wird davon in React basierten Projekten auch rege Gebrauch gemacht.

Aber was ist das jetzt überhaupt? Fangen wir mit dem Spread Operator an.

### Spread Operator

Der Spread Operator sorgt dafür Werte sozusagen „auszupacken“. Wollte man in ES5 mehrere Argumente aus einem Array an eine Funktion übergeben, geschah das bisher meist über `Function.prototype.apply()`:

```javascript
function sumAll(number1, number2, number3) {
  return number1, number2, number3
}
var myArray = [1, 2, 3];
sumAll.apply(null, myArray);
```

**Ausgabe:**

{% hint style="info" %}
6
{% endhint %}

Mit dem Spread Operator, der aus drei Punkten \(...\) besteht, kann ich diese Argumente nun auspacken oder eben „spreaden“:

```javascript
function sumAll(number1, number2, number3) {
  return number1, number2, number3
}
var myArray = [1, 2, 3];
sumAll(...myArray);
```

**Ausgabe:**

{% hint style="info" %}
6
{% endhint %}

Ich muss also nicht mehr den Umweg über `apply()` gehen. Aber nicht nur bei Funktionsargumenten ist das hilfreich. Ich kann ihn auch nutzen um bspw. auf einfache Art und Weise zwei Arrays zu einem einzigen zu kombinieren:

```javascript
const greenFruits = ['kiwi', 'apple', 'pear'];
const redFruits = ['strawberry', 'cherry', 'raspberry'];
const allFruits = [...greenFruits, ...redFruits];
```

**Ergebnis:**

{% hint style="info" %}
`['kiwi', 'apple', 'pear', 'strawberry', 'cherry', 'raspberry']`
{% endhint %}

Dabei wird ein neues Array erstellt, welches alle Werte sowohl aus dem `greenFruits` als auch aus dem `redFruits` Array enthält. Doch nicht nur das: dabei wird auch ein gänzlich neues Array erstellt und nicht bloß eine Referenz der beiden alten. Dies wird im weiteren Verlauf wenn wir an die **readonly**-Anforderung unserer Props noch sehr nützlich sein. Und so kann man den Spread Operator auch verwenden um eine einfache Kopie eines Arrays zu erstellen:

```javascript
const users = ['Manuel', 'Chris', 'Ben'];
const selectedUsers = [...users];
```

`selectedUsers` ist in diesem Fall eine Kopie unseres `users` Arrays mit all seinen Werten. Verändern wir nun das Users Array, hat dies auf unser `selectedUsers` Array keinerlei Auswirkungen.

Bei Objekten verhält sich der Spread Operator sehr ähnlich. Hier werden statt der einzelnen Werte alle Eigenschaften eines Objekts die „enumerable“ \(aufzählbar\) sind, also ganz grob gesagt bei der Verwendung in einer `for(… in …)` Schleife angezeigt werden würden.

Hier eignet sich der Spread Operator hervorragend um neue Objekte zu erstellen:

```text
const teamSettings = { language: 'en-US', 
```



