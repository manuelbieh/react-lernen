# Code Splitting

Wer ein Projekt mit React entwickelt nutzt in den allermeisten Fällen auch einen **Bundler** wie **Webpack,** **Browserify** oder **Rollup**. Diese sorgen dafür, dass alle einzelnen Dateien, alle Imports, später zu einer einzigen großen Dateien gebündelt wird die dann relativ einfach deployed werden kann ohne dass sich ein Entwickler noch all zu viele Gedanken um relative Verlinkungen machen muss. Dieser Vorgang wird dementsprechend **Bundling** genannt. In sehr großen und komplexen Projekten kann so ein **Bundle** schnell mal ein Megabyte groß oder gar größer werden, insbesondere wenn viele Third Party Bibliotheken im Einsatz sind. Das ist in vielerlei Hinsicht ein Problem, denn große Bundles benötigen länger um vom Browser heruntergeladen zu werden und auch das Ausführen unnötig großer Bundles führt unweigerlich zu Performance-Einbußen.

Um dem Problem der großen Bundles zu begegnen gibt es das sog. **Code Splitting**. Beim Code Splitting wird die Anwendung in mehrere kleinere Bundles aufgesplittet die allesamt für sich allein gesehen lauffähig sind und weitere Bundles nachladen, sollten diese später benötigt werden. So ist die Aufteilung in ein Bundle mit den meist benutzten Abhängigkeiten \(bspw. React, React DOM, ...\) und jeweils ein Bundle pro Route eine recht gängige Methode beim **Code Splitting**.

Die einfachste Methode dazu ist die Verwendung der **Dynamic Import** Syntax. Dynamic Import ist momentan ein Proposal beim **TC39**, befindet sich also momentan im Standardisierungsprozess. Dank Webpack und Babel ist es aber auch heute schon möglich die Verwendung zu benutzen. Notwendig ist hierfür das Babel Plugin `@babel/plugin-syntax-dynamic-import`. Create React App und andere Tools wie next.js oder Gatsby bringen von Haus aus Unterstützung für dynamische Imports mit und müssen nicht speziell für die Verwendung von Code Splitting konfiguriert werden.

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

Und damit wären wir auch schon beim nächsten Thema: Lazy Loading mit React. Um die Entwickler-Erfahrung beim **Lazy Loading** möglichst angenehm zu gestalten, bietet React seit Version **16.6.** eine hauseigene Methode um Komponenten dynamisch nachzuladen. Diese wird kombiniert mit der Dynamic Import Syntax und erlaubt es dem Entwickler bestimmte React-Komponenten erst zur Laufzeit der Anwendung zu laden und so die Größe der Bundles weiter zu verkleinern. 

Ein via `React.lazy()` geladener Import kann innerhalb von React als gewöhnliche Komponente verwendet werden. Ihr können Props übergeben werden wie auch Refs. Sie kann eigene Kind-Elemente beinhalten oder in sich geschlossen sein.

