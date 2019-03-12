# Code Splitting

Wer ein Projekt mit React entwickelt nutzt in den allermeisten Fällen auch einen **Bundler** wie **Webpack,** **Browserify** oder **Rollup**. Diese sorgen dafür, dass alle einzelnen Dateien, alle Imports, später zu einer einzigen großen Dateien gebündelt wird die dann relativ einfach deployed werden kann ohne dass sich ein Entwickler noch all zu viele Gedanken um relative Verlinkungen machen muss. Dieser Vorgang wird dementsprechend **Bundling** genannt. In sehr großen und komplexen Projekten kann so ein **Bundle** schnell mal ein Megabyte groß oder gar größer werden, insbesondere wenn viele Third Party Bibliotheken im Einsatz sind. Das ist in vielerlei Hinsicht ein Problem, denn große Bundles benötigen länger um vom Browser heruntergeladen zu werden und auch das Ausführen unnötig großer Bundles führt unweigerlich zu Performance-Einbußen.

Um dem Problem der großen Bundles zu begegnen gibt es das sog. **Code Splitting**. Beim Code Splitting wird die Anwendung in mehrere kleinere Bundles aufgesplittet die allesamt für sich allein gesehen lauffähig sind und weitere Bundles nachladen, sollten diese später benötigt werden. So ist die Aufteilung in ein Bundle mit den meist benutzten Abhängigkeiten \(bspw. React, React DOM, ...\) und jeweils ein Bundle pro Route eine recht gängige Methode beim **Code Splitting**.

Die einfachste Methode dazu ist die Verwendung der **Dynamic Import** Syntax. Diese ist momentan ein Proposal beim **TC39**, befindet sich also momentan im Standardisierungsprozess. Dank Webpack und Babel ist es aber auch heute schon möglich die Verwendung zu benutzen. Notwendig ist hierfür das Babel Plugin `@babel/plugin-syntax-dynamic-import`. Create React App und andere Tools wie next.js oder Gatsby bringen von Haus aus Unterstützung für dynamische Imports mit und müssen nicht speziell für die Verwendung von Code Splitting konfiguriert werden.

### Verwendung dynamischer Imports

Im Kapitel zu ES2015+ wurde die Import-Syntax bereits angesprochen. Die Dynamic Import Syntax ist eine Erweiterung dieser und erlaubt es innerhalb einer Anwendung Imports dynamisch nachzuladen, daher der Name. Dabei funktioniert ein dynamischer Import nicht anders als ein Promise:

```jsx
// greeter.js
export sayHi = (name) => `Hi ${name}!`;
```

```jsx
// app.js
import('./greeter').then((greeter) => {
  console.log(greeter.sayHi('Manuel'); // "Hi Manuel!"
});
```

Findet Webpack einen dynamischen Import nutzt Webpack an dieser Stelle automatisch seine Code-Splitting Funktion und lagert die entsprechende Datei beim Erstellen des Bundles in ein eigenes kleines Bundle aus dass es dann selbstständig lädt sobald dieses in der Anwendung benötigt wird. Dieses Verhalten wird allgemein als **Lazy Loading** bezeichnet.

### React.lazy\(\)

Und damit wären wir auch schon beim nächsten Thema: Lazy Loading mit React. Um die Entwickler-Erfahrung beim **Lazy Loading** möglichst angenehm zu gestalten, bietet React seit Version **16.6.** eine hauseigene Methode um Komponenten dynamisch nachzuladen. Diese wird kombiniert mit der **Dynamic Import Syntax** und erlaubt es dem Entwickler bestimmte React-Komponenten erst zur Laufzeit der Anwendung zu laden und so die Größe der Bundles weiter zu verkleinern. 

Ein via `React.lazy()` geladener Import kann in React als gewöhnliche Komponente verwendet werden. Ihr können Props übergeben werden wie auch Refs. Sie kann eigene Kind-Elemente beinhalten oder in sich geschlossen sein. Die Methode erwartet eine Funktion als Parameter, die einen dynamischen Import zurückgibt. Dieser Import muss eine Komponente importieren, die einen Default Export hat, die wiederum eine React-Komponente sein muss:

```jsx
// LazyLoaded.js
import React from 'react';

const LazyLoaded = () => (
  <p>Diese Komponente wird erst bei ihrer Verwendung vom Server geladen</p>
);
```

```jsx
// app.js
import React, { Suspense } from 'react';
import ReactDOM from 'react-dom';

const LazyLoaded = React.lazy(() => import('./LazyLoaded.js'));

const App = () => (
  <Suspense fallback={<div>Anwendung wird geladen</div>}>
    <LazyLoaded />
  </Suspense>
);

ReactDOM.render(<App />, document.getElementById("root"));
```

In komplexen Komponenten und insbesondere bei wachsenden Anwendungen kann man so sehr schön die Größe des JavaScript Bundles optimieren und relevante Dateien erst dann vom Server laden wenn diese wirklich benötigt werden. Während die Datei vom Server geladen bis sie schließlich ausgeführt wurde wird an der Stelle der Komponente der Hinweis `<div>Anwendung wird geladen</div>` angezeigt. Dafür sorgt eine weitere Neuerung, die in Version 16.6. den Weg in React fand:

### Suspense

Diese Komponente hieß in ihrer ursprünglichen Version einmal **Placeholder** \(dt. _Platzhalter_\) und das beschreibt ziemlich genau was sie macht: sie agiert als Platzhalter für Komponenten die noch nicht geladen wurden und rendert eine Alternative. Dies kann wie im obigen Beispiel bspw. eine Nachricht sein, dass Teile der Anwendung geladen werden oder auch eine ganz klassische Lade-Animation. Der Platzhalter wird dabei als `fallback`-Prop angegeben und muss zwingend definiert werden. Als gültiger Wert der Prop kann jedes beliebige valide React-Element verwendet werden. Dazu gehören auch Strings. `<Suspense fallback="Wird geladen">[…]</Suspense>` wäre demnach also ebenfalls ein valider Platzhalter.

Solange die zu ladende Komponente noch nicht vollständig geladen wurde, werden dann sämtliche Kind-Elemente des `Suspense`-Elements durch den festgelegten Platzhalter ersetzt und erst wenn die Komponente vollständig geladen wurde durch den eigentlichen Inhalt ersetzt. Dabei können beliebig viele via `React.lazy()` geladene Komponenten innerhalb eines `Suspense`-Elements verwendet werden. Der `fallback`-Platzhalter wird dann so lange angezeigt bis **sämtliche** Komponenten vollständig geladen sind und angezeigt werden können!

Auch eine Verschachtelung ist möglich und teilweise sogar sinnvoll. Gibt es z.B. Teile in der Seite die eher unwichtig sind und das Rendering des User Interfaces nicht verzögern sollten wenn andere, wichtigere Teile bereits geladen sind, so kann der entsprechende Seitenbaum von einem eigenen `Suspense`-Element umschlossen werden. Dies führt dazu, dass die anderen, möglicherweise wichtigeren Teile des Interfaces angezeigt werden sobald diese geladen sind, während für die anderen, unwichtigeren Teile erneut ein Platzhalter angezeigt wird.

Ein denkbares Szenario könnte eine Anwendung zur Bildbearbeitung sein. Hier kann es sinnvoll sein das zu bearbeitende Bild bereits anzuzeigen sobald die Komponente, die für die Anzeige des Bildes verantwortlich ist, geladen wurde. Das User Interface mit den eigentlichen Funktionen zur Bearbeitung wird dann erst im nächsten Schritt angezeigt, sollte das Laden der entsprechenden Komponente länger dauern. Und nur dann.

```jsx
import React, { Suspense } from "react";
import ReactDOM from "react-dom";

const ImageCanvas = React.lazy(() => import("./ImageCanvas"));
const ImageToolbar = React.lazy(() => import("./ImageToobar"));

function App() {
  return (
    <Suspense fallback={<div>Anwendung wird geladen</div>}>
      <ImageCanvas url="https://via.placeholder.com/350x240"/>
      <Suspense fallback={<div>Bearbeitungsfunktionen werden geladen</div>}>
        <ImageToolbar />
      </Suspense>
    </Suspense>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));

```

Was hier passiert ist folgendes: die `ImageCanvas` \(zum Anzeigen des Bildes\) und die `ImageToolbar` \(für die Bearbeitungsfunktionen\) befinden sich in einem `Suspense`-Element. Dieses zeigt die Platzhalter-Nachricht: _„Die Anwendung wird geladen“_  solange, bis die `ImageCanvas`-Komponente vom Server geladen wurde. 

Sollte dies passieren **bevor** die `ImageToolbar` geladen wurde, wird sie bereits angezeigt und das zweite, innere `Suspense`-Element kommt zum Einsatz. Dieses sorgt dafür, dass die Nachricht _„Bearbeitungsfunktionen werden geladen“_ angezeigt werden und zwar bis diese tatsächlich geladen wurden.

Ist die `ImageCanvas`-Komponente erst geladen **nachdem** die `ImageToolbar`-Komponente geladen wurde wird das innere `Suspense`-Element aufgelöst, jedoch verhindert das äußere Suspense-Element die anzeige der Toolbar und zeigt diese erst an sobald auch die `ImageCanvas` geladen ist. Dann jedoch ohne weitere Verzögerung.

Unser UI kennt also drei mögliche Darstellungen:

* ImageCanvas und ImageToolbar wurden erfolgreich geladen und werden beide dargestellt
* ImageCanvas wurde noch nicht geladen und es erscheint nur die „Anwendung wird geladen“ Nachricht, unabhängig vom Lade-Status der ImageToolbar
* ImageCanvas wude geladen, ImageToolbar jedoch noch nicht. Dann wäre die ImageCanvas bereits sichtbar, anstelle der Toolbar stünde jedoch der Hinweis „Bearbeitungsfunktionen werden geladen“

Wir schließen somit also bewusst aus, dass ein Benutzer zwar bereits die Bearbeitungsfunktionen für ein Bild sieht, nicht jedoch die Zeichenfläche auf der das zu bearbeitende Bild angezeigt wird. Eine kluge Verschachtelung von `Suspense` gibt uns also alle Flexibilität die nötig ist um sehr fein granular festzulegen wann welche Teile der Anwendung bereits angezeigt werden sollen und wo wir vorübergehend einen Platzhalter anzeigen wollen.

Momentan wird **Suspense** als Platzhalter offiziell nur für das Laden von Komponenten mittels `React.lazy()` unterstützt. In Zukunft soll auch das asynchrone Laden von Daten verschiedenster Art \(wie z.B. API Abfragen\) durch **Suspense** unterstützt werden.

**Vorsicht:** momentan werden **Lazy** und **Suspense** nur bei der Verwendung in clientseitigen Anwendungen unterstützt. Unterstützung für serverseitiges Rendering gibt es für dieses Feature aktuell noch nicht und befindet sich noch in Arbeit.

