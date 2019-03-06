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
* in asynchronem Code, wie bspw. `setTimeout()` oder `requestAnimationFrame()`
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

Wir definieren also zuerst einmal eine neue Komponente. Diese heißt hier `ErrorBoundary`, das ist jedoch kein vorgeschriebener Name der zwingend notwendig ist. Der Name einer **Error Boundary** Komponente kann hier grundsätzlich genauso frei gewählt werden wie bei jeder anderen Komponente auch, er muss lediglich den Regeln für Komponentennamen entsprechen. Also im Wesentlichen mit einem Großbuchstaben anfangen und ein gültiger JavaScript-Funktionsname sein. 

Ich empfehle dennoch der besseren Übersicht halber bereits im Namen der Komponente erkennbar zu machen, dass es sich um eine Komponente handelt, die insbesondere zur Fehlerbehandlung dient. Also bspw. Namen wie `AppErrorBoundary` oder `DataTableErrorFallback`. So wird auch dem neuen Kollegen im Projekt schnell klar welchen Zweck die entsprechende Komponente erfüllt.

Im obigen Beispiel initialisieren wir einen State in der Komponente mit der Eigenschaft `hasError` und dessen Standardwert `false`. Schließlich ist beim Initialisieren der Komponente normalerweise noch kein Fehler aufgetreten.

Als nächstes sehen wir die statische `getDerivedStateFromError()`-Methode. An dieser Stelle signalisieren wir React, dass wir es mit einer Komponente zu tun haben die als **Error Boundary** fungiert und die zum Einsatz kommt wenn unterhalb der Komponente \(also in den Kind-Elementen\) ein Fehler auftritt. Die Methode bekommt ein `error`-Objekt hereingereicht, das auch dem Objekt entspricht, den der `catch`-Block bei einem `try`/`catch` übergeben bekommt. 

Die Methode funktioniert sehr ähnlich wie auch die `getDerivedStateFromProps()`-Methode in den Lifecycles. Sie kann ein Objekt oder `null` zurückgeben und damit einen neuen State erzeugen oder eben alles beim Alten belassen, dazu muss sie lediglich `null` zurückgeben. Im Beispiel setzen wir die `hasError`-Eigenschaft auf `true` und speichern zusätzlich das `error`-Objekt im State. Da die Methode statisch ist kann sie jedoch nicht auf andere Methoden der Komponente zugreifen. 

Die `getDerivedStateFromError()`-Methode wird während der _Render_-Phase ausgeführt. Also während React den letzten Stand des Komponenten-Baums mit dem aktuellen vergleicht und bevor die neuen Änderungen letztlich in den DOM geschrieben \(_„commited“_\) werden.

Ebenfalls implementiert haben wir hier die `componentDidCatch()`-Methode. Diese bekommt als ersten Parameter das Error-Objekt übergeben, als zweiten Parameter eine React eigene Info. Diese enthält den _„Component Stack“_, also die Information darüber in welcher Komponente der Fehler aufgetreten ist und welche Kind-Komponenten und Kindes-Kind-Komponenten involviert waren, bildet also den Komponenten-Baum bis zur fehlerhaften Komponente ab. Soll ein externer Service zum loggen der Errors benutzt werden ist hier der richtige Ort dafür, denn das ist der beabsichtigte Zweck dieser Methode. Hier sollen Side Effects stattfinden. Die Methode wird erst in der _Commit_-Phase ausgefüht, also nachdem React die Änderungen am State im DOM abgebildet hat. 

Da `componentDidCatch()` keine statische Methode ist, wäre es hier zwar auch möglich den State der Komponente mittels `this.setState()` zu modifizieren, allerdings ist geplant dies in Zukunft zu unterbinden weshalb hier mit Bedacht vorgegangen werden sollte. Der bessere Weg ist in jedem Fall die statische `getDerivedStateFromError()`-Methode zu verwenden um einen neuen State nach Auftreten eines Fehlers zu erzeugen und auf den Fehler zu reagieren.

Zuletzt reagieren wir auf einen möglicherweise aufgetretenen Fehler in unserer render\(\)-Methode. Ist die `hasError`-Eigenschaft im State der Komponente `true` wissen wir, dass ein Fehler aufgetreten ist und geben eine Meldung `<h1>Ein Fehler ist aufgetreten</h1>` auf. Ist alles in Ordnung geben wir lediglich die Kind-Elemente der Komponente \(`this.props.children`\) zurück. Wie genau auf den Fehler reagiert wird bleibt dabei dem Entwickler überlassen. Denkbar wäre es bei schweren Fehlern den Benutzer zum Neuladen der Anwendung aufzufordern oder bei kleineren Fehlern nur einen Hinweis an Ort und Stelle anzuzeigen, dass eine Komponente momentan nicht angezeigt werden kann.

### Eine Error Boundary verwenden

Wir wissen nun wie wir eine **Error Boundary** implementieren: durch das Hinzufügen von `static getDerivedStateFromError()` oder `componentDidCatch()` in einer Komponente. **Error Boundaries** sollten dabei so unabhängig wie möglich sein und wenig bis gar keine eigene Logik implementieren oder gar zu eng an bestimmte Komponenten gekoppelt werden. Wie granular eine solche **Error Boundary** aber letztendlich ist liegt in der Entscheidung des Entwicklers. 

So ist es in komplexen Anwendungen durchaus sinnvoll verschiedene, auch ineinander verschachtelte, **Error Boundaries** zu haben die jeweils unterschiedliche Fehlerfälle abdecken. Eine, die sich um die komplette Anwendung legt und Fehler sämtliche Fehler abfängt, eine weitere für bestimmte Bereiche im Seitenbaum die vielleicht nur optional sind. Schauen wir uns auch das einmal anhand eines Beispiels an:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const App = () => {
  return (
    <ErrorBoundary>
      <ApplicationLogic />
      <ServiceUnavailableBoundary>
        <WeatherWidget />
      </ServiceUnavailableBoundary>
    </ErrorBoundary>
  )
};

ReactDOM.render(<App />, document.querySelector('#root'));
```

Hier haben wir es mit zwei Error Boundary Komponenten zu tun: `ErrorBoundary` und `ServiceUnavailableBoundary`. Während die äußere `ErrorBoundary`-Komponente durchaus 1:1 das obige Beispiel repräsentieren könnte, also lediglich ein `<h1>Ein Fehler ist aufgetreten.</h1>` ausgibt sollte in der `ApplicationLogic`-Komponente ein fehler auftreten könnte die `ServiceUnavailableBoundary`-Komponente eine alternative Meldung ausgeben wie bspw _„Der angeforderte Dienst ist momentan nicht erreichbar. Bitte versuchen Sie es später wieder“_, wenn der Fehler in der \(fiktiven\) `WeatherWidget`-Komponente auftritt.

Tritt in der `WeatherWidget`-Komponente ein Fehler auf wird dieser dann von der `ServiceUnavailableBoundary` abgefangen und alles was in der `ApplicationLogic`-Komponente gerendert wird bleibt intakt! Würden wir das `WeatherWidget` nicht in seiner eigene **Error Boundary** einschließen, würde die äußere **Error Boundary** greifen und auch die `ApplicationLogic`-Komponente nicht mehr anzeigen!

Generell lässt sich sagen dass es empfehlenswert ist immer mindestens eine **Error Boundary** in seiner Anwendung zu haben die sich möglichst weit oben in der Komponenten-Hierarchie befindet und sämtliche unerwartete Fehler abfängt \(und möglichst logged!\) und ähnlich wie eine `500 Internal Server Error` Seite agiert. Nach Bedarf kann und sollte man dann weitere **Error Boundaries** hinzufügen und bestimmte Seitenbäume damit umschließen. Je nachdem wie groß die Gefahr eines Rendering-Fehlers in einem bestimmten Bereich eines Baums ist \(bspw. weil man mit unbekannten oder mit sich-ändernden Daten zu tun hat\) oder wie stark vernachlässigbar ein bestimmter Komponentenbaum ist.

{% hint style="info" %}
Seit Version 16 werden Komponenten in denen ein schwerer Fehler auftritt, kurz gesagt in denen ein Error/Exception geworfen wird durch React komplett „unmounted“ und verschwinden aus dem Seitenbaum. Dies ist wichtig, damit bei einem unbehandelten Fehler das User Interface nicht plötzlich in einem unerwarteten und fehlerhaften Zustand bleibt. Dies ist bspw. in einer Online-Banking Anwendung kritisch, wenn Überweisungen dann an einen falschen Empfänger gehen oder ein falscher Betrag überwiesen wird.

Um derartige Fehler im Rendering von Komponenten vernünftig behandeln zu können wurden die **Error Boundaries** eingeführt. Durch sie können Benutzer auf den Fehlerhaften Zustand der Anwendung hingewiesen werden. Da Fehler in einer Anwendung nie ganz ausgeschlossen werden können \(Benutzer können hier sehr einfallsreich sein!\) ist die Verwendung von **Error Boundaries** sehr zu empfehlen!
{% endhint %}



