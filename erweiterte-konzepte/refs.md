# Refs

**Refs** \(also **References/Referenzen**\) erlauben es in React, direkt auf DOM-Elemente zuzugreifen, die während des Render-Prozesses erzeugt wurden. Auch wenn das in React in den meisten Fällen gar nicht notwendig ist, gibt es einige Situationen, in denen man nicht drumherum kommt, direkt auf die DOM-Elemente zuzugreifen. Etwa, wenn die Position oder die Größe eines Elements benötigt wird, um basierend darauf bspw. ein Tooltip einzublenden oder um ein Eingabefeld mittels `.focus()` nach dem Laden einer Komponente zu fokussieren.

Hier hat sich React im Laufe der Zeit weiterentwickelt und hat uns so historisch bedingt verschiedene Möglichkeiten geschaffen, solche **Refs** zu erstellen. Die **Refs**, egal in welcher Form, werden dabei stets über die `ref`-Prop eines DOM-Elements im **JSX** bzw. `createElement()`-Aufruf definiert.

Doch eine Warnung jedoch gleich noch vorweg: Auch wenn React es erlaubt, durch die Verwendung von **Refs** direkt auf DOM-Elemente zuzugreifen, sollte dies nur getan werden, wenn der deklarative Weg, also das Neurendern von Komponenten mit geänderten Props und State, nicht weiterhilft. Sämtliche Manipulation von Attributen oder Attribut-Werten, das Hinzufügen oder Entfernen bspw. von Klassen oder Event-Listenern oder das Ändern von anderen Eigenschaften wie `aria-hidden` sollte **immer deklarativ** über die Props, den State, JSX und entsprechende Re-Renderings realisiert werden!

Dennoch gibt es einige wenige Fälle in denen die Verwendung einer Ref sinnvoll oder sogar notwendig ist. Dazu gehören:

* Das Setzen eines Focus auf input-Elemente \(`input.focus()`\) oder der Aufruf anderer Methoden wie `.play()` oder `.pause()` auf `video`- und `audio`-Elementen
* Das Auslösen imperativer Animationen
* Das Lesen von Eigenschaften eines Elements im DOM \(bspw. via `.getBoundingClientRect()`\)

### String Refs

Die simpelste und älteste Variante sind die sogenannten **String Refs**. Mittlerweile wird von der Nutzung eher abgeraten, da sie die Performance beeinträchtigen können und in Zukunft ggf. entfernt werden. Der Vollständigkeit halber möchte ich sie dennoch erwähnen, da sie noch Teil der offiziellen API sind und euch gelegentlich auch noch bei der Arbeit mit React begegnen werden, insbesondere wenn ihr mit Legacy-Code zu tun habt.

Um eine **String Ref** zu definieren, gebt ihr einem DOM-Element eine Prop mit dem Namen `ref` und weist dieser Prop einen **String** als Wert zu. Das enstprechende DOM-Element ist dann **innerhalb der Komponente** in der Instanz-Eigenschaft `this.ref` zugänglich.

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class ComponentWithStringRef extends React.Component {
  componentDidMount() {
    this.refs.username.focus();
  }
  
  render() {
    return (
      <input type="text" ref="username" name="username" />
    );
  }
}

ReactDOM.render(
  <ComponentWithStringRef />, 
  document.getElementById('app')
);
```

Im obigen Beispiel können wir im `componentDidMount()` Lifecycle-Hook über `this.refs.wrapper` auf das umgebende div und über `this.refs.username` auf das Eingabefeld zugreifen. Über `this.refs.username.focus()` lässt sich letzteres dann bspw. fokussieren.

### Callback Refs

Eine Alternative zu **String Refs** sind die sogenannten **Callback Refs**. Diese erlauben mehr Flexibilität, sind aber dafür natürlich auch etwas umständlicher zu implementieren, da ihr euch um deren Handling selbst kümmern müsst. Dafür ist es mit **Callback Refs** möglich, diese an **Kind-Komponenten** weiterzugeben um auch auf DOM-Elemente innerhalb dieser zugreifen zu können.

**Callback Refs** werden, wie der Name es vermuten lässt, in **Callback-Form** definiert und bekommen beim **Mounting** als einzigen Parameter das DOM-Element oder bei Anwendung auf eine React-Komponente deren Instanz übergeben, beim **Unmounting** wird der Callback erneut aufgerufen, dann allerdings mit `null` als Parameter. 

Was ihr dann damit macht, ist euch selbst überlassen. Ein gängiger Ansatz ist es jedoch, die Referenz zu diesem DOM-Element als Instanz-Eigenschaft zu speichern, um von innerhalb der Komponente überall darauf zugreifen zu können.

Angewendet auf das obige Beispiel sähe das dann so aus:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class ComponentWithCallbackRef extends React.Component {
  usernameEl = null;

  componentDidMount() {
    this.usernameEl.focus();
  }

  setUsernameEl = (el) => {
    this.usernameEl = el;
  };
      
  render() {
    return (
      <input type="text" ref={this.setUsernameEl} name="username" />
    );
  }
}

ReactDOM.render(
  <ComponentWithCallbackRef />, 
  document.getElementById('app')
);
```

**Callback Refs** können auch in Kind-Komponenten benutzt und auf diese **Refs** dann von innerhalb der Eltern-Komponente zugegriffen werden. Dazu übergebt ihr die Callback-Funktion der Kind-Komponente in einer eigenen Prop, die nicht `ref` heißen darf, da dies ja ein reservierter Name ist:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class UsernameInput extends React.Component {
  render() {
    return (
      <input type="text" name="username" ref={this.props.inputRef} />
    );
  }
}

class ComponentWithRefChild extends React.Component {  
  componentDidMount() {
    this.usernameEl.focus();
  }

  usernameEl = null;

  setUsernameRef = (el) => {
    this.usernameEl = el;
  }

  render() {
    return (
      <div>
        Username:
        <UsernameInput inputRef={this.setUsernameRef} />
      </div>
    );
  }
}

ReactDOM.render(
  <ComponentWithRefChild />, 
  document.getElementById('app')
);
```

Würdet ihr die Prop hier `ref` nennen, also `<UsernameInput ref={this.setUsernameRef} />`, würdet ihr stattdessen eine Referenz zur `UsernameInput`-**Instanz** erhalten, statt zu deren Input-Element. Bei **Function Components** wäre `UsernameInput` sogar `null`, da SFCs nicht instanziiert werden!

Für die Weiterleitung von Refs in Kind-Komponenten sollte nach Möglichkeit jedoch die `forwardRef()`-Methode verwendet werden. Diese wird am Ende dieses Kapitels beschrieben.

### Refs über createRef\(\)

Neu in React 16.3. eingeführt wurde die Top-Level-Methode `React.createRef()`. Sie ähnelt von der Art der Verwendung her ein wenig den **Callback Refs**, jedoch mit kleinen Unterschieden. So müsst ihr euch auch hier um das Handling selbst kümmern. Durch ihre Ähnlichkeit zu **Callback Refs**, ist es auch hier gängige Praxis, die **Refs** einer **Instanz-Eigenschaft** zuzuweisen.

Statt jedoch jedesmal eine nahezu identische Methode in der Form `(el) => { this.property = el }` zu übergeben, erstellt ihr bei der Instanziierung der Komponente bereits die Referenz und übergebt diese dann an die `ref`-Prop des jeweiligen Elements.

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class ComponentWithCreatedRef extends React.Component {
  usernameEl = React.createRef();

  componentDidMount() {
    this.usernameEl.current.focus();
  }

  render() {
    return (
      <input type="text" name="username" ref={this.usernameEl} />
    );
  }
}

ReactDOM.render(
  <ComponentWithCreatedRef />, 
  document.getElementById('app')
);
```

Vom Prinzip her also sehr ähnlich zu den **Callback Refs**, allerdings mit einem weiteren entscheidenden Unterschied: Auf die entsprechende Referenz greift ihr hier via `this.usernameEl.current` zu. 

Die Referenz zum Element wird hier also nicht in der Instanz-Eigenschaft gespeichert, der ihr die Ref zuordnet, sondern in deren `.current` Eigenschaft. Ansonsten ist ihr Verhalten soweit vergleichbar mit den Callback Refs. Ihr könnt diese ebenfalls an Kind-Komponenten über deren Props weitergeben und dann aus der Eltern-Komponente auf das jeweilige DOM-Element zugreifen.

Im direkten Vergleich hier noch einmal die **Callback Ref**:

```jsx
class MyComponent extends React.Component {
  usernameEl = null;
  
  render () {
    return (
      <input ref={(el) => { this.usernameEl = el; }} />
    )
  }
}
```

Zugriff auf das Element via:  `this.usernameEl` 

Und alternativ dazu die mittels **`React.createRef()`** erstellte Referenz:

```jsx
class MyComponent extends React.Component {
  usernameEl = React.createRef();
  
  render () {
    return (
      <input ref={this.usernameEl} />
    )
  }
}
```

Zugriff auf das Element via: `this.usernameEl.current`.

### Weiterleitung von Refs

Das sog. **Ref forwarding**, also die _Weiterleitung von Refs_ \(Referenzen zu einer Komponente oder einem DOM Element\) ermöglicht es, eine Referenz durch eine Komponente hindurch zu einer Kind-Komponente zu übergeben. Dies ist in den allermeisten Fällen nicht notwendig, kann manchmal aber wichtig werden, insbesondere wenn man eine wiederverwendbare Komponenten-Bibliothek erstellt. 

Weitergeleitet wird eine **Ref** über die Methode `React.forwardRef()`. Sie bekommt dabei als Parameter eine Funktion übergeben, an die sie wiederum Props, sowie die **Ref** übergibt.

Klingt wieder fürchterlich umständlich, werfen wir also einen Blick auf ein Code-Beispiel. Im folgenden Beispiel implementieren wir zunächst eine eigene Input-Komponente ohne ForwardRef:

```jsx
import React from "react";
import ReactDOM from "react-dom";

const UsernameField = (props) => (
  <div className="myInput">
    <input ref={props.ref} {...props} />
  </div>
);

class App extends React.Component {
  usernameEl = React.createRef();

  componentDidMount() {
    console.log(this.usernameEl);
  }

  render() {
    return (
      <div className="App">
        <UsernameField ref={this.usernameEl} />
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById("root"));
```

Hier hätten wir in der `componentDidMount()` Lifecycle Methode keinen Zugriff auf unser Eingabefeld aus der `UsernameField`-Komponente. Stattdessen wäre die Instanz der Komponente selbst die **Ref**. Da `UsernameField` jedoch eine **Function Component** ist, existiert nicht mal eine Instanz der Komponente. Somit wäre die `console.log` Ausgabe an der Stelle: `{ current: null }` - unschön, wollen wir doch Zugriff auf das `input`-Element bekommen, um es bspw. fokussieren zu können.

Dazu reicht es aus, die `UsernameField`-Komponente von einem `React.forwardRef()`-Aufruf umschließen zu lassen. Im obigen Beispiel ändern wir die Komponente also wie folgt:

```jsx
const UsernameField = React.forwardRef((props, ref) => (
  <div className="myInput">
    <input ref={ref} {...props} />
  </div>
));
```

Hier teilen wir React mit, dass es die `ref`-Prop auf dem `UsernameField` in unserer `App`-Komponente bitte an die Komponente weiterleiten soll. Diese bekommt die **Ref** dann als zweiten Parameter der Funktion übergeben und kann diese an ein beliebiges Element im DOM oder auch an eine weitere Komponente weiterreichen. 

Wird die **Ref** an eine weitere Komponente tiefer im Baum weitergereicht sollte beachtet werden dass hier die selben Restriktionen gelten: Die entsprechende Komponente muss entweder eine Klassen-Komponente sein - dann würde die Referenz auf die Instanz der Klasse zeigen - oder die Komponente muss ihrerseits wiederum eine Ref-Weiterleitung via `forwardRef()` machen.

#### Vorsicht bei Higher Order Components!

Bei der Implementierung von **Higher Order Components** ist Vorsicht geboten. Ist unklar, ob in ihnen auf weitergeleitete Refs zugegriffen werden soll, müssen sie selbst von einem `forwardRef()`-Aufruf umschlossen werden.

Erweitern wir unser Beispiel von oben und nehmen an, wir wollen eine **HOC** erstellen um Formular-Elemente in einem bestimmten einheitlichen Stil anzuzeigen. Dazu erstellen wir die **HOC** `withInputStyles`. Diese kann und wird `input`-Elemente umschließen und in diesen soll es nicht ausgeschlossen sein, dass wir ihnen eine Ref zuweisen. 

Das ganze Verfahren ist etwas kompliziert und es fiel mir enorm schwer es in Worte zu fassen, die leicht verständlich gewesen wären, weshalb ich hier stattdessen einmal mehr den mit Kommentaren versehenen Code sprechen lassen möchte. Sobald das Prinzip von **Higher Order Components** und **forwardRefs** klar ist, sollte das Beispiel ausreichend sein um die Implementierung zu verstehen. Und selbst wenn nicht, ein kleiner Trost vorab: Das ist ein Anwendungsfall der in der Praxis so selten sein sollte, dass man ihm in den seltensten Fällen in einer echten Anwendung benötigen wird. 

Der Vollständigkeit halber möchte ich ihn hier dennoch erwähnt haben. Hier also das Beispiel:

```jsx
import React from "react";
import ReactDOM from "react-dom";

const withInputStyles = (InputComponent) => {
  class WithInputStyles extends React.Component {
    render() {
      // Wir holen uns die weitergeleitete Ref aus den Props der Komponente
      const { forwardedRef, ...props } = this.props;
      
      // ... und setzen sie als Ref für die von der HOC umschlossenen 
      // Komponente ein:
      return (
        <InputComponent
          {...props}
          ref={forwardedRef}
          style={{ border: "2px solid black", padding: 8 }}
        />
      );
    }
  }
  
  // Wir geben einen ForwardRef aus der HOC zurück
  return React.forwardRef((props, ref) => (
    // Wir geben die Ref als temporäre Prop `forwardedRef` weiter
    <WithInputStyles {...props} forwardedRef={ref} />
  ));
};

// Hier leiten wir die Ref unserer Komponente an das input weiter mittels 
// eines gewöhnlichen React.forwardRef()-Aufrufs
const UsernameField = React.forwardRef((props, ref) => (
  <input ref={ref} {...props} />
));

// Hier verbinden wir die HOC mit unserer UsernameField-Komponente
const StyledUsername = withInputStyles(UsernameField);

class App extends React.Component {
  // Hier wird ganz gewöhnlich die Ref erstellt auf die wir später zugreifen
  usernameEl = React.createRef();

  componentDidMount() {
    this.usernameEl.current.focus();
  }

  render() {
    return (
      <div>
        <StyledUsername ref={this.usernameEl} />
      </div>
    );
  }
}
```



