# State und Lifecycle-Methods

Kommen wir zu dem, was die Arbeit mit React erstmal wirklich effizient macht: **State**, **stateful Components** und die sogenannten **Lifecycle-Methods**.

Wie im Kapitel über Komponenten bereits angesprochen können **stateful Components** einen eigenen  Zustand, den **State**, halten, verwalten und verändern. Dabei gilt der Grundsatz: **ändert sich der State einer Komponente, löst dies immer auch ein Re-Rendering der Komponente aus!** Dieses Verhalten kann in **Class Components** auch explizit unterbunden werden, was in einigen Fällen sinnvoll ist. Aber der Grundsatz bleibt unverändert: eine State-Änderung löst ein Re-Rendering einer Komponente und ihrer Kind-Komponenten aus.

Das ist insofern hilfreich, als dass wir nicht mehr manuell `ReactDOM.render()` aufrufen müssen wann immer wir meinen dass sich etwas an unserem Interface geändert hat, sondern die Komponenten dies stattdessen selbst entscheiden können.

Neben dem State an sich gibt es auch eine handvoll sogenannter **Lifecycle-Methoden**. Dies sind Methoden die **optional** in einer **Class Component** definiert werden können und von React bei bestimmten Anlässen aufgerufen werden. Beispielsweise wenn eine Komponente erstmals gemountet wird, die Komponente neue Props empfangen oder sich der State innerhalb der Komponente geändert hat.

## Eine erste stateful Component

Der **State** innerhalb einer Komponente ist verfügbar über die Instanz-Eigenschaft `this.state` und ist somit innerhalb einer **Komponente** gekapselt. Weder Eltern- noch Kind-Komponenten können ohne weiteres auf den State einer anderen Komponente zugreifen.

Um in einer Komponente einen initialen Zustand zu definieren gibt es zwei einfache Wege, einen dritten, von der Funktionalität her etwas erweiterten Weg lernen wir später noch mit der **Lifecycle-Methode** `getDerivedStateFromProps()` kennen.

**Initialen State** kann man definieren, indem man im Constructor die Instanz-Eigenschaft `this.state` setzt:

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    this.state = {
      counter: props.counter,
    };
  }
}
```

… oder indem man den State als **ES2017** **Class Property** definiert, was deutlich kürzer ist, jedoch momentan noch das **Babel-Plugin** `@babel/plugin-proposal-class-properties` \(vor Babel 7: `babel-plugin-transform-class-properties`\) benötigt:

```javascript
class MyComponent extends React.Component {
  state = {
    counter: this.props.counter,
  };
}
```

**Create-React-App** unterstützt die **Class Property Syntax** standardmäßig und da viele React-Projekte heute vollständig oder zumindest zu gewissen Teilen auf dem CRA-Setup oder Varianten davon basieren, kommt diese Syntax heute in den meisten Projekten zum Einsatz und kann genutzt werden. Sollte dies mal in einem Projekt nicht der Fall zu sein, empfehle ich dringend die Installation und Nutzung des Babel-Plugins, da es wirklich viele unnötige Zeilen Code bei der täglichen Arbeit mit React einspart, während es gleichzeitig in nur wenigen Minuten eingerichtet ist.

Ist der **State** erst einmal definiert, können wir innerhalb der **Class Component** mittels `this.state` **lesend** auf ihn zugreifen. **Lesend** ist hier ein entscheidendes Stichwort. Denn auch wenn es prinzipiell möglich ist den State direkt über `this.state` zu verändern sollte dies aus verschiedenen Gründen vermieden werden.

## Den State verändern mit this.setState\(\)

Um State zu verändern stellt React eine eigene Methode innerhalb einer **Class Component** bereit: 

```javascript
 this.setState(updatedState)
```

Wann immer der State innerhalb einer Komponente verändert werden soll, sollte dafür `this.setState()` verwendet werden. Der Aufruf von `this.setState()` führt dann dazu, dass React entsprechende **Lifecycle-Methoden** \(wie bspw. `componentDidUpdate()`\) ausführt und eine Komponente **neu rendert!** Würden wir den State stattdessen direkt verändern, also bspw. `this.state.counter = 1;` schreiben, hätte dies vorerst keinerlei Auswirkungen auf unsere Komponente und alles würde aussehen wie bisher, da der Render-Prozess **nicht** ausgelöst werden würde!

Die Methode ist von der Funktionsweise her allerdings etwas komplexer als es auf den ersten Moment aussehen mag. Und so wird nicht einfach nur der alte State durch den neuen State ersetzt und ein Re-Rendering ausgelöst. Es passieren auch noch allerhand andere Dinge. Der Reihe nach.

Zuerst einmal kann die Funktion **zwei verschiedene Arten von Argumenten** entgegennehmen. Das ist einerseits ein **Objekt** mit neuen oder aktualisierten State-Eigenschaften, sowie andererseits eine **Updater-Funktion,** die wiederum ein Objekt zurückgibt oder `null`, falls nichts geändert werden soll. Bestehende gleichnamige Eigenschaften innerhalb des State-Objekts werden dabei **überschrieben**, alle anderen bleiben **unangetastet!** Möchten wir eine Eigenschaft im State zurücksetzen, müssen wie diese dazu also explizit auf `null` oder `undefined` setzen. Der übergebene State wird also immer mit dem bestehenden State **zusammengefügt**, niemals **ersetzt!**

Nehmen wir nochmal unseren oben definierten State mit einer `counter` Eigenschaft, deren initialer Wert für dieses Beispiel erst einmal `0` ist. Nun verändern wir den State und möchten diesem zusätzlich eine `date` Eigenschaft mit dem aktuellen Datum hinzufügen. Übergeben als Objekt wäre unser Aufruf:

```javascript
this.setState({
  date: new Date(),
});
```

Nutzen wir stattdessen eine **Updater-Funktion**, wäre unser Aufruf:

```javascript
this.setState(() => {
  return {
    date: new Date(),
  };
});
```

Oder kurz:

```javascript
this.setState(() => ({
  date: new Date(),
});
```

Unsere Komponente hat anschließend den neuen State:

```javascript
{
  counter: 0,
  date: new Date(),
}
```

Um sicherzustellen stets auf den aktuellen State zuzugreifen, sollte eine **Updater-Funktion** verwendet werden, die den jeweils aktuellen State als ersten Parameter übergeben bekommt. Ein beliebter Fehler der vielen Entwicklern bei der Arbeit mit React schon passiert ist, ist es direkt nach einem `setState()`-Aufruf auf `this.state` zuzugreifen und sich zu wundern, dass der State noch immer der alte ist.

{% hint style="danger" %}
React „sammelt“ schnell aufeinanderfolgende `setState()`-Aufrufe und **führt diese nicht unmittelbar aus**, um unnötig häufiges und überflüssiges Re-Rendering von Komponenten zu vermeiden. Schnell aufeinanderfolgende `setState()`-Aufrufe werden später in gesammelter Form als **Batch-Prozess** ausgeführt. Das ist wichtig zu wissen, da wir nicht unmittelbar nach einem `setState()`-Aufruf mittels `this.state` auf den neu gesetzten State zugreifen können.
{% endhint %}

Stellen wir uns eine Situation vor in der unser `counter`-State dreimal in schneller Abfolge erhöht werden soll. Intuitiv würde man nun vermutlich folgenden Code schreiben:

```javascript
this.setState({ counter: this.state.counter + 1 });
this.setState({ counter: this.state.counter + 1 });
this.setState({ counter: this.state.counter + 1 });
```

Was denkst du, wie ist der neue State wenn der initiale State `0` war? `3`? Falsch. Er ist `1`! Hier kommt der angesprochene **Batching-Mechanismus** von React zum Zug. Um ein sich zu schnell aktualisierendes User Interface zu vermeiden, wartet React hier erst einmal ab. Am Ende kann man den obigen Code simpel ausgedrückt **vom Funktionsprinzip her** in etwa gleichsetzen mit:

```javascript
this.state = Object.assign(
  this.state, 
  { counter: this.state.counter + 1 },
  { counter: this.state.counter + 1 },
  { counter: this.state.counter + 1 },
);
```

Die `counter`-Property überschreibt sich hier während eines Batch-Updates also immer wieder selbst, nimmt aber stets `this.state.counter` als Basiswert für die Erhöhung um 1. Nachdem alle State-Updates ausgeführt wurden, ruft React dann erneut die `render()`-Methode der Komponente auf. 

Bei der Verwendung einer **Updater-Funktion** wird dieser Funktion dabei der jeweils aktuellste State als Parameter übergeben und wir haben Zugriff auf den zum Zeitpunkt des Funktionsaufrufs gültigen State: 

```javascript
this.setState((state) => ({ counter: state.counter + 1 });
this.setState((state) => ({ counter: state.counter + 1 });
this.setState((state) => ({ counter: state.counter + 1 });
```

In diesem Fall nutzen wir eine solche **Updater-Funktion** und aktualisieren jeweils den aktuellsten Wert. Der Wert von `this.state.counter` wäre hier also wie erwartet `3`, da wir über den übergebenen `state`-Parameter mit jedem Aufruf auf den aktuellen State zugreifen. Grundsätzlich ist es allerdings zu empfehlen sich die benötigten Werte erst einmal zu erzeugen und anschließend gesammelt in einem einzigen `setState()`-Aufruf zu übergeben. So ist sichergestellt, dass es keine unnötigen zwischenzeitlichen `render()`-Aufrufe mit im nächsten Moment direkt wieder veraltetem State gibt.

Sollte es doch einmal nötig werden unmittelbar nach einem `setState()`-Aufruf auf den eben neu gesetzten State zuzugreifen bietet React die Möglichkeit als optionalen zweiten Parameter eine Callback-Funktion zu übergeben. Diese wird aufgerufen **nachdem** der State aktualisiert wurde, so dass in dieser mittels `this.state` sicher auf den dann neu gesetzten State zugegriffen werden kann.

```javascript
this.setState({ time: new Date().toLocaleTimeString() }, () => {
  console.log('Neue Zeit:', this.state.time);
});
```

## Lifecycle-Methoden

**Stateful Class Components** bieten gegenüber ihren vereinfachten Kollegen, den **Stateless Functional Components** noch einen weiteren wichtigen Mehrwert: die sogenannten Lifecycle-Methoden. Diese können als Methode einer Class Component definiert werden und werden zu unterschiedlichen Zeitpunkten während eines Komponenten-Lebenszyklus \(daher der Name\) ausgeführt. 

Der Lifecycle einer Methode beginnt in dem Moment, in der diese **gemounted** wird, also innerhalb einer `render()`-Methode tatsächlich Teil zurückgegebenen Element-Baumes ist und endet, wenn die Komponente aus dem Baum der zu rendernden Elemente entfernt wird. Währenddessen gibt es noch Lifecycle-Methoden die auf Updates und auf Fehler reagieren.

### Liste der Lifecycle-Methoden

Im folgenden die Liste der Lifecycle-Methoden in der Reihenfolge wann und in welcher Phase diese durch React aufgerufen werden, sofern diese in einer Komponente definiert wurden:

#### Mounting

Die folgenden Methoden werden **einmalig** aufgerufen wenn die Komponente erstmals gerendert werden, also, vereinfacht gesagt, erstmals zum DOM hinzugefügt werden:

* `constructor()`
* `getDerivedStateFromProps()`
* `componentWillMount()` \(deprecated ab React 17\)
* `render()`
* `componentDidMount()`

#### Updating

Die folgenden Methoden werden aufgerufen wenn Komponenten entweder durch die Hereingabe neuer Props von außen oder durch die Veränderung des eigenen States ein Update erhalten oder oder explizit die von React bereitgestellte `forceUpdate()`-Methode aufgerufen wird:

* `componentWillReceiveProps()` \(deprecated ab React 17\)
* `static getDerivedStateFromProps()`
* `shouldComponentUpdate()`
* `componentWillUpdate()`
* `render()`
* `getSnapshotBeforeUpdate()`
* `componentDidUpdate()`

#### Unmounting

Hier gibt es nur eine Methode, diese wird aufgerufen sobald die Komponente aus dem DOM entfernt wird. Dies ist nützlich um bspw. Event-Listener oder `setTimeout()`/`setInterval()`-Aufrufe, die beim Mounting der Komponente hinzugefügt wurden, wieder zu entfernen:

* `componentWillUnmount()`

#### Fehlerbehandlung

Zuletzt gibt es noch eine Methode die in React 16 neu hinzukam und immer dann aufgerufen wird wenn während des Renderings, in einer der Lifecycle-Methoden oder im Constructor einer **Kind-Komponente** ein Fehler geworfen wird:

* `componentDidCatch()`

Komponenten die eine `componentDidCatch()`-Methode implementieren werden auch als **Error Boundary** bezeichnet und dienen dazu eine Alternative zum fehlerhaften Elementenbaum darzustellen. Dies kann eine High Level Komponente sein \(bezogen auf die Komponenten-Hierarchie\), die grundsätzliche eine Fehler-Seite anzeigt und den Nutzer auffordert die Anwendung neu zu laden sollte ein Fehler auftreten, dies kann aber auch eine Low Level Komponente sein die nur einen kurzen Fehlertext neben einem Button ausgibt, sollte die Aktion die der Button ausgelöst hat einen Fehler geworfen haben.

### Lifecycle-Methoden in der Praxis

Werfen wir einmal einen Blick darauf wie sich die **Lifecycle-Methoden** in einer einfachen Komponente verhalten. Zu diesem Zweck implementieren wir beispielhalber eine Komponente die sekündlich ihren eigenen State verändert und jeweils die aktuelle Zeit ausgibt. Dazu wird beim **Mounting** der Komponente, also in der `componentDidMount()`-Methode, ein Interval gestartet, welches den State der Komponente aktualisiert, wodurch ein Rerendering ausgelöst und wieder die aktuelle Zeit angezeigt wird:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class Clock extends React.Component {
  state = {
    date: new Date(),
  };
  
  componentDidMount() {
    this.intervalId = setInterval(() => {
      this.setState(() => ({
        date: new Date(),
      }));
    }, 1000);
  }
  
  componentWillUnmount() {
    clearTimeout(this.intervalId);
  }
  
  render() {
    return (
      <div>{this.state.date.toLocaleTimeString()}</div>
    );
  }
}

ReactDOM.render(<Clock />, document.getElementById('root'));
```

Hier sehen wir die Lifecycle-Methoden `componentDidMount()` und `componentWillUnmount()` im Einsatz. Wir definieren einen default **State** mit einer Eigenschaft `date`, die eine Instanz des Date-Objekts hält. Beim **Mounting** der Komponente \(`componentDidMount()`\) wird dann via `setInterval()` der Intervall gestartet und dessen Intervall ID in der Instanz-Eigenschaft `this.intervalId` gespeichert. Da der Intervall sekündlich die `setState()`-Methode aufruft, verursacht die Komponente auch regelmäßig ein Re-Rendering, d.h. die `render()`-Methode wird erneut aufgerufen und zeigt wieder die aktuelle Zeit an.

Da die Intervall-Funktion grundsätzlich unabhängig von der React-Komponente ist und abgesehen davon, dass sie die `setState()`-Methode der Komponente aufruft, keinerlei Verbindung zu ihr hat, kümmert sich React auch nicht automatisch darum, dass der Intervall-Aufruf der Funktion gestoppt wird wenn wir die Komponente nicht mehr weiter benötigen. Dafür müssen wir selber sorgen und genau zu diesem Zweck hält React für uns die nächste Lifecycle-Methode bereit: `componentWillUnmount()`.

Diese Methode wird unmittelbar bevor React die Komponente aus dem DOM entfernt aufgerufen und kann dazu benutzt werden um bspw. noch laufende XHRs abzubrechen, Event Listener zu entfernen oder eben einen laufenden Funktionsintervall zu beenden. Genau das tun wir hier: bevor die Komponente entfernt wird, rufen wir `clearTimeout()` auf und übergeben der Funktion die Intervall ID, die wir zuvor in der entsprechenden Instanz-Eigenschaft gespeichert haben.

Sollten wir dies einmal vergessen werden wir im Development-Modus von React aber spätestens beim Aufruf von `this.setState()` in einer bereits entfernten Komponente mit einer Warnung erinnert:

{% hint style="danger" %}
**Warning:** Can't call setState \(or forceUpdate\) on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in the componentWillUnmount method.  
    in Clock
{% endhint %}

### Das Zusammenspiel von State und Props



