# Error Boundaries

Tritt innerhalb einer React-Anwendung ein Fehler auf und wird eine Exception geworfen führt dies unter Umständen dazu, dass die Anwendung nicht mehr angezeigt wird und der Benutzer nur noch einen weißen Bildschirm sieht. Um dieses unschöne Verhalten zu unterbinden wurden in React 16.0 die sog. **Error Boundaries** eingeführt.

Dabei handelt es sich um eine bestimmte Art von Komponente die diverse Fehler **innerhalb ihrer Kind-Hierarchie** abfangen und verarbeiten kann und im Falle eines Fehlers einen alternativen Seitenbaum rendern kann, um den Benutzer vor dem völligen Absturz und dem Anblick eines _weißen Screens_ zu bewahren. **Error Boundaries** agieren also stets als Eltern-Komponente eines Seitenbaums. Kommt es in einer Komponente innerhalb dieses Seitenbaums zu einem Fehler springt die Error Boundary ein und kümmert sich um das Handling des entsprechenden Fehlers. Man kann dieses Verhalten durchaus als eine besondere Art eines `try`/`catch` Blocks für Komponenten-Hierarchien verstehen.

Sie kümmern sich dabei um die Behandlung von Fehlern die aus einer der folgenden Situationen resultieren:

* Fehler in **Lifecycle-Methods**
* Fehler in der `render()`-Methode irgendwo **unterhalb** der Error Boundary
* Fehler im `constructor()` einer Komponente

Tritt in einer Lifecycle-Methode, einer `render()`-Methode oder im Constructor einer Komponente nun ein Fehler auf, wird dieser von der **Error Boundary** abgefangen. Diese kann dann mit einer Fallback-Darstellung reagieren, dem Benutzer zum Neustarten der Anwendung auffordern oder darüber informieren was falsch gelaufen ist. Ähnlich wie Context-Komponenten können auch Error Boundaries ineinander verschachtelt werden. Tritt dann ein Fehler auf, greift die Implementierung der nächst höheren Error Boundary Komponente.

{% hint style="info" %}
**Achtung:** bei **Error Boundary**-Komponenten geht es primär um das Abfangen und die Behandlung von **User Interface spezifischen Fehlern** die das Rendering eines bestimmten Applikations-Status unmöglich machen. Zwar wäre es auch denkbar etwa Formular-Validierung mittels Error Boundaries zu implementieren, das würde aber dem angedachten Zweck widersprechen und ist daher nicht zu empfehlen! 
{% endhint %}

Es gibt auch Situationen in denen Error Boundaries nicht greifen. Dies ist der Fall:

* in Event-Handlern
* in asynchronemn Code, wie bspw. `setTimeout()` oder `requestAnimationFrame()`
* bei serverseitig gerenderten Komponenten \(Server Side Rendering\)
* bei Fehlern, die in der **Error Boundary** selbst auftreten

In diesen Situationen greift eine **Error Boundary** _nicht_, da es entweder nicht möglich oder nicht nötig ist. Wirft bspw. ein Event-Handler einen Fehler betrifft das nicht direkt das Rendering und React kann weiterhin ein User Interface anzeigen. Es wird dann eben nur keine entsprechende Interaktion basierend auf dem stattgefundenen Event mehr ausgeführt.

### Eine Error Boundary implementieren

Für die Implementierung einer **Error Boundary** gibt es zwei einfache Regeln: 

1. nur Klassen-Komponenten können zu einer **Error Boundary** werden
2. eine Klasse muss entweder die statische Methode `getDerivedStateFromError()` oder die Klassen-Methode `componentDidCatch()` implementieren oder auch gleich beide Methoden

Eine **Error Boundary** ist also aus technischer Sicht lediglich eine Komponente die eine der beiden o.g. Methoden oder gleich beide implementiert. Ansonsten gelten für sie genau die gleichen Regeln wie für andere Klassen-Komponenten auch.

Sehen wir uns also eine einfache Implementierung einer **Error Boundary** einmal an:

```jsx
class ErrorBoundary extends React.Component {
  state = {
    hasError: false,
  };

  static getDerivedStateFromError(error) {
    return {
      hasError: true,
      error,
    };
  }

  componentDidCatch(error, info) {
    console.log(error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Ein Fehler ist aufgetreten.</h1>;
    }

    return this.props.children; 
  }
}
```

Wir definieren hier also zuerst einmal eine neue Komponente. Diese heißt hier `ErrorBoundary`, das ist jedoch kein vorgeschriebener Name der zwingend notwendig ist. Der Name einer **Error Boundary** Komponente kann hier grundsätzlich genauso frei gewählt werden wie bei jeder anderen Komponente auch, er muss lediglich den Regeln für Komponentennamen entsprechen. Also im Wesentlichen mit einem Großbuchstaben anfangen und ein gültiger JavaScript-Funktionsname sein. 

Ich empfehle dennoch der besseren Übersicht halber bereits im Namen der Komponente erkennbar zu machen, dass es sich um eine Komponente handelt, die insbesondere zur Fehlerbehandlung dient. Also bspw. Namen wie `AppErrorBoundary` oder `DataTableErrorFallback`. So wird auch dem neuen Kollegen im Projekt schnell klar welchen Zweck die entsprechende Komponente erfüllt.

Im obigen Beispiel initialisieren wir einen State in der Komponente mit der Eigenschaft `hasError` und dessen Standardwert `false`. Schließlich ist beim Initialisieren der Komponente normalerweise noch kein Fehler aufgetreten.

Als nächstes sehen wir die statische `getDerivedStateFromError()`-Methode. An dieser Stelle signalisieren wir React, dass wir es mit einer Komponente zu tun haben die als **Error Boundary** fungiert und die zum Einsatz kommt wenn unterhalb der Komponente \(also in den Kind-Elementen\) ein Fehler auftritt. Die Methode bekommt ein `error`-Objekt hereingereicht, das auch dem Objekt entspricht, den der `catch`-Block bei einem `try`/`catch` übergeben bekommt. 

Die Methode funktioniert sehr ähnlich wie auch die `getDerivedStateFromProps()`-Methode in den Lifecycles. Sie kann ein Objekt oder `null` zurückgeben und damit einen neuen State erzeugen oder eben alles beim Alten belassen, dazu muss sie lediglich `null` zurückgeben. Im Beispiel setzen wir die `hasError`-Eigenschaft auf `true` und speichern zusätzlich das `error`-Objekt im State.

Ebenfalls implementiert haben wir hier die `componentDidCatch()`-Methode. Diese bekommt als ersten Parameter das Error-Objekt übergeben, als zweiten Parameter eine React eigene Info. Diese enthält den _„Component Stack“_, also die Information darüber in welcher Komponente der Fehler aufgetreten ist und welche Kind-Komponenten und Kindes-Kind-Komponenten involviert waren. Soll ein externer Service zum loggen der Errors benutzt werden ist hier der richtige Ort dafür.



