# Event-Handling

Ein wesentlicher Teil bei der Entwicklung von Anwendungen mit einem komplexen User Interfaces ist natürlich die Interaktion zwischen Benutzer und dem Interface an sich. Insbesondere in Form von **Events**. 

Ich drücke einen Knopf und es passiert etwas. Ich schreibe einen Text in ein Feld und es passiert etwas. Ich wähle ein Element aus einer Liste und es passiert etwas. In einfachem JavaScript stellt uns der Browser dafür die Methoden `addEventListener()` und `removeEventListener()` bereit. Auf diese beiden Event-Methoden kann in React in den meisten Fällen nahezu komplett verzichtet werden, da React ein eigenes System zum Definieren von Benutzerinteraktion gleich mitbringt. Und zwar, nicht erschrecken: über **Inline-Events**. 

Diese **Inline-Events** haben rein äußerlich sehr große Ähnlichkeit zu den HTML Event-Attributen \(also bspw. `<button onclick="myFunction" />`\) und doch ist ihre Funktionsweise grundlegend anders.

Da wurde uns Web-Entwicklern über Jahre immer wieder beigebracht, dass man seine Event-Listener sauber von seinem Markup trennen sollte, **Separation of Concerns**, dann kommt React daher und macht wieder einfach alles anders. 

Und das ist in diesem Fall auch gut so, denn durch die Verwendung eines eigenen Systems zur Verwaltung von Events, nimmt uns React hier wieder eine ganze Menge Arbeit ab und ermöglicht es uns außerdem sehr leicht im Komponenten-Kontext zu bleiben, indem wir alle Event-Handler als **Klassen-Methoden** implementieren können und so sowohl Darstellungslogik als auch Verhaltenslogik in einer einzigen Komponente kapseln können. Kein mühsames und unübersichtliches hin- und herspringen zwischen Controllern und Views!

### Unterschiede zur nativen Event API

Events in React und JSX werden wie erwähnt sehr ähnlich definiert wie HTML Event-Attribute. Doch es gibt einige Unterschiede. So werden Events in React in **CamelCase**-Form definiert, statt in **Lowercase**. Somit wird aus `onclick` in React `onClick`, aus `onmouseover` wird `onMouseOver`, `ontouchstart` wird zu `onTouchStart` usw.

Der dem Event-Handler übergebene erste Parameter ist auch kein Objekt vom Typ `Event`, sondern ein sogenannter `SyntheticEvent`, ein React eigener Wrapper um das native Event-Objekt. Dieser Wrapper ist Teil des React Event Systems und dient als eine Art Normalisierungsschicht, um Cross-Browser-Kompatibilität zu gewährleisten. Dabei hält diese sich, anders als so mancher Browser, strikt an die [Event-Spezifikation des W3C](https://www.w3.org/TR/DOM-Level-3-Events/). 

Ein weiterer Unterschied zur nativen Browser-Event API ist der, dass explizit `preventDefault()` aufgerufen werden muss um das Standardverhalten des Browsers bei einem Event zu unterdrücken statt lediglich `false` aus dem Event-Handler zurückzugeben.

Und nicht zuletzt, ist der im Event-Attribut \(bzw. Event-**Prop** in JSX\) angegebene Wert eine **Referenz zu einer Funktion** statt wie in HTML ein String. Folglich benötigen wir hier die geschweiften Klammern um JSX mitzuteilen, dass es sich um einen JavaScript-Ausdruck handelt. 

Wie das aussieht? So:

```jsx
<button onClick={validateInput}>Validate</button>
```

Wohingegen ein vergleichbarer Event in HTML wie folgt aussähe:

```markup
<button onclick="validateInput">Validate</button>
```

Dieses Vorgehen, das auf den ersten Blick auf einige vielleicht erst einmal etwas eigenartig anmuten könnte bringt aber einige großartige Vorteile mit sich. Und so bekommen wir, wie eingangs bereits angesprochen, Cross-Browser-Kompatibilität „for free“. React registriert dabei im Hintergrund die Events sauber mittels `addEventListener()` und entfernt diese auch automatisch wieder für uns, sobald die Komponente entfernt \(_unmounted_\) wird. Praktisch!

### Scopes in Event-Handlern

Bei der Verwendung von ES2015-Klassen in React ist für gewöhnlich so, dass Event-Handler als Methoden der Klassen-Komponente implementiert werden. Hierbei muss jedoch beachtet werden, dass **Klassen-Methoden nicht automatisch an die Instanz gebunden werden**. Klingt kompliziert, bedeutet aber lediglich, dass `this` in eurem Event-Handler grundsätzlich erstmal `undefined` ist. 

Hierzu ein kurzes Beispiel:

```jsx
class Counter extends React.Component {
  state = {
    counter: 0,
  };

  increase() {
    this.setState((state) => ({
        counter: state.counter + 1,
    }));
  }

  render() {
    return (
      <div>
        <p>{this.state.counter}</p>
        <button onClick={this.increase}>+1</button>
      </div>
    )
  }
}
```

Hier definieren wir also einen `onClick`-Event um den Zähler jeweils um `1` zu erhöhen, sobald der Benutzer auf den Button `+1` klickt. Beim Klick auf den Button sieht unser Benutzer aber stattdessen:

{% hint style="danger" %}
**TypeError  
** Cannot read property 'setState' of undefined
{% endhint %}

Und warum? **Scoping!** Da wir uns beim Klick auf den Button im Event-Handler `increase()` außerhalb der Komponenten-Instanz bewegen, können wir eben auch nicht auf `this.setState()` zugreifen. Dies ist kein fehlerhaftes Verhalten von React sondern das Standardverhalten von ES2015-Klassen. Um dieses Problem zu lösen gibt es nun verschiedene Möglichkeiten.

#### Method Binding in der render\(\)-Methode

Die trivialste Methode ist das Binden der Methode innerhalb der `render()`-Methode. Dazu fügen wir ein `.bind(this)` an die Referenz zur Klassen-Methode:

```jsx
<button onClick={this.increase.bind(this)}>+1</button>
```

Die Methode wird nun im Scope der Instanz aufgerufen und unser Counter zählt problemlos hoch. Dieses Muster sieht man relativ häufig, da es einfach implementiert ist, hat jedoch einen grundlegenden Nachteil. Denn es wird bei jedem Aufruf der Methode 

