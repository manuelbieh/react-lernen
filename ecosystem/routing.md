# Routing

Eine Funktionalität die in nahezu allen **Single Page Applikationen** \(SPA\) früher oder später \(meist früher als später\) benötigt wird ist das Routing. Also das Zuordnen einer URL in der Anwendung zu einer bestimmten Funktion. Rufe ich bspw. die URL `/users/manuel` auf, möchte ich dort sehr wahrscheinlich das Benutzerprofil des Users `manuel` anzeigen.

Hier hat sich der **React Router** in den vergangenen Jahren als de facto Standard etabliert. Entwickelt von Michael Jackson \(ja, der Kerl heißt wirklich so!\) und Ryan Florence \(der laut eigener Aussage inzwischen über 10 verschiedene Router für diverse Zwecke entwickelt hat\) bringt er es auf mittlerweile über 35.000 Stars bei GitHub. Er wird regelmäßig gepflegt, hat eine Community bestehend aus über 500 Contributors auf GitHub und passt sich durch seine deklarative Natur wunderbar die React-Prinzipien an. Darüber hinaus ist er kompatibel sowohl mit dem Web \(client- und serverseitig!\) wie auch React Native. Er ist also sehr universell einsetzbar, sehr gut getestet und durch seine weite Verbreitung auch bewährt.

Dabei ist sein Interface selbst ziemlich simpel. In ca. 95% der Zeit wird man mit lediglich fünf Komponenten in Berührung kommen: `BrowserRouter`, `Link`, `Route`, `Redirect` und `Switch`. Darüber hinaus gibt es noch die imperative History API die auf dem gleichnamigen `history` Package basiert und die HOC `withRouter` um für das Routing relevante Daten aus dem Router in eine Komponente herein zu reichen. 

Installiert wird er via:

```bash
npm install --save react-router-dom
```

bzw.

```bash
yarn add react-router-dom
```

Die generelle Benutzung ist dabei wie bereits angesprochen _deklarativ_, also erfolgt in Form der oben erwähnten Komponenten. Router können dadurch innerhalb einer Anwendung an jeder beliebigen Stelle verwendet werden. Voraussetzung ist lediglich, dass der jeweilige Seitenbaum sich in einem **Router Context** befindet. Dieser existiert in einer typischen Anwendung nur ein einziges Mal und legt sich meist ganz außen um die Anwendung. Etwa in der folgenden Form:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router } from 'react-router-dom';

const App = () => {
  return (
    <Router>
      [...]
    </Router>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

Jede Komponente innerhalb des `<Router></Router>`-Elements kann nun auf den Router Context zugreifen, darauf reagieren und ihn steuern. Verschiedene Routen legen wir durch die Verwendung der Route-Komponente an, die immer zwingend eine `path`-Prop enthalten muss und wahlweise eine `render`-Prop oder eine `component`-Prop enthält. Der Unterschied liegt hier darin, dass der Wert der `render`-Prop eine **Funktion** sein muss \(hier sei auch nochmal an das entsprechende Kapitel zu **Render-Props** erinnert\), während die `component`-Prop eine **Komponente** \(kein _Element!_\) erwartet. 

Also sieht eine korrekte Verwendung beider Props z.B. so aus:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router } from 'react-router-dom';

const Example = () => <p>Example Komponente</p>

const App = () => {
  return (
    <Router>
      <Route path="/example" component={Example} />
      <Route path="/example" render={() => <Example />} />
    </Router>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

In diesem Beispiel würde die `Example`-Komponente beim Aufruf der `/example` URL zweimal gerendert werden, da die Route-Komponente lediglich überprüft ob der Pfad der aktuellen URL mit dem in der `path`-Prop angegebenen Wert übereinstimmt. Dies mag erst einmal verwunderlich klingen, lässt sich aber ganz logisch erklären.

Da React Router eben deklarativ ist, sagen wir React mit der Angabe zweier gleicher Routen eben erst einmal, dass wir auch zwei Komponenten rendern wollen wenn die URL übereinstimmt. Dies kann dann sinnvoll sein, wenn sich verschiedene Teile einer Seite unabhängig voneinander und basierend auf der URL ändern sollen. Angenommen wir haben eine App mit einer Sidebar und einem Content-Bereich. Beide sollen nun auf eine URL reagieren:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router } from 'react-router-dom';

import { Home, User } from './MainDummies';
import { HomeSidebar, UserSidebar } from './SidebarDummies';

const App = () => {
  return (
    <Router>
      <main>
        <Route path="/user" component={User} />
        <Route path="/" component={Home} />
      </main>
      <aside>
        <Route path="/user" component={UserSidebar} />
        <Route path="/" component={HomeSidebar} />
      </aside>
    </Router>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

In diesem Beispiel sehen wir wie an zwei verschiedenen Stellen in unserer Anwendung unterschiedliche Komponenten verwendet werden, je nachdem welche URL momentan aufgerufen wird.

