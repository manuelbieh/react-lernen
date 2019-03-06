# Portals

Mit **Portals** \(dt. _Portale_\) bietet React die Möglichkeit Komponenten in DOM Nodes zu rendern, die sich _außerhalb_ der Parent-Node der jeweiligen Komponenten-Hierarchie befinden, die aber dennoch Zugriff auf die aktuelle Komponenten-Umgebung haben. Ein möglicher \(aber bei weitem nicht ihr einziger\) Anwendungsfall hierfür sind u.a. Overlays, die in einem eigenen `<div>` außerhalb der tatsächlichen Anwendung gerendert werden.

Ein Portal befindet sich dabei weiter im Kontext der Komponente die das Portal erstellt, hat also Zugriff auf alle Daten die in der Eltern-Komponente zur Verfügung stehen, wie etwa die Props oder den State, befindet sich im HTML jedoch an einer ggf. völlig anderen Stelle als die restliche Anwendung. Dies ist wichtig wenn innerhalb des Portals bspw. auf Daten aus dem State der Eltern-Komponente oder wenn auf einen gemeinsamen Context wie bspw. Übersetzungen zugegriffen werden soll. 

### Ein Portal erstellen

Die Erstellung eines solchen Portals ist dabei denkbar einfach. So muss eine Komponente dazu lediglich die `createPortal()`-Methode aus ReactDOM aufrufen, ihr eine gültige Komponente als _ersten_ und eine \(existierende\) Ziel-Node als _zweiten_ Parameter übergeben.

Nehmen wir einmal folgendes HTML-Dokument an:

```markup
<!doctype html>
<html>
<head>
<title>Portale in React</title>
</head>
<body>
<div id="root"><!-- hier befindet sich unsere React App --></div>
<div id="portal"><!-- und hier landet gleich der Inhalt unseres Portals --></div>
</body>
</html>
```

Und dazu die folgende einfache React App:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const App = () => {
  return (
    <div>
      <h1>Portale in React</h1>
    </div>
  )
}

ReactDOM.render(<App />, document.querySelector('#root'));
```

Da wir unsere `<App />` in das `div` mit der id `root` rendern, sähe der `<body>` unserer obigen App nun entsprechend wie folgt aus:

```markup
<body>
  <div id="root">
    <div>
      <h1>Portale in React</h1>
    </div>
  </div>
  <div id="portal"><!-- und hier landet gleich der Inhalt unseres Portals --></div>
</body>
```

Jede weitere Komponente bzw. jedes weitere HTML Element, das wir im JSX unserer App-Komponente verwenden würde entsprechend im root-div landen. Außer eben, es handelt sich um ein **Portal**. Eine solche Komponente würde dann so aussehen:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const PortalExample = () => {
  return ReactDOM.createPortal(
    <div>Hallo aus dem Portal</div>,
    document.querySelector('#portal')
  );
}
```

Neben dem JSX das wir an der Stelle ausgeben möchten geben wir also noch den Ziel-Container an und verpacken beides zusammen hübsch in einem `ReactDOM.createPortal()`-Aufruf, den wir dann statt des reinen JSX aus der Komponente \(bzw. aus der `render()`-Methode bei Class Components\) zurückgeben. Ergänzen wir unsere Beispiel-App von oben, sähe die Benutzung wie folgt aus:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const App = () => {
  return (
    <div>
      <h1>Portale in React</h1>
      <PortalExample />
    </div>
  )
}

ReactDOM.render(<App />, document.querySelector('#root'));
```

Der `<body>` unseres HTML-Dokuments ist dann folgender:

```markup
<body>
  <div id="root">
    <div>
      <h1>Portale in React</h1>
    </div>
  </div>
  <div id="portal">
    <div>Hallo aus dem Portal</div>
  </div>
</body>
```

Das Portal wird also in die `#portal` Node gerendert statt in die `#root` Node, in der sich die Komponente befindet. Dabei ein Portal immer dann gerendert wenn die Komponente gemounted wird und folglich auch wieder aus dem DOM entfernt wenn die Komponente, die das Portal enthält aus dem Komponenten-Baum entfernt wird.

### Ein Portal im Zusammenspiel mit seiner Eltern-Komponente

Um die Funktionsweise eines Portals noch einmal deutlicher zu demonstrieren entwickeln wir im nächsten Schritt – Überraschung – ein Modal-Portal. Als Ausgangsbasis nutzen wir hierbei das identische HTML wie auch schon in der Einleitung zuvor. Wir haben also zwei divs, in die wir einmal unsere Anwendung und einmal unser Portal rendern.

Das Portal öffnen wir diesmal jedoch erst, nachdem der Benutzer einen Button geklickt hat. Im Portal selbst befindet sich dann ein Button der das Fenster wieder schließt. Dabei setzen wir die State-Eigenschaft `modalIsOpen` in der Eltern-Komponente entsprechend auf `true` oder `false`. Die `ModalPortal`-Komponente rendern wir über ein `&&` Conditional in JSX, also nur dann, wenn der Wert von `this.state.modalIsOpen` auch tatsächlich `true` ist.

In dem Moment, in dem der Wert von `false` auf `true` wechselt, wird die `ModalPortal`-Komponente gemounted und das Modal-Popup wird mit einem leicht transparenten schwarzen Hintergrund in das `<div id="portal">` gerendert. Wechselt der Wert von `true` zurück auf `false` nehmen wir es in der App-Komponente aus der Komponenten-Hierarchie heraus und React sorgt dann automatisch dafür, dass sich die `ModalPortal`-Komponente mitsamt ihres Inhalts nicht mehr in der Seite befindet.

Und im Code ergibt das dann das folgende Bild:

```jsx
import React from "react";
import ReactDOM from "react-dom";

const ModalPortal = (props) => {
  return ReactDOM.createPortal(
    <div
      style={{
        background: "rgba(0,0,0,0.7)",
        height: "100vh",
        left: 0,
        position: "fixed",
        top: 0,
        width: "100vw",
      }}
    >
      <div style={{ background: "white", margin: 16, padding: 16 }}>
        {props.children}
      </div>
    </div>,
    document.getElementById("portal")
  );
};

class App extends React.Component {
  state = {
    modalIsOpen: false,
  };

  openModal = () => {
    this.setState({ modalIsOpen: true });
  };

  closeModal = () => {
    this.setState({ modalIsOpen: false });
  };

  render() {
    return (
      <div>
        <h1>Portale in React</h1>
        <button onClick={this.openModal}>Modal öffnen</button>
        {this.state.modalIsOpen && (
          <ModalPortal>
            <p>Dieser Teil wird in einem Modal-Fenster geöffnet.</p>
            <button onClick={this.closeModal}>Modal schließen</button>
          </ModalPortal>
        )}
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById('root'));

```

Ein besonderer Augenmerk gilt hier der `this.closeModal` Methode. Diese wird als Methode der App-Komponente definiert, wird aber innerhalb der `ModalPortal`-Komponente beim Klick auf den „Modal schließen“-Button im Kontext der `App`-Komponente aufgerufen. Sie kann also problemlos den `modalIsOpen` State der Komponente verändern. Und das, obwohl die Komponente sich gar nicht innerhalb `<div id="root>` befindet, wie der Rest unserer kleinen App. Dies ist möglich, da es sich eben um ein Portal handelt, das sich aus React-Sicht im selben Komponenten-Baum wie die App selbst befindet.

