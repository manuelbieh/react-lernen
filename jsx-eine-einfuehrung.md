# JSX – eine Einführung

## JSX als wichtiger Bestandteil in der React-Entwicklung

Bevor wir tiefer in die Entwicklung von Komponenten einsteigen möchte ich zuerst einmal auf **JSX** eingehen, da JSX einen wesentlichen Teil bei der Arbeit mit React darstellt. Wie eingangs schon erwähnt stellt JSX einen ganz grundlegenden Teil der meisten React-Komponenten dar und ist aus meiner Sicht einer der Gründe, warum React so schnell und positiv von so vielen Entwicklern angenommen wurde. Mittlerweile bieten auch andere Frameworks wie Vue.js die Möglichkeit JSX zur Komponenten getriebenen Entwicklung einzusetzen.

JSX sieht auf den ersten Blick erst einmal gar nicht sehr viel anders aus als HTML, oder eher noch XML, da in JSX, eben wie auch in XML und XHTML jedes geöffnete Element ein schließendes Element \(`</div>`\) besitzen oder selbstschließend \(`<img />`\) sein muss. Mit dem grundlegenden Unterschied, dass JSX auf **JavaScript-Ausdrücke** zurückgreifen kann und dadurch sehr mächtig wird. 

Unter der Haube werden in JSX verwendete Elemente in einem späteren Build-Prozess in verschachtelte `React.createElement()`-Aufrufe umgewandelt. Wir erinnern uns zurück an die Einleitung. Dort hatte ich bereits kurz erwähnt das React eine Baumstruktur an Elementen erzeugt, die selbst aus verschachtelten `React.createElement()`-Aufrufen besteht.

Klingt jetzt alles fürchterlich kompliziert, ist aber ganz einfach. Wirklich. Ein Beispiel: sehen wir uns einmal das folgende kurze HTML-Snippet an:

```jsx
<div id="app">
  <p>Ein Paragraph in JSX</p>
  <p>Ein weiterer Paragraph</p>
</div>
```

Wird dieses HTML so in dieser Form in **JSX** verwendet, werden diese Elemente werden später durch Babel in das folgende ausführbare JavaScript transpiliert:

```javascript
React.createElement(
  "div",
  { id: "app" },
  React.createElement(
    "p",
    null,
    "Ein Paragraph in JSX"
  ),
  React.createElement(
    "p",
    null,
    "Ein weiterer Paragraph"
  )
);
```

Das erste Funktionsargument für `createElement()` ist dabei jeweils der Tag-Name eines DOM-Elements als String-Repräsentation oder ein anderes JSX-Element, dann allerdings als Funktionsreferenz.

Das zweite Argument repräsentiert die **Props** eines Elements, also in etwa vergleichbar mit HTML-Attributen, wobei die Props in React deutlich flexibler sind und anders als herkömmliche HTML-Attribute nicht auf Strings beschränkt sind, sondern auch Arrays, Objekte oder gar andere React-Komponenten als Wert enthalten können.

Alle weiteren Argumente stellen die Kind-Elemente \(_„children“_\) des Elements dar. Im obigen Beispiel hat unser `div` zwei Paragraphen \(`<p>`\) als Kind-Elemente, welche selbst keine eigenen Props haben \(`null`\) und lediglich einen Text-String \(`Ein Paragraph in JSX` bzw `Ein weiterer Paragraph`\) als Kind-Element besitzt. 

Wem das jetzt zu kompliziert klingt den kann ich beruhigen: das geht in der Praxis hinterher wie von selbst von der Hand. Fast so, als würde man HTML-Markup schreiben. Dennoch halte ich es für wichtig die Hintergründe zumindest einmal gelesen zu haben um spätere Beispiele besser nachvollziehen zu können.

## Ausdrücke in JavaScript

Was bedeutet dies nun für unsere JavaScript-Ausdrücke, auf die wir ja nun auch in JSX zurückgreifen können? 

Zuerst einmal ist ein Ausdruck in JavaScript, kurz gesagt, ein Stück Code, der am Ende einen „Wert“ erzeugt bzw ein „Ergebnis“ zur Folge hat. Vereinfacht gesagt: alles was man bei der Variablenzuweisung auf die **rechte** Seite des Gleich-Zeichens \(=\) schreiben kann.

```javascript
1 + 5
```

… ist ein solcher Ausdruck, dessen Wert `6` beträgt. 

```javascript
'Hal'+'lo'
```

… ist ein anderer Ausdruck der die zwei Strings `Hal` und `lo` per **String Concatenation** zu einem Wert `Hallo` zusammenfügt. 

Stattdessen könnten wir aber auch einfach gleich schreiben:

```javascript
6
'Hallo'
[1,2,3,4]
{a: 1, b: 2, c: 3}
true
null
```

… da **JavaScript-Datentypen** allesamt auch als Ausdruck verwendet werden können.

Die ES2015 **Template String Syntax**, die Backticks \(`````\) benutzt, ist ebenfalls ein Ausdruck. Klar, sind sie doch letztlich nichts anderes als ein String:

```javascript
`Hallo ${name}`
```

Was hingegen **kein** Ausdruck ist, ist:

```javascript
if (active && visibility === 'visible') { … } 
```

… da ich zum Beispiel auch nicht schreiben könnte:

```javascript
const isVisible = if (active && visibility === 'visible') { … }
```

Das würde mir jeder JavaScript-Interpreter wegen ungültiger Syntax um die Ohren hauen.

Lasse ich das `if` hier hingegen allerdings weg habe ich einen **Logical AND Operator**, der wiederum ein Ausdruck ist und einen Wert zum Ergebnis hat \(in diesem Fall `true` oder `false`\):

```javascript
const isVisible = active && visibility === 'visible'
```

Ebenso ist der Ternary-Operator \( ? : \) ein Ausdruck:

```javascript
Bedingung ? wahr : unwahr
```

Ausdrücke sind aber nicht auf boolsche Werte, Nummern, Strings beschränkt sondern können auch Objekte, Arrays, Funktionsaufrufe und sogar Arrow-Functions sein, die ebenfalls neu in ES2015 eingeführt werden und uns hier nicht das letzte mal begegnet sein werden. 

Beispiel für eine Arrow-Function:

```javascript
(number) => {
  return number * 2;
}
```

All das wird später noch wichtig werden. Um dem ganzen jetzt endgültig die Krone aufzusetzen können Ausdrücke selbst wiederum wieder JSX beinhalten und so kann man das Spiel endlos weiterführen. Uff.

Da das später noch wichtig wird, hier nochmal einige Beispiele für JSX, das gültige Ausdrücke beinhaltet:

#### Simple Mathematik

```jsx
<span>5 + 1 = {5 + 1}</span>
```

#### Ternary Operator

```jsx
(
  <span>
    Heute ist {new Date().getDay() === 1 ? 'Montag' : 'nicht Montag'}
  </span>
)
```

#### Ternary Operator als Wert einer Prop

```jsx
<div className={user.isAdmin ? 'is-admin' : null}>…</div>
```

#### Array.map\(\) mit JSX als Rückgabewert das wiederum einen Ausdruck enthält

```jsx
(
  <ul>
    {['Tim', 'Struppi'].map((name) => <li>{name})</li>)}
  </ul>
)
```

#### Zahlenwerte in Props

```jsx
<input type="range" min={0} max={100} />
```

All dies sind erste Beispiele wie Ausdrücke dafür verwendet werden können um aus JSX mehr als nur simples HTML zu machen.

## **Was man außerdem wissen sollte**

Wer die Beispiele aufmerksam studiert hat, dem werden je nach JavaScript-Kenntnissen vielleicht einige Dinge aufgefallen sein. Zuerst einmal tauchen in den Beispielen scheinbar willkürlich Klammern auf. Dies hat den Hintergrund, dass JSX stets in Klammern, also „`(`“ und „`)`“ geschrieben werden muss wenn sich das JSX sich über mehr als eine Zeile erstreckt \(also doch nicht willkürlich.\)

Prinzipiell schadet es nicht sein komplettes JSX immer in Klammern zu schreiben, auch wenn es sich nur um eine einzige Zeile handelt, Viele Leute bevorzugen das sogar aus Gründen der Einheitlichkeit, wirklich zwingend notwendig ist das aber nur bei mehrzeiligem JSX.

Möchten wir statt eines **Strings** einen **Ausdruck** innerhalb der Props nutzen wie im Beispiel _„Ternary Operator als Wert einer Prop“_, so nutzen wir dafür statt einfacher oder doppelter Attribut-Anführungszeichens auch hier die geschweiften Klammern um React mitzuteilen: hier drin befindet sich ein Ausdruck.

{% hint style="warning" %}
Bei Objekten als Wert müssen jeweils **zwei** öffnende und schließende Klammern geschrieben werden. Die äußeren Klammern leiten den Ausdruck ein \(bzw. beenden diesen\) und die inneren sind die, des eigentlichen Objekts:

`<User data={{ name: 'Manuel', location: 'Berlin' }} />`

Das gilt in ähnlicher Form auch für Array-Literals, natürlich mit dem Unterschied, dass die inneren Klammern die eckigen sind, die das Array-Literal kennzeichnen:

`<List items={[1, 2, 3, 4, 5]} />`
{% endhint %}

Weiter könnte manch einem aufgefallen sein, dass im gleichen Beispiel die Prop `className` verwendet wird. Wer jemals mit der DOM Element im Browser gearbeitet hat dem wird vielleicht im Gedächtnis geblieben sein, dass mittels `Element.className` auf das `class`-Attribute eines Elements zugegriffen werden kann. Ganz genau so ist das auch in React, das sich an den Eigenschaften der DOM `Element` Klasse bedient.

Möchte man gewisse HTML-Attribute in JSX setzen, ist dafür also die JavaScript-Entsprechung zu verwenden. `class` ist ein geschütztes Keyword in JavaScript um eine Klasse zu kennzeichnen, also verwenden wir an dieser Stelle stattdessen `className`. Gleiches gilt für `for`, was in JavaScript als Keyword benutzt wird um Schleifen einzuleiten, in HTML aber hingegen um `<label>` Elementen mitzuteilen, welches Eingabefeld sie beschreiben. Statt `for` schreiben wir in JSX also angelehnt an das DOM [HTMLLabelElement Interface](https://developer.mozilla.org/en-US/docs/Web/API/HTMLLabelElement/htmlFor) `htmlFor`:

```jsx
<fieldset>
  <input type="text" id="name" />
  <label htmlFor="name">Name</label>
</fieldset>
```

Dieses Muster zieht sich konsequent durch alle bekannten HTML-Attribute. Möchtest du ein HTML-Attribut in JSX setzen, musst du dafür die Schreibweise der entsprechenden JavaScript DOM Element-Eigenschaft verwenden. So wird aus `tabindex` eben `tabIndex`, `readonly` wird zu `readOnly`, `maxlength` wird zu `maxLength`, usw.

Aber: Keine Angst! Im Development-Modus gibt React eine entsprechende Warnung aus, so dass es dir selten passieren sollte, dass du entsprechenden fehlerhaften Code nicht bemerkst:

![Warnung in der Browser Console bei der Verwendung illegaler Props](https://lh6.googleusercontent.com/cgdej1K-RlV-97RDzKO2X_yQFhNNknM1rBge0--1I8ID59-IQTHWK9nfGNM0PdN44WTRKe4hPE-r3zZpgnQUstLHR-vCbegZn440oLLGCU7chWa6gkbfcMhQbwBB6swb1lL83jTM)

Wer es genau wissen will, hier die vollständige Liste unterstützter HTML-Attribute wie sie in der offiziellen React-Doku steht \(festhalten, wird lang\):

> `accept acceptCharset accessKey action allowFullScreen alt async autoComplete autoFocus autoPlay capture cellPadding cellSpacing challenge charSet checked cite classID className colSpan cols content contentEditable contextMenu controls controlsList coords crossOrigin data dateTime default defer dir disabled download draggable encType form formAction formEncType formMethod formNoValidate formTarget frameBorder headers height hidden high href hrefLang htmlFor httpEquiv icon id inputMode integrity is keyParams keyType kind label lang list loop low manifest marginHeight marginWidth max maxLength media mediaGroup method min minLength multiple muted name noValidate nonce open optimum pattern placeholder poster preload profile radioGroup readOnly rel required reversed role rowSpan rows sandbox scope scoped scrolling seamless selected shape size sizes span spellCheck src srcDoc srcLang srcSet start step style summary tabIndex target title type useMap value width wmode wrap`

Das gleiche gilt übrigens auch für SVG-Elemente, die man ebenfalls innerhalb von JSX verwenden kann, da SVG valides XML ist! Die Liste der unterstützten SVG-Attribute ist aber gut und gerne 3 mal so lang und längst nicht so relevant im Alltag, weswegen ich dich in diesem Fall aber nun wirklich an die entsprechende Stelle in der offiziellen Doku verweisen muss: [https://reactjs.org/docs/dom-elements.html\#all-supported-html-attributes](https://reactjs.org/docs/dom-elements.html#all-supported-html-attributes)

## Sonderfall Inline-Styles

Natürlich gibt es auch Sonderfälle. Das style-Attribut ist ein solcher. Während Inline-Styles in HTML mit den originalen CSS-Eigenschaftsnamen und als String geschrieben werden, benutzt React, du ahnst es: JavaScript-Eigenschaften und außerdem ein Objekt statt eines einfachen Strings.

Schreibst du in herkömmlichem HTML:

```markup
<div style="margin-left: 12px; border-color: red; padding: 8px"></div>
```

Sieht das Gegenstück in JSX so aus:

```jsx
<div style={{ marginLeft: '12px', borderColor: 'red', padding: '8px'}}></div>
```

Ein weiterer Sonderfall sind Events. Da diese sehr umfangreich sind, widmen wir uns dem Thema später in einem eigenen Kapitel nochmal.

## Kommentare in JSX

Auch Kommentare sind möglich in JSX, funktionieren aber nicht wie in HTML in der Form:

```markup
<!-- Dies ist ein Beispiel für einen Kommentar -->
```

… sondern werden ebenfalls wie Ausdrücke in geschweifte Klammern gefasst, und dann in Form eines JavaScript Mehrzeilenkommentars geschrieben:

```jsx
{/* So sieht ein Kommentar in JSX aus */}
```

Derartige Kommentare können sich natürlich auch über mehrere Zeilen erstrecken. Einzeilige JavaScript Kommentare die mit einem doppelten Slash \(`//`\) eingeleitet werden sind hingegen nicht möglich in JSX. Hier muss also auch bei einem kurzen einzeiligen Kommentar die `/* */` Syntax für mehrzeilige Kommentare verwendet werden.

Damit solltest du auch schon ausreichend Kenntnisse über JSX erlangt haben, um die nachfolgenden Beispiele und Beschreibungen mit weiteren Verlauf immer besser verstehen und nachvollziehen zu können

## Fazit

{% hint style="success" %}
* Mehrzeiliges JSX muss stets in Klammern gesetzt werden
* JSX kann JavaScript-Ausdrücke verarbeiten. Diese müssen in geschweifte Klammern gesetzt werden und können dann auch in Props verwendet werden.
* Um Attribute für HTML-Elemente zu setzen muss die Schreibweise des DOM Element Interface benutzt werden \(`htmlFor` statt `for`, `className` statt `class`\)
* CSS Inline-Styles werden als JavaScript Object geschrieben
* Kommentare werden ebenfalls in geschweifte Klammern gesetzt und verwenden Multiline-Comment Syntax: `/* */`
{% endhint %}

