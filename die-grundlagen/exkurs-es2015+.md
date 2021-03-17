# Exkurs ES2015+

## Das „neue“ JavaScript

**ES2015** ist kurz gesagt eine modernisierte, aktuelle Version von JavaScript mit vielen neuen Funktionen und Syntax-Erleichterungen. **ES2015** ist der Nachfolger von **ECMAScript** in der Version 5 \(**ES5**\), hieß daher ursprünglich auch einmal **ES6** und wird auch in einigen Blogs und Artikeln immer noch so bezeichnet. Stößt du also beim Lesen von Artikeln zu React auf den Begriff **ES6,** ist damit **ES2015** gemeint. Ich schreibe hier meist von **ES2015+** und meine damit Änderungen, die seit 2015 in JavaScript eingeflossen sind. Dazu gehören ES2016 \(ES7\), ES2017 \(ES8\) und ES2018 \(ES9\).

{% hint style="info" %}
Das **ES** in **ES2015** und **ES6** steht für **ECMAScript**. Die ECMA International ist die Organisation, die hinter der Standardisierung der **ECMA-262-**Spezifikation steht, auf der JavaScript basiert. Seit 2015 werden jährlich neue Versionen der Spezifikation veröffentlicht, die aus historischen Gründen erst eine fortlaufende Versionsnummer beginnend ab Version 1 hatten, dann jedoch für mehr Klarheit die Jahreszahl ihrer Veröffentlichung angenommen haben. So wird **ES6** heute offiziell als **ES2015** bezeichnet, **ES7** als **ES2016**, usw.
{% endhint %}

Wer mit React arbeitet, nutzt in vermutlich 99% der Fälle auch **Babel** als **Transpiler,** um sein **JSX** entsprechend in `createElement()`-Aufrufe zu transpilieren. Doch **Babel** transpiliert nicht nur **JSX** in ausführbares JavaScript, sondern hieß ursprünglich mal **6to5** und hat genau das gemacht: mit **ES6**-Syntax geschriebenes JavaScript in **ES5** transpiliert, sodass neuere, zukünftige Features und Syntax-Erweiterungen auch in älteren Browsern ohne Unterstützung für „das neue“ JavaScript genutzt werden konnten.

Auf die meiner Meinung nach wichtigsten und nützlichsten neuen Funktionen und Möglichkeiten in **ES2015** und den folgenden Versionen möchte ich in diesem Kapitel eingehen. Dabei werde ich mich auf die neuen Funktionen beschränken, mit denen man bei der Arbeit mit React häufiger zu tun haben wird und die euch Entwicklern das Leben am meisten vereinfachen.

**Wenn du bereits Erfahrung mit ES2015 und den nachfolgenden Versionen hast, kannst du dieses Kapitel überspringen!**

<span class="force-break-before"></span>

## Variablen-Deklarationen mit let und const

Gab es bisher nur `var` um in JavaScript eine Variable zu deklarieren, kommen in ES2015 zwei neue Schlüsselwörter dazu, mit denen Variablen deklariert werden können: `let` und `const`. Eine Variablendeklaration mit `var` wird dadurch in fast allen Fällen überflüssig, meist sind `let` oder `const` die sauberere Wahl. Doch wo ist der Unterschied?

Anders als `var` existieren mit `let` oder `const` deklarierte Variablen **nur innerhalb des Scopes ,in dem sie deklariert wurden!** Ein solcher Scope kann eine Funktion sein wie sie bisher auch schon bei `var` einen neuen Scope erstellt hat, aber auch Schleifen oder gar `if-`Statements!

**Grobe Merkregel:** überall dort, wo man eine öffnende geschweifte Klammer findet, wird auch ein neuer Scope geöffnet. Konsequenterweise schließt die schließende Klammer diesen Scope wieder. Dadurch sind Variablen deutlich eingeschränkter und gekapselter, was für gewöhnlich eine gute Sache ist.

Möchte man den Wert einer Variable nochmal überschreiben, beispielsweise in einer Schleife, ist die Variable dafür mit `let` zu deklarieren. Möchte man die Referenz der Variable unveränderbar halten, sollte `const` benutzt werden.

Doch Vorsicht: anders als bei anderen Sprachen bedeutet `const` nicht, dass der komplette Inhalt der Variable konstant bleibt. Bei Objekten oder Arrays kann deren Inhalt auch bei mit `const` deklarierten Variablen noch verändert werden. Es kann lediglich das Referenzobjekt, auf welche die Variable zeigt, nicht mehr verändert werden.

### Der Unterschied zwischen `let`/`const` und `var`

Erst einmal zur Demonstration ein kurzes Beispiel dafür, wie sich die Variablendeklaration von `let` und `const` von denen mit `var` unterscheiden und was es bedeutet, dass erstere nur in dem Scope sichtbar sind, in dem sie definiert wurden:

```javascript
for (var i = 0; i < 10; i++) {}
console.log(i);
```

**Ausgabe:**

{% hint style="info" %}
10
{% endhint %}

Nun einmal dasselbe Beispiel mit `let`

```javascript
for (let j = 0; j < 10; j++) {}
console.log(j);
```

**Ausgabe:**

{% hint style="danger" %}
Uncaught ReferenceError: `j` is not defined
{% endhint %}

Während auf die Variable `var i`, einmal definiert, auch außerhalb der `for`-Schleife zugegriffen werden kann, existiert die Variable `let j` nur innerhalb des Scopes, in dem sie definiert wurde. Und das ist in diesem Fall innerhalb der `for`-Schleife, die einen neuen Scope erzeugt.

Dies ist ein kleiner Baustein, der uns später dabei helfen wird unsere Komponenten gekapselt und ohne ungewünschte Seiteneffekte zu erstellen.

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
  a: 1,
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
myArray = myArray.concat(1, 2);
```

{% hint style="danger" %}
Uncaught TypeError: Assignment to constant variable.
{% endhint %}

Möchten wir `myArray` also überschreibbar halten, müssen wir stattdessen `let` verwenden oder uns damit begnügen, dass zwar der Inhalt des mittels `const` deklarierten Arrays veränderbar ist, nicht jedoch die Variable selbst.

<span class="force-break-before"></span>

## Arrow Functions

**Arrow Functions** sind eine weitere **deutliche** Vereinfachung, die uns ES2015 gebracht hat. Bisher funktionierte eine Funktionsdeklaration so: man schrieb das Keyword `function`, optional gefolgt von einem Funktionsnamen, Klammern, in der die Funktionsargumente beschrieben wurden, sowie dem **Function Body**, also dem eigentlichen Inhalt der Funktion:

```javascript
function(arg1, arg2) {}
```

**Arrow Functions** vereinfachen uns das ungemein, indem sie erst einmal das `function`-Keyword überflüssig machen:

```javascript
(arg1, arg2) => {};
```

Haben wir zudem nur einen Parameter, sind sogar die Klammern bei den Argumenten optional. Aus unserer Funktion

```javascript
function(arg) {}
```

würde also die folgende **Arrow Function** werden:

```javascript
(arg) => {};
```

Jap, das ist eine gültige Funktion in ES2015!

Und es wird noch wilder. Soll unsere Funktion lediglich einen Ausdruck als `return`-Wert zurückgeben, sind auch noch die Klammern optional. Vergleichen wir einmal eine Funktion, die eine Zahl als einziges Argument entgegennimmt, diese verdoppelt und als `return`-Wert wieder aus der Funktion zurückgibt. Einmal in ES5:

```javascript
function double(number) {
  return number * 2;
}
```

… und als ES2015 **Arrow Function**:

```javascript
const double = (number) => number * 2;
```

In beiden Fällen liefert uns die eben deklarierte Funktion beim Aufruf von bspw. `double(5)` als Ergebnis `10` zurück!

Aber es gibt noch einen weiteren gewichtigen Vorteil, der bei der Arbeit mit React sehr nützlich sein wird: Arrow Functions haben keinen eigenen Constructor, können also nicht als Instanz in der Form `new MyArrowFunction()` erstellt werden, und binden auch kein eigenes `this` sondern erben `this` aus ihrem **Parent Scope**. Insbesondere Letzteres wird noch sehr hilfreich werden.

Auch das klingt fürchterlich kompliziert, lässt sich aber anhand eines einfachen Beispiels ebenfalls recht schnell erklären. Nehmen wir an, wir definieren einen Button, der die aktuelle Zeit in ein `div` schreiben soll, sobald ich ihn anklicke. Eine typische Funktion in ES5 könnte wie folgt aussehen:

```javascript
function TimeButton() {
  var button = document.getElementById('btn');
  var self = this;
  this.showTime = function() {
    document.getElementById('time').innerHTML = new Date();
  };
  button.addEventListener('click', function() {
    self.showTime();
  });
}
```

Da die als **Event Listener** angegebene Funktion keinen Zugriff auf ihren **Parent Scope**, also den **TimeButton** hat, speichern wir hier hilfsweise `this` in der Variable `self`. Kein unübliches Muster in ES5. Alternativ könnte man auch den Scope der Funktion explizit an `this` binden und dem **Event Listener** beibringen, in welchem Scope sein Code ausgeführt werden soll:

```javascript
function TimeButton() {
  var button = document.getElementById('btn');
  this.showTime = function() {
    document.getElementById('time').innerHTML = new Date();
  };
  button.addEventListener(
    'click',
    function() {
      this.showTime();
    }.bind(this)
  );
}
```

Hier spart man sich zumindest die zusätzliche Variable `self`. Auch das ist möglich, aber nicht besonders elegant.

An dieser Stelle kommt nun die **Arrow Function** ins Spiel, die, wie eben erwähnt, `this` aus ihrem **Parent Scope** erhält, also in diesem Fall aus unserer `TimeButton`-Instanz:

```javascript
function TimeButton() {
  var button = document.getElementById('btn');
  this.showTime = function() {
    document.getElementById('time').innerHTML = new Date();
  };
  button.addEventListener('click', () => {
    this.showTime();
  });
}
```

Und schon haben wir im **Event Listener** Zugriff auf `this` des überliegenden Scopes!

Keine `var self = this` Akrobatik mehr und auch kein `.bind(this)`. Wir können innerhalb des Event Listeners so arbeiten als befänden wir uns noch immer im `TimeButton` Scope! Das ist später insbesondere bei der Arbeit mit umfangreichen React-Komponenten mit vielen eigenen Class Properties und Methods hilfreich, da es Verwirrungen vorbeugt und nicht immer wieder einen neuen Scope erzeugt.

## Neue Methoden bei Strings, Arrays und Objekten

Mit ES2015 erhielten auch eine ganze Reihe neue statische und Prototype-Methoden Einzug in JavaScript. Auch wenn die meisten davon nicht direkt relevant sind für die Arbeit mit React, erleichtern sie die Arbeit aber gelegentlich doch ungemein, weshalb ich hier ganz kurz auf die wichtigsten eingehen möchte.

### String-Methoden

Hat man in der Vergangenheit auf `indexOf()` oder reguläre Ausdrücke gesetzt, um zu prüfen ob ein String einen bestimmten Wert enthält, mit einem bestimmten Wert anfängt oder aufhört, bekommt der String Datentyp nun seine eigenen Methoden dafür.

Dies sind:

```javascript
string.includes(value);
string.startsWith(value);
string.endsWith(value);
```

Zurückgegeben wird jeweils ein Boolean, also `true` oder `false.` Möchte ich wissen ob mein String `Beispiel`ein `eis` enthält, prüfe ich ganz einfach auf

```javascript
'Beispiel'.includes('eis');
```

Analog verhält es sich mit `startsWith`:

```javascript
'Beispiel'.startsWith('Bei');
```

… wie auch mit `endsWith`:

```javascript
'Beispiel'.endsWith('spiel');
```

Die Methode arbeitet dabei case-sensitive, unterscheidet also zwischen Groß- und Kleinschreibung.

Zwei weitere hilfreiche Methoden, die mit ES2015 Einzug in JavaScript erhalten haben, sind `String.prototype.padStart()` und `String.prototype.padEnd()`. Diese Methoden könnt ihr nutzen, um einen String auf eine gewisse Länge zu bringen, indem ihr am Anfang \(`.padStart()`\) oder am Ende \(`.padEnd()`\) Zeichen hinzufügt bis die angegebene Länge erreicht ist. Dabei gibt der erste Parameter die gewünschte Länge an, der optionale zweite Parameter das Zeichen mit dem ihr den String bis zu dieser Stelle auffüllen wollt. Gebt ihr keinen zweiten Parameter an, wird standardmäßig ein Leerzeichen benutzt.

Hilfreich ist das bspw. wenn ihr Zahlen auffüllen wollt, so dass diese immer einheitlich dreistellig sind:

```javascript
'7'.padStart(3, '0'); // 007
'72'.padStart(3, '0'); // 072
'132'.padStart(3, '0'); // 132
```

`String.prototype.padEnd()` funktioniert nach dem gleichen Muster, mit dem Unterschied, dass es euren String am Ende auffüllt, nicht am Anfang.

### Arrays

Bei den Array-Methoden gibt es sowohl neue statische als auch Methoden auf dem Array-Prototype. Was bedeutet dies? Prototype-Methoden arbeiten „mit dem Array“ als solches, also mit einer bestehenden **Array-Instanz**, statische Methoden sind im weiteren Sinne Helper-Methoden, die gewisse Dinge tun, die „mit Arrays zu tun haben“.

#### Statische Array-Methoden

Fangen wir mit den statischen Methoden an:

```javascript
Array.of(3); // [3]
Array.of(1, 2, 3); // [1, 2 ,3]
Array.from('Example'); // ['E', 'x', 'a', 'm', 'p', 'l', 'e']
```

`Array.of()` erstellt eine neue Array-Instanz aus einer beliebigen Anzahl an Parametern, unabhängig von deren Typen. `Array.from()` erstellt ebenfalls eine Array-Instanz, allerdings aus einem „array-ähnlichen“ iterierbaren Objekt. Das wohl griffigste Beispiel für ein solches Objekt ist eine `HTMLCollection` oder eine `NodeList`. Solche erhält man bspw. bei der Verwendung von DOM-Methoden wie `getElementsByClassName()` oder dem moderneren `querySelectorAll()`. Diese besitzen selbst keine Methoden wie `.map()` oder `.filter()`. Möchte man über eine solche also iterieren, muss man sie erst einmal in einen Array konvertieren. Dies geht mit ES2015 nun ganz einfach durch die Verwendung von `Array.from()`.

```javascript
const links = Array.from(document.querySelectorAll('a'));
Array.isArray(links); // true
```

#### Methoden auf dem Array-Prototypen

Die Methoden auf dem Array-Prototypen können **direkt auf eine Array-Instanz** angewendet werden. Die gängigsten während der Arbeit mit React und insbesondere später mit Redux sind:

```javascript
Array.find(func);
Array.findIndex(func);
Array.includes(value);
```

Die `Array.find()`-Methode dient, wie der Name es erahnen lässt, dazu, das **erste** Element eines Arrays zu finden, das bestimmte Kriterien erfüllt, die mittels der als ersten Parameter übergebenen Funktion geprüft werden.

```javascript
const numbers = [1, 2, 5, 9, 13, 24, 27, 39, 50];
const biggerThan10 = numbers.find((number) => number > 10); // 13

const users = [
  { id: 1, name: 'Manuel' },
  { id: 2, name: 'Bianca' },
  { id: 3, name: 'Brian' },
];

const userWithId2 = users.find((user) => user.id === 2);
// { id: 2, name: 'Bianca'}
```

Die `Array.findIndex()`-Methode folgt der gleichen Signatur, liefert aber anders als die `Array.find()`-Methode nicht das gefundene Element selbst zurück, sondern nur dessen Index im Array. In den obigen Beispielen wären dies also `4` im ersten Beispiel, sowie `1` im zweiten.

Die in ES2016 neu dazu gekommene Methode `Array.includes()` prüft, ob ein Wert innerhalb eines Array existiert und gibt uns **endlich** einen Boolean zurück. Wer selbiges in der Vergangenheit mal mit `Array.indexOf()` realisiert hat, wird sich erinnern wie umständlich es war. Nun also ein simples `Array.includes()`:

```javascript
[1, 2, 3, 4, 5].includes(4); // true
[1, 2, 3, 4, 5].includes(6); // false
```

Aufgepasst: die Methode ist case-sensitive. `['a', 'b'].includes('A')` gibt also `false` zurück.

### Objekte

#### Statische Objekt-Methoden

Natürlich haben auch Objekte eine Reihe neuer Methoden und anderer schöner Möglichkeiten spendiert bekommen. Die wichtigsten im Überblick:

```javascript
Object.assign(target, source[, source[,...]]);
Object.entries(Object)
Object.keys(Object)
Object.values(Object)
Object.freeze(Object)
```

Wieder der Reihe nach. Die wohl nützlichste ist aus meiner Sicht `Object.assign()`. Damit ist es möglich, die Eigenschaften eines Objekts oder auch mehrerer Objekte zu einem bestehenden Objekt hinzuzufügen \(sozusagen ein Merge\). Die Methode gibt dabei das Ergebnis als Objekt zurück. Allerdings findet dabei auch eine Mutation des **Ziel-Objekts** statt, weswegen die Methode mit Bedacht benutzt werden sollte. Beispiele sagen mehr als Worte, bitteschön:

```javascript
const user = { id: 1, name: 'Manuel' };
const modifiedUser = Object.assign(user, { role: 'Admin' });
console.log(user);
// -> { id: 1, name: 'Manuel', role: 'Admin' }
console.log(modifiedUser);
// -> { id: 1, name: 'Manuel', role: 'Admin' }
console.log(user === modifiedUser);
// -> true
```

Hier fügen wir also die Eigenschaft `role` aus dem Objekt im zweiten Parameter der `Object.assign()`-Methode zum bestehenden **Ziel-Objekt** hinzu.

Da React dem Prinzip von **Pure Functions** folgt, das sind Funktionen die in sich geschlossen sind und ihre Eingabeparameter nicht modifizieren, sollten derartige Mutationen möglichst vermieden werden. Dies können wir umgehen, indem wir als ersten Parameter einfach ein Object-Literal übergeben:

```javascript
const user = { id: 1, name: 'Manuel' };
const modifiedUser = Object.assign({}, user, { role: 'Admin' });
console.log(user);
// -> { id: 1, name: 'Manuel' }
console.log(modifiedUser);
// -> { id: 1, name: 'Manuel', role: 'Admin' }
console.log(user === modifiedUser);
// -> false
```

Durch die Verwendung eines neu erstellten Objekts als Ziel-Objekt bekommen wir hier eben auch als Rückgabewert ein anderes Objekt als im ersten Beispiel. In einigen Fällen kann es gewünscht sein, das **Ziel-Objekt** zu mutieren statt ein neues Objekt zu erstellen, während der Arbeit mit React ist dies jedoch in den deutlich überwiegenden Fällen nicht so.

Die Methode verarbeitet dabei auch beliebig viele Objekte als Parameter. Gibt es gleichnamige Eigenschaften in einem Objekt, haben spätere Eigenschaften Vorrang:

```javascript
const user = { id: 1, name: 'Manuel' };
const modifiedUser = Object.assign(
  {},
  user,
  { role: 'Admin' },
  { name: 'Nicht Manuel', job: 'Developer' }
);
console.log(modifiedUser);
// -> { id: 1, name: 'Nicht Manuel', role: 'Admin', job: 'Developer' }
```

Die drei statischen Objekt-Methoden `Object.entries()`, `Object.keys()` und `Object.values()` funktionieren im Grunde sehr ähnlich, sie liefern zu einem übergebenen Objekt die Eigenschaften \(`keys`\), die Werte \(`values`\) oder die Einträge \(`entries`\) als **Array** zurück, wobei die **Entries** ein verschachteltes Array sind in der Form`[[key, value], [key2, values2], …]`.

Angewendet auf unser obiges Beispiel hat dies also folgende Return-Values zum Ergebnis:

```javascript
Object.keys({ id: 1, name: 'Manuel' });
// -> ['id', 'name']
Object.values({ id: 1, name: 'Manuel' });
// -> [1, 'Manuel']
Object.entries({ id: 1, name: 'Manuel' });
// -> [['id', 1], ['name', 'Manuel']]
```

Zuletzt schauen wir uns `Object.freeze()` an. Auch diese Methode ist ziemlich selbsterklärend und tut genau, was der Name vermuten lässt: sie friert ein Objekt ein, untersagt es dem Entwickler also, neue Eigenschaften hinzuzufügen und alte Eigenschaften zu löschen oder auch nur zu verändern. Auch dies ist im Umgang mit den Objekten, die in React in den meisten Fällen unveränderlich sind \(oder zumindest sein sollten\), unglaublich praktisch.

```javascript
const user = Object.freeze({ id: 1, name: 'Manuel' });
user.id = 2;
delete user.name;
user.role = 'Admin';
console.log(user);
// -> { id: 1, name: 'Manuel' }
```

Ein mittels `Object.freeze()` erstelltes Objekt bietet auch guten Schutz vor versehentlicher Mutation mittels der oben beschriebenen, ebenfalls neuen `Object.assign()`-Methode. Wird versucht, ein mittels `Object.freeze()` erstelltes Objekt per `Object.assign()` zu modifizieren, führt dies unweigerlich zu seinem `TypeError`.

#### Syntax-Erweiterungen und Vereinfachungen

Die letzten Änderungen an der Funktionsweise von Objekten sind keine Methode, sondern Syntax-Erweiterungen.

Die erste sind die **Computed Properties** \(also etwa _berechnete Eigenschaften_\). Dahinter verbirgt sich die Möglichkeit, Ausdrücke \(bzw. deren Werte\) als Objekt-Eigenschaften zu verwenden. Wollte man bspw. früher eine Eigenschaft in einem Objekt setzen, lief das meist so, dass man das Objekt erstellte \(bspw. als **Object-Literal** `{}` oder per `Object.create()`\), dieses einer Variablen zuwies und anschließend die neue Eigenschaft zum Objekt hinzufügte:

```javascript
const nationality = 'german';
const user = {
  name: 'Manuel',
};
user[nationality] = true;
console.log(user);
// -> { name: 'Manuel', german: true };
```

**ES2015** erlaubt uns nun, Ausdrücke direkt als Objekt-Eigenschaft zu nutzen, indem wir sie in eckige Klammern `[]` setzen. Dadurch sparen wir uns den Umweg, nachträglich noch Eigenschaften zum bereits erstellten Objekt hinzuzufügen:

```javascript
const nationality = 'german';
const user = {
  name: 'Manuel',
  [nationality]: true,
};
console.log(user);
// -> { name: 'Manuel', german: true };
```

Das Beispiel ist aus Gründen der einfacheren Verständlichkeit ein simples, doch die Verwendungsmöglichkeiten werden später mitunter noch deutlich komplexer und schaffen uns viele Möglichkeiten um sauberen und gut verständlichen Code zu schreiben, insbesondere wenn es um **JSX** geht.

Die letzte nennenswerte Neuerung bei Objekten sind die sogenannten **Shorthand Property Names**. Diese ersparen uns eine Menge unnötige Schreibarbeit. Nicht erst seit React kennt man es, dass man auf Code wie den folgenden stößt:

```javascript
const name = 'Manuel';
const job = 'Developer';
const role = 'Author';

const user = {
  name: name,
  job: job,
  role: role,
};
```

Ziemlich viele unnötige Dopplungen wenn man sich das mal genau anschaut, oder? Genau diese nimmt uns die **Shorthand Property Name Syntax** in **ES2015** endlich ab. Und so reicht es, nur noch die Variable zu schreiben, wenn diese den Namen der entsprechenden Objekt-Eigenschaft trägt. Im obigen Fall also:

```javascript
const name = 'Manuel';
const job = 'Developer';
const role = 'Author';

const user = {
  name,
  job,
  role,
};
```

Jep. Seit **ES2015** führen beide Schreibweisen tatsächlich zum identischen Objekt! Dabei kann die Shorthand Syntax auch problemlos mit der herkömmlichen Syntax kombiniert werden:

```javascript
const name = 'Manuel';
const job = 'Developer';

const user = {
  name,
  job,
  role: 'Author',
};
```

## Classes

Mit **ES2015** fanden auch **Klassen** Einzug in JavaScript. **Klassen** kennt man eher aus objektorientierten Sprachen wie Java, in JavaScript gab es sie so explizit bisher allerdings noch nicht. Zwar war es auch schon vorher durch die Verwendung von Funktionsinstanzen möglich, objektorientiert zu arbeiten und durch die `prototype`-Eigenschaft einer Funktion eigene Methoden und Eigenschaften zu definieren, dies war verglichen mit echten objektorientierten Sprachen jedoch sehr mühsam und schreiblastig.

Dies ändert sich mit **ES2015**, wo es nun erstmals auch Klassen gibt, die mittels `class`-Keyword definiert werden. Das ist für uns insofern interessant, als React, obwohl es viele Prinzipien der funktionalen Programmierung \(**Functional Programming**\) verfolgt, gleichzeitig auch in einem wesentlichen Punkt auf ES2015-Klassen setzt;- nämlich bei der Erstellung von Komponenten, in diesem Fall speziell von **Class Components**. Auch vor der Einführung von ES2015 Klassen war es natürlich möglich, Komponenten in React zu definieren, dazu gab es eine eigene `createClass()`-Methode. Diese ist aber mittlerweile nicht mehr Teil des React Cores und sollte möglichst auch nicht mehr verwendet werden.

Eine Klasse besteht aus einem Namen, kann \(optional\) einen **Constructor** haben, der bei der Erstellung einer Klassen-Instanz aufgerufen wird, und beliebig viele Klassen-Methoden besitzen.

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

Auch eine `super()`-Funktion kennt die neu eingeführte **ES2015**-Klasse, um damit den **Constructor** ihrer Elternklasse aufzurufen. Im Falle von React ist dies immer notwendig, wenn ich in meiner eigenen Klasse eine `constructor`-Methode definiere. Diese muss dann `super()` aufrufen und ihre `props` an den Constructor der `React.Component`-Klasse weiterzugeben:

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
  }
}
```

Tätet ihr das nicht, wäre `this.props` innerhalb eurer Komponente `undefined` und ihr könntet nicht auf die Props eurer Komponente zugreifen. Grundsätzlich sollte die Verwendung eines Constructors aber in den allermeisten Fällen nicht nötig sein, denn React stellt eigene sog. **Lifecycle-Methoden** bereit, die der Verwendung des Constructors vorzuziehen sind.

## Rest und Spread Operators und Destrukturierung

Eine weitere deutliche Vereinfachung ist die Einführung der sog. Rest und Spread Operators für Objekte und Arrays. Streng genommen handelt es sich bei deren Verwendung in Kombination mit Objekten noch gar nicht um ES2015 Features, da diese sich noch in der Diskussion befinden und noch gar nicht endgültig in die ECMAScript-Spezifikation aufgenommen wurden. Dies ändert sich erst mit ES2018. Eingeführt wurden Rest und Spread in ES2015 erstmals für Arrays. Durch die Verwendung von Babel ist aber auch heute bereits die Nutzung mit Objekten möglich und für gewöhnlich wird davon in React-basierten Projekten auch rege Gebrauch gemacht.

Aber was ist das jetzt überhaupt? Fangen wir mit dem Spread Operator an.

### Spread Operator

Der Spread Operator sorgt dafür, Werte sozusagen „auszupacken“. Wollte man in ES5 mehrere Argumente aus einem Array an eine Funktion übergeben, geschah das bisher meist über `Function.prototype.apply()`:

```javascript
function sumAll(number1, number2, number3) {
  return number1 + number2 + number3;
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
  return number1 + number2 + number3;
}
var myArray = [1, 2, 3];
sumAll(...myArray);
```

**Ausgabe:**

{% hint style="info" %}
6
{% endhint %}

Ich muss also nicht mehr den Umweg über `apply()` gehen. Aber nicht nur bei Funktionsargumenten ist das hilfreich. Ich kann ihn auch nutzen, um bspw. auf einfache Art und Weise zwei Arrays zu einem einzigen zu kombinieren:

```javascript
const greenFruits = ['kiwi', 'apple', 'pear'];
const redFruits = ['strawberry', 'cherry', 'raspberry'];
const allFruits = [...greenFruits, ...redFruits];
```

**Ergebnis:**

{% hint style="info" %}
`['kiwi', 'apple', 'pear', 'strawberry', 'cherry', 'raspberry']`
{% endhint %}

Dabei wird ein neues Array erstellt, welches alle Werte sowohl aus dem `greenFruits` als auch aus dem `redFruits` Array enthält. Doch nicht nur das: dabei wird auch ein gänzlich neues Array erstellt und nicht bloß eine Referenz der beiden alten. Dies wird im weiteren Verlauf, wenn wir an die **readonly**-Anforderung unserer Props noch sehr nützlich sein. Und so kann man den Spread Operator auch verwenden um eine einfache Kopie eines Arrays zu erstellen:

```javascript
const users = ['Manuel', 'Chris', 'Ben'];
const selectedUsers = [...users];
```

`selectedUsers` ist in diesem Fall eine Kopie unseres `users` Arrays mit all seinen Werten. Verändern wir nun das Users Array, hat dies auf unser `selectedUsers` Array keinerlei Auswirkungen.

Bei Objekten verhält sich der Spread Operator sehr ähnlich. Hier werden statt der einzelnen Werte alle Eigenschaften eines Objekts die „enumerable“ \(aufzählbar\) sind, also ganz grob gesagt bei der Verwendung in einer `for(… in …)` Schleife angezeigt werden würden.

Hier eignet sich der Spread Operator hervorragend um neue Objekte zu erstellen:

```javascript
const globalSettings = { language: 'en-US', timezone: 'Berlin/Germany' };
const userSettings = { mutedUsers: ['Manuel'] };
const allSettings = { ...globalSettings, ...userSettings };
console.log(allSettings);
```

**Ausgabe:**

```javascript
{
  language: 'en-US',
  timezone: 'Berlin/Germany',
  mutedUsers: ['Manuel'],
}
```

Die Eigenschaften beider Objekte finden sich dabei im neu erstellten, kombinierten `allSettings`-Objekt wieder. Dabei ist der **Spread Operator** hier nicht auf zwei Objekte beschränkt, sondern kann beliebige weitere Objekte zu einem einzelnen neuen Objekt kombinieren. Auch die Kombination mit einzelnen Eigenschaften ist möglich:

```javascript
const settings = {
  ...userSettings,
  showWarnings: true,
};
```

Befinden sich in beiden Objekten Eigenschaften mit dem gleichen Namen, hat das letztgenannte Objekt Vorrang:

```javascript
const globalSettings = { language: 'en-US', timezone: 'Berlin/Germany' };
const userSettings = { language: 'de-DE' };
const allSettings = { ...globalSettings, ...userSettings };
console.log(allSettings);
```

**Ausgabe:**

```text
{
  language: 'de-DE',
  timezone: 'Berlin/Germany',
}
```

Das zuletzt genannte `userSettings`-Objekt überschreibt hier die gleichnamige Eigenschaft `language`, die sich auch im `globalSettings`-Objekt befindet. Der Spread Operator funktioniert hier ganz ähnlich wie die in ES2015 neu eingeführte Objekt-Methode `Object.assign()`. Auch diese wird in Anwendungen, die auf ES2015+ basieren, gelegentlich genutzt.

Allerdings gibt es hier den nennenswerten Unterschied, dass sie ein bestehendes Objekt mutiert und nicht per se ein neues Objekt generiert, wie das die Object-Spread-Variante tut. Und Mutation ist bezogen auf React-Komponenten und ihre Props eben das, was wir ja nicht wollen. Dennoch der Vollständigkeit halber ein kurzes Beispiel.

#### Objekte kombinieren mittels Object.assign\(\)

`Object.assign()` nimmt beliebig viele Objekte entgegen und kombiniert diese zu einem einzigen Objekt:

```javascript
const a = { a: 1 };
const b = { b: 2 };
const c = { c: 3 };
console.log(Object.assign(a, b, c));
```

**Ausgabe:**

```javascript
{a: 1, b: 2, c: 3}
```

Die Funktion gibt uns also ein neues Objekt zurück, in dem alle 3 übergebenen Objekte zu einem einzigen kombiniert wurden. Aber ist das wirklich ein neues Objekt? **Nein!** Lassen wir uns doch anschließend mal `a`, `b` und `c` in der Console ausgeben:

```javascript
console.log(a);
console.log(b);
console.log(c);
```

**Ausgabe:**

```javascript
{a: 1, b: 2, c: 3}
{b: 2}
{c: 3}
```

Wir stellen also fest: `Object.assign()` hat uns nicht wirklich ein komplett neues Objekt aus den 3 übergebenen Objekten erstellt sondern lediglich die Eigenschaften des zweiten und dritten Objekts zum ersten übergebenen Objekt hinzugefügt. Und das ist, in Bezug auf **Pure Functions** und **Immutable Objects**, äußerst schlecht und in jedem Fall zu vermeiden!

Hier gibt es aber einen einfachen Trick um Objekte mittels `Object.assign()` zu kombinieren und dabei gleichzeitig ein neues Objekt zu erstellen. Dazu übergebt ihr der Funktion als erstes Argument ein leeres Object-Literal `{}`:

```text
Object.assign({}, a, b, c);
```

… und schon werden dem neu erstellten `{}` Objekt die Eigenschaften unserer Objekte `a`, `b` und `c` übertragen, die bestehenden Objekte `a`, `b` und `c` bleiben dabei unangetastet!

### Destructuring Assignment / destrukturierende Zuweisung

Bevor ich zum **Rest Operator** komme, der logisch sehr eng mit dem **Spread Operator** in Verbindung steht und meist mit diesem in einem Atemzug genannt wird, möchte ich auf das **Destructuring Assignment** \(kurz: **Destructuring**\) oder eben die **destrukturierende Zuweisung** \(kurz: **Destrukturierung**\), wie der schöne Begriff auf Deutsch heißt, eingehen. Ich werde hier wie so oft beim englischen Begriff bleiben, da ich den deutschen Begriff selbst in deutschsprachigen Texten selten bisher gelesen habe.

Mittels **Destructuring** ist es möglich, einzelne Elemente aus Objekten oder Arrays zu extrahieren und Variablen zuzuweisen. Eine weitere **deutliche** Syntax-Erweiterung, die uns ES2015 hier beschert hat.

<span class="force-break-before"></span>

#### Destructuring von Arrays

Stellen wir uns vor, wir möchten aus einem geordneten Array mit den Olympia-Teilnehmern im 100m-Lauf jeweils den Gewinner der Gold-, Silber- und Bronzemedaille in eine eigene Variable schreiben. Auf herkömmlichem \(also ES5\) Weg funktionierte das bisher folgendermaßen:

```javascript
const athletes = [
  'Usain Bolt',
  'Andre De Grasse ',
  'Christophe Lemaitre ',
  'Adam Gemili',
  'Churandy Martina',
  'LaShawn Merritt',
  'Alonso Edward',
  'Ramil Guliyev',
];

const gold = athletes[0];
const silver = athletes[1];
const bronze = athletes[2];
```

Dank **Destructuring** können wir dies auf ein einzelnes Statement verkürzen:

```javascript
const [gold, silver, bronze] = athletes;
```

Die Werte der Array-Elemente `0`, `1` und `2` befinden sich dann der Reihe nach in den Variablen `gold`, `silver` und `bronze`, wie auch im ersten Beispiel, jedoch mit deutlich weniger Schreibarbeit!

Dies funktioniert überall wo wir mit einem Array auf der rechten Seite \(also hinter dem `=` Zeichen\) einer Zuweisung arbeiten, also auch wenn wir diesen als `return`-Wert aus einer Funktion erhalten:

```javascript
const getAllAthletes = () => {
  return [
    'Usain Bolt',
    'Andre De Grasse ',
    'Christophe Lemaitre ',
    'Adam Gemili',
    'Churandy Martina',
    'LaShawn Merritt',
    'Alonso Edward',
    'Ramil Guliyev',
  ];
};

const [gold, silver, bronze] = getAllAthletes();
```

Die Arrow Function gibt uns hier ein Array mit allen Athleten zurück, dementsprechend können wir hier direkt beim Aufruf bereits das Destructuring nutzen und müssen den `return`-Wert bspw. nicht erst eigens in einer temporären Variable speichern.

Möchten wir auf diese Weise einzelne Elemente des Arrays auslassen, ist das buchstäblich durch Auslassen des entsprechenden Wertes möglich:

```javascript
const [, silver, bronze] = athletes;
```

Hier würden wir auf das Deklarieren einer Variable `gold` verzichten und nur die Gewinner der Silber- und Bronze-Medaille in entsprechenden Variablen speichern.

Doch nicht nur bei der offensichtlichen Variablenzuweisung mittels `let` oder `const` kann **Array Destructuring** verwendet werden. Auch bei weniger offensichtlichen Zuweisungen, wie bei der Übergabe von Funktionsargumenten in Form eines Arrays.

```javascript
const logWinners = (athletes) => {
  const gold = athletes[0];
  const silver = athletes[1];
  const bronze = athletes[2];
  console.log('Winners of Gold, Silver and Bronze are', gold, silver, bronze);
};
```

Das geht einfacher:

```javascript
const logWinners = ([gold, silver, bronze]) => {
  console.log('Winners of Gold, Silver and Bronze are', gold, silver, bronze);
};
```

Hier reichen wir das Array in unsere `logWinners()`-Funktion herein und statt für jeden Medaillengewinner eine Variable pro Zeile zu deklarieren, nutzen wir auch in diesem Fall ganz einfach wieder die Destructuring-Methode von oben.

#### Destructuring von Objekten

Das Prinzip des **Destructurings** ist nicht allein auf Arrays beschränkt. Auch Objekte können auf diese Art Variablen zugeordnet werden, die standardmäßig mit dem Namen einer Eigenschaft übereinstimmen.

Die Schreibweise ist dabei ähnlich der bei Arrays, mit dem Unterschied, dass wir die Werte nicht anhand ihrer Position im Objekt zuweisen sondern anhand ihres Eigenschafts-Namens. Außerdem setzen wir die Zuweisung in die für Objekte typischen geschweiften Klammern statt in eckige Klammern.

```javascript
const user = {
  firstName: 'Manuel',
  lastName: 'Bieh',
  job: 'JavaScript Developer',
  image: 'manuel.jpg',
};
const { firstName } = user;
```

Die Variable `firstName` enthält nun den Wert aus `user.firstName`!

Das Object Destructuring ist eins der wohl meist verwendeten Features, das man in den meisten React-Komponenten findet. Es erlaubt uns, einzelne Props in Variablen zu schreiben und an entsprechenden Stellen im JSX auf unkomplizierte Weise zu verwenden.

Nehmen wir an dieser Stelle einmal die folgende Stateless Functional Component als Beispiel:

```javascript
const UserPersona = (props) => {
  return (
    <div>
      <img src={props.image} alt="User Image" />
      {props.firstName} {props.lastName}
      <br />
      <strong>{props.job}</strong>
    </div>
  );
};
```

Die ständige Wiederholung von `props` vor jeder Eigenschaft erschwert die Lesbarkeit der Komponente unnötig. Hier können wir uns Object Destructuring zu Nutze machen um einmalig eine Variable für jede Eigenschaft unserer `props` zu deklarieren.

```javascript
const UserPersona = (props) => {
  const { firstName, lastName, image, job } = props;
  return (
    <div>
      <img src={image} alt="User Image" />
      {firstName} {lastName}
      <br />
      <strong>{job}</strong>
    </div>
  );
};
```

Damit wirkt unsere Komponente schon deutlich aufgeräumter und lesbarer. Doch es geht noch einfacher. Wie auch bei Arrays ist es möglich, Objekte direkt bei der Übergabe als Funktionsargument zu destrukturieren. Statt des `props`-Arguments nutzen wir dafür das **Destructuring Assignment** direkt:

```javascript
const UserPersona = ({ firstName, lastName, image, job }) => (
  <div>
    <img src={image} alt="User Image" />
    {firstName} {lastName}
    <br />
    <strong>{job}</strong>
  </div>
);
```

Als Bonus nutzen wir hier sogar die direkte Rückgabe aus der Funktion ohne geschweifte Klammern und explizites `return` Statement aus dem Kapitel über Arrow Functions, da wir ja nun mit unserem auf 5 Zeilen reduzierten **JSX** einen Ausdruck haben, den wir direkt aus der **Arrow Function** zurückgeben können.

Während der Arbeit mit React trifft man ständig auf derartige Syntax in **SFCs**, auch bei **Class Components** findet man sehr häufig zu Beginn der `render()`-Methode einer Komponente ein ähnliches Destructuring Assignment in der Form:

```javascript
render() {
  const { firstName, lastName, image, job } = this.props;
  // weiterer Code
}
```

Es ist euch natürlich hinterher freigestellt, ob ihr das so macht oder innerhalb der Funktion einfach weiterhin direkt auf `this.props.firstName` zugreift. Dieses Muster hat sich aber mittlerweile zu einer Art Best Practice entwickelt und wurde in den meisten Projekten so gehandhabt, da es den Code am Ende in den meisten Fällen lesbarer werden lässt und auch leichter verständlich ist.

**Umbenennung von Eigenschaften beim Destructuring**

Manchmal ist es notwendig, Eigenschaften umzubenennen; entweder weil es bereits Variablen mit dem selben Namen gibt oder die Eigenschaft kein gültiger Variablenname wäre. All das ist denkbar und möglich. Und ES2015 bietet uns auch eine Lösung dafür.

```javascript
const passenger = {
  name: 'Manuel Bieh',
  class: 'economy',
};
```

Das obige `passenger` Objekt enthält die Eigenschaft class, die als Name für eine Eigenschaft gültig ist, als Name für eine Variable jedoch nicht. Ein direktes Destructuring wäre hier also nicht möglich und würde zu einem Fehler führen:

```javascript
const { name, class } = passenger;
```

{% hint style="danger" %}
Uncaught SyntaxError: Unexpected token }
{% endhint %}

Um hier den Namen der Variable umzubenennen, muss der Eigenschaft der neue Name getrennt durch einen Doppelpunkt `:` übergeben werden. Ein korrektes **Destructuring Assignment** wäre also in diesem Fall in etwa folgendes:

```javascript
const { name, class: ticketClass } = passenger;
```

Hier schreiben wir den Wert der `class` Eigenschaft in eine Variable `ticketClass`, was anders als `class` ein gültiger Name für eine Variable ist. Der Name des Passagiers landet dabei ganz gewöhnlich in einer Variable mit dem Namen `name`.

**Standardwerte beim Destructuring vergeben**

Auch die Vergabe von Standardwerten beim **Destructuring** ist möglich! Ist im Objekt, das destrukturiert wird, eine Eigenschaft nicht definiert, wird stattdessen der Default verwendet. Ähnlich wie bei der Umbenennung wird dabei die jeweilige Eigenschaft wie gehabt vorangestellt, jedoch gefolgt von einem Gleich-Zeichen und dem entsprechenden Standardwert:

```javascript
const { name = 'Unknown passenger' } = passenger;
```

Der Wert von `name` wäre nun `Unknown passenger`, wenn im `passenger`-Objekt keine Eigenschaft `name` existiert oder deren Wert `undefined` ist. Existiert diese hingegen, ist aber leer \(also bspw. ein leerer String oder `null`\) wird der Standardwert **nicht** an dessen Stelle verwendet!

**Kombination von Umbenennung und Standardwerten**

Jetzt wird es verrückt, denn auch das ist möglich. Die Umbenennung von Eigenschaften in Variablennamen bei gleichzeitiger Verwendung von Standardwerten. Die Syntax dafür ist allerdings etwas, wo man bei der ersten Begegnung sicherlich einen Moment länger hinschauen muss. Wir bleiben wieder bei unserem `passenger`-Objekt aus den Beispielen zuvor. Anforderung ist nun die Zuweisung der `name` Eigenschaft zu einer Variable mit dem Namen `passengerName`, die den Wert `Unknown Passenger` tragen soll, wenn kein Name vorhanden ist. Außerdem möchten wir weiterhin `class` in `ticketClass` umbenennen und den Passagier gleichzeitig in `Economy` einordnen, sollte es im entsprechenden Objekt keine `class` Eigenschaft geben.

```javascript
const {
  name: passengerName = 'Unknown passenger',
  class: ticketClass = 'economy',
} = passenger;
```

Hier besitzen die Variablen `passengerName` und `ticketClass` die Werte `Unknown passenger` und `economy`, wenn diese nicht im destrukturierten Objekt existieren. Doch Vorsicht: Das Objekt selbst darf nicht null sein, andernfalls bekommen wir vom JavaScript Interpreter einen unschönen Fehler geworfen:

```javascript
const {
  name: passengerName = 'Unknown passenger',
  class: ticketClass = 'economy',
} = null;
```

{% hint style="danger" %}
Uncaught TypeError: Cannot destructure property \`name\` of 'undefined' or 'null'.
{% endhint %}

Hier gibt es einen unsauberen aber doch oft praktischen Trick um sicherzustellen, dass das Objekt selbst nicht `null` oder `undefined` ist. Dazu machen wir uns den **Logical OR Operator** zu nutze und verwenden ein leeres Objekt als Fallback, falls unser eigentliches Objekt eben `null` oder `undefined` ist:

```javascript
const {
  name: passengerName = 'Unknown passenger',
  class: ticketClass = 'economy',
} = passenger || {};
```

Mit dem angehängten `|| {}` sagen wir: ist das `passenger` Objekt **falsy**, nutze stattdessen ein leeres Objekt. Die vermutlich „sauberere“ Variante wäre es vorab zu prüfen ob `passenger` auch wirklich ein Objekt ist und das Destructuring nur dann auszuführen. Die Variante mit dem **Logical OR** Fallback ist allerdings schön kurz und dürfte in vielen Fällen ausreichen.

**Destructuring** kann übrigens auch problemlos mit dem **Spread Operator** zusammen verwendet werden:

```javascript
const globalSettings = { language: 'en-US' };
const userSettings = { timezone: 'Berlin/Germany' };
const { language, timezone } = { ...globalSettings, ...userSettings };
```

Hier wird zuerst der **Spread Operator** aufgelöst, also ein Objekt mit allen Eigenschaften aus den beiden Objekten `globalSettings` und `userSettings` erzeugt und anschließend per **Destructuring Assignment** entsprechenden Variablen zugewiesen.

### Rest Operator

Der Rest Operator dient dazu, sich um die verbliebenen Elemente aus einem **Destructuring** und in **Funktionsargumenten** zu kümmern. Daher der Name: der Operator kümmert sich um den verbliebenen **„Rest“**. Wie auch schon der **Spread Operator** wird auch der **Rest Operator** mit drei Punkten `…` eingeleitet, jedoch nicht auf der **rechten** Seite einer Zuweisung, sondern auf der **linken**. Anders als beim Spread Operator kann es pro Ausdruck jedoch nur jeweils **einen** Rest Operator geben!

Schauen wir uns zuerst einmal den **Rest Operator** bei Funktionsargumenten an. Sagen wir, wir möchten nun eine Funktion schreiben, die beliebig viele Argumente empfängt. Hier möchten wir natürlich auch auf all diese Argumente zugreifen können, egal ob das 2, 5 oder 25 sind. In ES5 Standardfunktionen gibt es das Keyword `arguments`, mittels dessen auf ein Array aller übergebenen Funktionsargumente zugegriffen werden kann innerhalb der Funktion:

```javascript
function Example() {
  console.log(arguments);
}
Example(1, 2, 3, 4, 5);
```

**Ausgabe:**

{% hint style="info" %}
`Arguments(5) [1, 2, 3, 4, 5, callee: ƒ]`
{% endhint %}

**Arrow Functions** bieten diese Möglichkeit nicht mehr und werfen stattdessen einen Fehler:

```javascript
const Example = () => {
  console.log(arguments);
};
Example(1, 2, 3, 4, 5);
```

**Ausgabe:**

{% hint style="danger" %}
Uncaught ReferenceError: arguments is not defined
{% endhint %}

Hier kommt nun erstmals der **Rest Operator** ins Spiel. Dieser schreibt uns sämtliche übergebene Funktionsargumente, die wir nicht bereits in benannte Variablen geschrieben haben, in eine weitere Variable mit einem beliebigen Namen:

```javascript
const Example = (...rest) => {
  console.log(rest);
};
Example(1, 2, 3, 4, 5);
```

**Ausgabe:**

{% hint style="info" %}
`[1, 2, 3, 4, 5]`
{% endhint %}

Dies funktioniert nicht nur als einzelnes Funktionsargument, sondern auch, wenn wir vorher bereits benannte Parameter definiert haben. Hier kümmert sich der **Rest Operator** dann buchstäblich um den letzten verbliebenen **Rest:**

```javascript
const Example = (first, second, third, ...rest) => {
  console.log('first:', first);
  console.log('second:', second);
  console.log('third:', third);
  console.log('rest:', rest);
};
Example(1, 2, 3, 4, 5);
```

**Ausgabe:**

{% hint style="info" %}
`first: 1 second: 2 third: 3 rest: [4, 5]`
{% endhint %}

Der **Rest Operator** sammelt hier also die restlichen, verbliebenen Elemente aus einem **Destructuring** ein und speichert diese in einer Variable mit dem Namen, der hinter den drei Punkten angegeben wird. Dieser muss dabei nicht wie im obigen Beispiel `rest` heißen sondern kann jeden gültigen JavaScript-Variablennamen annehmen.

Das funktioniert aber nicht nur bei Funktionen sondern ebenso bei **Array Destructuring**:

```javascript
const athletes = [
  'Usain Bolt',
  'Andre De Grasse',
  'Christophe Lemaitre',
  'Adam Gemili',
  'Churandy Martina',
  'LaShawn Merritt',
  'Alonso Edward',
  'Ramil Guliyev',
];
const [gold, silver, bronze, ...competitors] = athletes;
console.log(gold);
console.log(silver);
console.log(bronze);
console.log(competitors);
```

**Ausgabe:**

{% hint style="info" %}

```javascript
'Usain Bolt'
'Andre De Grasse'
'Christophe Lemaitre'`
[
  'Adam Gemili',
  'Churandy Martina',
  'LaShawn Merritt',
  'Alonso Edward',
  'Ramil Guliyev'
]
```

{% endhint %}

… wie auch beim **Object Destructuring:**

```javascript
const user = {
  firstName: 'Manuel',
  lastName: 'Bieh',
  job: 'JavaScript Developer',
  hair: 'Brown',
};
const { firstName, lastName, ...other } = user;
console.log(firstName);
console.log(lastName);
console.log(other);
```

<span class="force-break-before"></span>

**Ausgabe:**

{% hint style="info" %}
`Manuel`  
`Bieh`  
`{ job: 'JavaScript Developer', hair: 'Brown' }`
{% endhint %}

All die Werte, die dabei nicht explizit in eine Variable geschrieben wurden während eines **Destructuring Assignments,** können dann in der als **Rest** deklarierten Variable abgerufen werden.

## Template Strings

**Template Strings** in ES2015 sind eine „dritte Schreibweise“ für Strings in JavaScript. Bisher konnten Strings entweder in einfache Anführungszeichen \(`'Beispiel'`\) oder in doppelte Anführungszeichen \(`"Beispiel"`\) gesetzt werden. Nun kommt auch die Möglichkeit hinzu, diese in Backticks \(`Beispiel` \) zu setzen.

**Template Strings** können in zwei Varianten auftreten. Als gewöhnliche **Template Strings**, die JavaScript Ausdrücke enthalten können, sowie in erweiterter Form als sog. **Tagged Template Strings**.

**Tagged Template Strings** sind eine deutlich mächtigere Form von **Template Strings**. Mit ihnen kann die Ausgabe von **Template Strings** mittels einer speziellen Funktion modifiziert werden. Das ist bei der gewöhnlichen Arbeit mit React erst einmal weniger wichtig.

Wollte man sie mit JavaScript-Ausdrücken oder Werten mischen, griff man in ES5 meist zu einfacher **String Concatenation:**

```javascript
var age = 7;
var text = 'Meine Tochter ist ' + age + ' Jahre alt';
```

```javascript
var firstName = 'Manuel';
var lastName = 'Bieh';
var fullName = firstName + ' ' + lastName;
```

Mit **Template Strings** wurde nun eine Variante von String eingeführt, die selbst wiederum **JavaScript-Ausdrücke** enthalten kann. Diese werden dazu innerhalb eines **Template Strings** in eine Zeichenkette in der Form `${ }` gesetzt. Um bei den obigen Beispielen zu bleiben:

```javascript
const age = 7;
const text = `Meine Tochter ist ${age} Jahre alt`;
```

```javascript
const firstName = 'Manuel';
const lastName = 'Bieh';
const fullName = `${firstName} ${lastName}`;
```

Dabei können innerhalb der geschweiften Klammern sämtliche JavaScript-Ausdrücke verwendet werden. Also auch Funktionsaufrufe:

```javascript
console.log(`Das heutige Datum ist ${new Date().toISOString()}`);
console.log(`${firstName.toUpperCase()} ${lastName.toUpperCase()}`);
```

## Promises und async/await

Promises \(dt. _Versprechen_\) sind kein grundsätzlich neues Konzept in JavaScript, in ES2015 haben sie aber erstmals Einzug in den Standard erhalten und können nativ ohne eine andere Library \(z.B. q, Bluebird, rsvp.js, …\) verwendet werden. Ganz grob erlauben Promises es, die asynchrone Entwicklung durch Callbacks zu _linearisieren_. Ein Promise bekommt eine **Executor-Funktion** übergeben, die ihrerseits die zwei Argumente `resolve` und `reject` übergeben bekommen, und kann einen von insgesamt drei verschiedenen Zuständen annehmen: als Initialwert ist dieser Zustand `pending` und je nachdem ob eine Operation erfolgreich oder fehlerhaft war, die Executor-Funktion also das erste \(`resolve`\) oder das zweite \(`reject`\) Argument ausgeführt hat, wechselt dieser Zustand zu `fulfilled` oder `rejected`. Auf die beiden Endzustände kann dann mittels der Methoden `.then()` und `.catch()` reagiert werden. Wird `resolve` aufgerufen, wird der `then()`-Teil ausgeführt; wird `reject` aufgerufen, werden **sämtliche** `then()`-Aufrufe übersprungen und der `catch()`-Teil wird stattdessen ausgeführt.

Eine Executor-Funktion **muss** dabei zwangsweise eine der beiden übergebenen Methoden ausführen, andernfalls bleibt das Promise dauerhaft _unfulfilled_, was zu fehlerhaften Verhalten und in bestimmten Fällen sogar zu Memory Leaks innerhalb einer Anwendung führen kann.

Um den Unterschied zwischen Promises und Callbacks einmal zu demonstrieren, werfen wir einen Blick auf das folgende fiktive Beispiel:

```javascript
const errorHandler = (err) => {
  console.error('An error occured:', err.message);
};

getUser(
  id,
  (user) => {
    user.getFriends((friends) => {
      friends[0].getSettings((settings) => {
        if (settings.notifications === true) {
          email.send(
            'You are my first friend!',
            (status) => {
              if (status === 200) {
                alert('User has been notified via email!');
              }
            },
            errorHandler
          );
        }
      }, errorHandler);
    }, errorHandler);
  },
  errorHandler
);
```

Wir rufen über die asynchrone `getUser()`-Funktion einen User zu einer entsprechenden `id` ab. Von diesem User besorgen wir uns mittels der asynchronen `getFriends()`-Methode eine Liste aller seiner Freunde. Vom ersten Freund \(`friends[0]`\) rufen wir mittels der asynchronen `getSettings()`-Methode die Benutzereinstellungen ab. Erlaubt der Benutzer E-Mail-Benachrichtigungen, schicken wir ihm eine E-Mail und reagieren, ebenfalls wieder asynchron, auf den Response des Mailservers.

Dabei ist das Beispiel noch ein relativ simples, es gibt keinerlei explizite Fehlerbehandlung und es gibt auch keine nennenswerten Ausnahmefälle. Dennoch ist der Code im Beispiel bereits **6 Ebenen** tief verschachtelt. Das Arbeiten mit Callbacks kann daher schnell unübersichtlich werden, insbesondere wenn innerhalb einer Callback-Funktion weitere Callback-Funktionen ausgeführt werden, wie in unserem Beispiel. So kommt es schnell zu der oft auch als **Pyramid of Doom** bezeichneten Verschachtelung von Callbacks.

Nun schreiben wir das Beispiel einmal um und gehen davon aus, unsere fiktiven API-Methoden geben jeweils ein Promise zurück:

```javascript
const errorHandler = (err) => {
  console.error('An error occured:', err.message);
};

getUser(id)
  .then((user) => user.getFriends())
  .then((friends) => friends[0].getSettings())
  .then((settings) => {
    if (settings.notifications === true) {
      return email.send('You are my first friend!');
    }
  })
  .then((status) => {
    if (status === 200) {
      alert('User has been notified via email');
    }
  })
  .catch(errorHandler);
```

Wir reagieren hier nach jedem Schritt mittels `then()` auf das zurückgegebene Promise, erreichen das gleiche Resultat wie vorher bei der Callback-Version, haben aber an der tiefsten Stelle lediglich eine Verschachtelung, die 2 Ebenen tief ist.

Dabei ist es relativ simpel, bestehenden, auf Callback basierenden Code in Promises umzuschreiben. Das möchte ich kurz anhand der Geolocation API und konkret deren `getCurrentPosition()`-Methode demonstrieren. Wer es nicht kennt: die Methode existiert auf dem `navigator.geolocation` Objekt, öffnet eine Benachrichtigung im Browser und fragt den Benutzer um Erlaubnis, ihn orten zu dürfen. Sie erwartet zwei Callbacks als Argument: das erste, der Success-Callback, bekommt ein Objekt mit der Position des Benutzers übergeben, falls dieser der Ortung zustimmt. Der zweite, der Error-Callback, bekommt ein Fehler-Objekt übergeben, falls der Benutzer einer Ortung entweder nicht zugestimmt hat oder eine Ortung aus anderen Gründen nicht möglich ist.

```javascript
navigator.geolocation.getCurrentPosition(
  (position) => {
    console.log(
      `User position is at ${position.coords.latitude}, ${
        position.coords.longitude
      }`
    );
  },
  () => {
    console.log('Unable to locate user');
  }
);
```

Und so wird der Callback in ein Promise umgewandelt:

```javascript
const getCurrentPositionPromise = () => {
  return new Promise((resolve, reject) => {
    navigator.geolocation.getCurrentPosition(resolve, reject);
  });
};
```

Jap. Das war es wirklich schon. Nun können wir statt mittels der Callback-Syntax über folgenden Aufruf auf die Position des Benutzers zugreifen:

```javascript
getCurrentPositionPromise()
  .then((position) => {
    console.log(
      `User position is at ${position.coords.latitude}, ${
        position.coords.longitude
      }`
    );
  })
  .catch(() => {
    console.log('Unable to locate user');
  });
```

Einige neuere JavaScript APIs im Browser sind bereits diesem Ansatz folgend implementiert worden. Wer mehr über Promises und deren Funktionsweise erfahren möchte, dem empfehle ich [den entsprechenden Artikel bei den MDN Web Docs](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Promise) zu lesen. Die Erklärung zu Promises sollte nur einleitend sein, um auf ein deutlich spannenderes neues Feature vorzubereiten, nämlich:

### Asynchrone Funktionen mit `async` und `await`

Asynchrone Funktionen mit den Schlüsselwörtern `async` und `await` können vielleicht ein bisschen als die „nächste Evolutionsstufe“ in der asynchronen Entwicklung nach Callbacks und Promises gesehen werden. Sie haben Einzug in die JavaScript-Spezifikation in ES2016 gehalten. Unter der Haube nutzen sie zwar noch immer Promises, machen diese aber weitgehend unsichtbar. Sie erlauben es uns asynchronen Code so zu schreiben, dass er nahezu wie synchroner Code aussieht. Keine Callbacks mehr und auch kein `then()` oder `catch()` mehr.

Dazu wird einem asynchronen Funktionsaufruf das Schlüsselwort `await` vorangestellt. Um `await` nutzen zu können, muss der umgebenden Funktion das zweite Schlüsselwort `async` vorangestellt werden, um dem JavaScript-Interpreter mitzuteilen, dass es sich um eine solche asynchrone Funktion handelt. Bei der Nutzung von `await` ohne eine Funktion als `async` zu markieren kommt es zu einer Exception.

Werfen wir also nochmal einen Blick auf das Beispiel unseres Users, der eine E-Mail an seinen ersten Freund schicken möchte. Diesmal mit asynchronen Funktionen:

```javascript
(async () => {
  try {
    const user = await getUser(id);
    const friends = await user.getFriends();
    const settings = await friends[0].getSettings();
    if (settings.notifications === true) {
      const status = await email.send('You are my first friend!');
      if (status === 200) {
        alert('User has been notified via email');
      }
    }
  } catch (err) {
    console.error('An error occured:', err.message);
  }
})();
```

Asynchrone Funktionen mit `async` und `await` sind für mich persönlich eine der nennenswertesten Veränderungen von JavaScript in den vergangenen Jahren, da sie das Arbeiten mit asynchronen Daten fast zum Kinderspiel werden lassen, verglichen mit komplexen und unübersichtlichen Callbacks. Und auch Promises, die bereits eine große Erleichterung ggü. herkömmlichen Callbacks waren, wirken im direkten Vergleich mit asynchronen Funktionen schon beinahe komplex.

## Import-Syntax und Javascript-Module

Module in JavaScript sind so eine Sache. Offiziell gab es sie bisher nicht, aber es gab immer wieder Versuche, Module in JavaScript einzuführen. Wer schon etwas länger dabei ist wird vielleicht noch **AMD** \(Asynchronous Module Definition\) kennen, wer mal mit Node.js gearbeitet hat, dem sollten außerdem CommonJS-Module \(d.h. `module.exports` und `require('./myModule')`\) ein Begriff sein. Lange gab es dann ausführliche Diskussionen darüber, auf welchen Modul-Standard man sich einigt, wie die Syntax aussieht und wie letztendlich die Implementierung auf Interpreter-Seite aussieht. Die Wahl fiel auf Module, die mittels `import` und `export` Keywords untereinander kommunizieren.

Babel ist dann vorangegangen und hat eine Lösung implementiert, die auf dem damaligen Stand der offiziellen Spezifikation basierte. Diese Umsetzung wurde dann zwischendurch mal geändert, weil es Updates am entsprechenden Standard gab, dann kam noch Webpack und implementierte einen eigenen Mechanismus zum Auflösen und Laden von JavaScript-Modulen, der sich am nun verabschiedeten Standard orientiert. Ebenso wie TypeScript.

Mittlerweile ist man sich nach gerade einmal **10** Jahren bei der Spezifikation einig und die Umsetzung auf Seiten der JavaScript-Engines ist im vollen Gange. Klingt kompliziert, ist es zwischendrin auch mal gewesen, inzwischen gibt es aber Konsens und für uns Entwickler herrscht allmählich Klarheit. Aber dennoch gibt es auch noch immer einige Fallstricke, durch die wir auch in Zukunft auf Webpack, Babel oder TypeScript setzen müssen \(oder eher: sollten\), um komfortabel mit Modulen zu arbeiten. Dazu später mehr.

Soviel zur Historie. Also wie funktionieren jetzt Imports und was sind Module überhaupt?

<span class="force-break-before"></span>

### Module in JavaScript

Das Ziel von Modulen ist es, Scopes in JavaScript auf einer **per Modul-Ebene zu kapseln**. Ein Modul in diesem Sinne ist tatsächlich ein einzelnes **File**. Sofern man sie nicht explizit durch das Erstellen eines neuen Scopes begrenzt, z.B. indem man es in eine **IIFE** \(_Immediately Invoked Function Expression_\) einschließt, ist jede Funktion, jede Variable, die in JavaScript definiert wird, erst einmal global verfügbar. Module wirken dem entgegen, indem sämtlicher Code erst einmal nur **innerhalb des Moduls** verfügbar ist. Dadurch vermeidet man Komplikationen, bspw. wenn zwei Libraries die gleiche Variable nutzen, außerdem schafft man auf einfache Art wiederverwendbaren Code, ohne auf der anderen Seite Angst haben zu müssen, dass dieser an anderer Stelle bereits existierende Variablen oder Funktionen ungewollt überschreibt.

Module können die in ihnen definierten Funktionen, Klassen oder Variablen **exportieren**, andere Module können diese Exports dann bei Bedarf importieren. Für den Export von Funktionen und Variablen gibt es ein `export`-Keyword, um diese Exports dann später an anderer Stelle zu importieren gibt es, ihr denkt es euch, das entsprechende `import`-Keyword. Exports können zwei Formen annehmen, nämlich zum einen die eines **Named Exports** \(dt. **benannte Exporte**\) und auf der anderen Seite den, des **Default Export** \(dt. **Standard Export**\).

#### Named Exports

Nehmen wir an, wir haben ein Modul `calc.mjs`, das allerhand Funktionen für uns bereitstellt, um Berechnungen verschiedener Art auszuführen. Das Modul könnte bspw. den folgenden Inhalt haben:

```javascript
export const double = (number) => number * 2;
export const square = (number) => number * number;
export const divideBy = (number, divisor) => number / divisor;
export const divideBy5 = (number) => divideBy(number, 5);
```

Wir kündigen hier also einen **Export** an, definieren direkt danach eine Variable, der wir eine Arrow Function zuweisen, die einen Parameter bekommt \(oder zwei\) und direkt das Ergebnis der Berechnung zurückgibt. Alternativ geht das auch in zwei separaten Schritten:

```javascript
const double = (number) => number * 2;
export double;
```

An anderer Stelle innerhalb unserer Anwendung können wir diese Funktionen nun mittels `import`-Keyword **importieren**. Dazu nutzen wir `import` gefolgt von den Exports, die wir importieren wollen, in geschweiften Klammern, gefolgt von `from` und dem Pfad zum Modul.

```javascript
import { double, square, divideBy5 } from './calc.mjs';

const value = 5;
console.log(double(value)); // 10
console.log(square(value)); // 25
console.log(divideBy5(value)); // 1
```

Ein File kann dabei theoretisch **unbegrenzt viele benannte Exports** haben, sie müssen sich jedoch in ihrem Namen unterscheiden und ein bereits exportierter Name **darf nicht ein weiteres Mal exportiert werden.**

#### Default Export

Zusätzlich zu den \(Plural\) sog. **Named Exports** aus dem obigen Beispiel gibt es noch den \(Singular\) `Default Export`, eine spezielle Form eines Exports, der innerhalb eines jeden Moduls nur **ein einziges Mal** vorkommen darf und der mit dem Keyword `default` gekennzeichnet wird. Wird eine Variable oder eine Funktion als `default` gekennzeichnet, ist es möglich diesen Export auch ohne geschweifte Klammern zu importieren. Der **Default Export** kann z.B. dazu dienen, mehrere benannte Exporte zu bündeln, um diese anschließend nicht einzeln importieren zu müssen.

```javascript
export const double = (number) => number * 2;
export const square = (number) => number * number;
export const divideBy = (number, divisor) => number / divisor;
export const divideBy5 = (number) => divideBy(number, 5);

export default {
  double,
  square,
  divideBy,
  divideBy5,
};
```

Unsere Anwendung müsste dann stattdessen lediglich das Modul selbst importieren, also dessen **Default Export** und einer Variable zuweisen:

```javascript
import Calc from './calc.mjs';

console.log(Calc.double(value)); // 10
console.log(Calc.square(value)); // 25
console.log(Calc.divideBy5(value)); // 1
```

Grundsätzlich ist es in vielen Fällen sinnvoll, dass ein Modul auch einen Default Export hat. Insbesondere bei komponentenbasierten Libraries wie React oder auch Vue.js ist es üblich, nur einen Export pro Modul zu haben, dieser sollte dann der **Default Export** sein. Auch wenn dies syntaktisch nicht zwingend notwendig wäre, ist dies inzwischen de-facto Standard bei der Arbeit mit React.

```javascript
export default class MyComponent extends React.Component {
  // ...
}
```

### Fallstricke: Browser vs. Node.js

Wer aufmerksam war und gut aufgepasst hat, dem wird vielleicht aufgefallen sein, dass wir oben aus einem File mit dem Namen `calc.mjs` importieren, nicht `calc.js` \(`.mjs` statt `.js`\). Dies ist die Konvention, auf die man sich bei der Verwendung von JavaScript-Modulen in Node.js im langwierigen, oben beschriebenen Standardisierungsprozess geeinigt hat.

Wollt ihr also universelles JavaScript schreiben, also JavaScript, das sowohl serverseitig mit Node.js ausgeführt werden kann, als auch clientseitig im Browser funktioniert, und wollt ihr das tun ohne einen Compiler-Zwischenschritt durch bspw. Babel, Webpack oder TypeScript einzulegen, **müsst** ihr zwangsweise die `.mjs`-Endung für eure Files verwenden.

Das Laden von Modulen funktioniert in Node.js also etwas anders als im Browser. Während es dem Browser egal ist, welche Datei-Endung ein Modul hat \(solange der Server den Content-Type `text/javascript` mitsendet\), benötigt Node.js zwangsweise die `.mjs` Datei-Endung um JavaScript-Module als solche zu identifizieren.

Um JavaScript-Module im Browser zu nutzen, muss das `type`-Attribut auf dem `<script></script>`-Element den Wert `module` erhalten. Also:

```markup
<script src="./myApp.mjs" type="module"></script>
```

Browser, die den `type="module"` unterstützen, unterstützen auch gleichzeitig das `nomodule`-Attribut zur Auslieferung von Fallbacks für Browser ohne Module-Unterstützung und ignorieren dieses.

```markup
<script src="./myApp.mjs" type="module"></script>
<script src="./myApp.bundle.js" nomodule></script>
```

Ein Browser mit Module-Support würde hier die `myApp.mjs` laden, während alle anderen stattdessen ein gebundletes \(bspw. durch Webpack\) `myApp.bundle.js` laden würden.

Doch das ist noch nicht alles, denn Node.js besitzt einen sehr eigenen Mechanismus zum Finden und Laden von Dateien. So werden bspw. Module, die keinen relativen Pfad haben, also nicht mit `./` oder `../` beginnen, z.B. in `node_modules` oder `node_libraries` gesucht. Außerdem lädt Node.js standardmäßig eine darin befindliche index.js wenn Node.js einen Ordner mit dem angegebenen Namen findet.

```javascript
import MyModule from 'myModule';
```

Node.js würde also nach diesem Import u.a. im Ordner `./node_modules/myModule` suchen, dort eine `index.js` laden oder alternativ im `main`-Feld der `package.json` nach dem korrekten File suchen. Der Browser kann hingegen nicht nach Belieben verschiedene Pfade ausprobieren um das richtige File zu finden, da dies jedes Mal einen teuren Netzwerkrequest und möglicherweise viele 404 Responses verursachen würde.

Hinzu kommt, dass **Import Specifier**, das ist der Part hinter dem `from`, also das Modul aus dem ihr importieren wollt, im Browser geschützt sind und aus einer gültigen URL oder einem relativen Pfad bestehen müssen.

Imports wie der folgende sind damit im Browser momentan gar nicht möglich:

```javascript
import React from 'react';
```

Abhilfe schaffen sollen hier später einmal die **Package Name Maps**, ein Proposal, also ein Vorschlag für kommende ECMAScript Versionen, das aber momentan noch ganz am Anfang der Diskussion steht. Darum kommen wir wie eingangs erwähnt in absehbarer Zeit nicht drum herum, auch weiterhin einen Module Bundler wie Webpack zu benutzen, um komfortabel mit ES-Modules arbeiten zu können wenn wir JavaScript-Module gleichzeitig sowohl serverseitig als auch clientseitig nutzen wollen.

<span class="force-break-before"></span>

## Fazit

ES2015 und die nachfolgenden Versionen bieten eine Menge nützliche neue Funktionen, die es bisher in JavaScript nicht gab. Viele davon sind bei der Arbeit mit React nahezu nicht wegzudenken. Zu den wichtigsten Neuerungen gehören die hier beschriebenen:

- Variablendeklarationen mit `let` und `const`
- **Arrow Functions**, um Funktionen zu erstellen, die kein eigenes `this` binden
- **Klassen**. Machen vieles einfacher und sind die Basis von **React Class Components**
- Die **Rest und Spread Operatoren**, die das Lesen und Schreiben von Daten in Arrays und Objekten deutlich vereinfachen
- **Template Strings**, um die Arbeit mit JavaScript-Ausdrücken in Strings einfacher zu machen
- **Promises** und **Asynchrone Funktionen** mittels `async`/`await` um die Arbeit mit asynchronen Daten deutlich zu vereinfachen
- **Import** und **Export** für die Kapselung von wiederverwendbarem JavaScript auf Module-Ebene
