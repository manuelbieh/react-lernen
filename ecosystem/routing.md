# Routing

Eine Funktionalität die in nahezu allen **Single Page Applikationen** \(SPA\) früher oder später \(meist früher als später\) benötigt wird ist das **Routing**. Also das Zuordnen einer URL in der Anwendung zu einer bestimmten Funktion. Rufe ich bspw. die URL `/users/manuel` auf, möchte ich dort sehr wahrscheinlich das Benutzerprofil des Users `manuel` anzeigen.

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

Jede Komponente innerhalb des `<Router></Router>`-Elements kann nun auf den Router Context zugreifen, darauf reagieren und ihn steuern. Verschiedene Routen legen wir durch die Verwendung der Route-Komponente an, die eine `path`-Prop enthalten sollte \(Ausnahme: 404 Fehler-Routen\) und wahlweise eine `render`-Prop oder eine `component`-Prop enthält. Der Unterschied liegt hier darin, dass der Wert der `render`-Prop eine **Funktion** sein muss die ein valides **React-Element** zurückgibt \(hier sei auch nochmal an das entsprechende Kapitel zu **Render-Props** erinnert\), während die `component`-Prop eine **Komponente** \(kein _Element!_\) erwartet. 

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
import { BrowserRouter as Router, Route } from 'react-router-dom';

const Home = () => <p>Home Content</p>;
const Account = () => <p>AccountContent</p>;
import HomeSidebar = () => <p>Home Sidebar</p>
import AccountSidebar = () => <p>AccountSidebar</p>

const App = () => {
  return (
    <Router>
      <main>
        <Route path="/account" component={Account} />
        <Route path="/" component={Home} />
      </main>
      <aside>
        <Route path="/account" component={AccountSidebar} />
        <Route path="/" component={HomeSidebar} />
      </aside>
    </Router>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

In diesem Beispiel sehen wir wie an zwei verschiedenen Stellen in unserer Anwendung unterschiedliche Komponenten gerendert werden, je nachdem welche URL momentan aufgerufen wird.

Hier sind wir in der Struktur der Komponenten jedoch ziemlich frei. Und so ist nicht unüblich, dass eine solche Doppelung von Routen durch eine etwas andere Struktur vermieden wird. Das obige Beispiel würde bspw. dann so geschrieben werden:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router, Route } from 'react-router-dom';

const Home = () => (
  <>
   <main>Home Content</main>
   <aside>Home Sidebar</aside>
  </>
);

const Account = () => (
  <>
   <main>Account Content</main>
   <aside>Account Sidebar</aside>
  </>
);

const App = () => {
  return (
    <Router>
      <Route path="/account" component={Account} />
      <Route path="/" component={Home} />
    </Router>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

Hier haben wir nun das doppelte Routing vermieden, allerdings zu Ungunsten der Layout-Struktur, die nun dupliziert wurde. In einer typischen Anwendung könnten wir nun hergehen und die Struktur in eine eigene Layout-Komponente abstrahieren:

```jsx
import React from "react";

const Layout = props => (
  <>
    <main>{props.content}</main>
    <aside>{props.sidebar}</aside>
  </>
);

const Home = () => <Layout content="Home Content" sidebar="Home Sidebar" />;
const Account = () => <Layout content="Account Content" sidebar="Account Sidebar" />;

const App = () => {
  return (
    <Router>
      <Route path="/account" component={Account} />
      <Route path="/" component={Home} />
    </Router>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

Wer die Code-Beispiele jetzt ausprobiert wird hierbei ein Verhalten feststellen, das in den meisten Fällen ungewollt ist: **React Router** geht sehr locker und lässig mit den Pfaden beim Path-Matching um. Und so sehen wir beim Aufruf der `/account` URL nicht nur die `Account`-Komponente sondern ebenfalls die `Home`-Komponente. Dies passiert, weil `/account` **auch** den Pfad `/` **beinhaltet**, und somit werden beide Komponenten gerendert. Das ist durchaus so gewollt, wo wäre es z.B. möglich einzelne Seitenbereiche unter einem bestimmten URL Präfix zu gliedern und dabei eine Komponente auf jeder dieser Routen rendern zu lassen.

Denken wir an einen Benutzer-Account und stellen uns eine Sidebar vor. Vielleicht entwickeln wir gerade eine Community. Dort gibt es einen Benutzerbereich in verschiedene Unterkategorien unterteilt ist: `/account/edit`, um das eigene Profil zu bearbeiten, `/account/images`, um die eigenen Bilder anzusehen oder `/account/settings`, um Änderungen an den eigenen Einstellungen vorzunehmen. Wir können nun eine generische Route in unserer Anwendung einfügen:

```jsx
<Route path="/account" component={AccountSidebar} />
```

Die `AccountSidebar`-Komponente würde nun auf jeder Unterseite innerhalb des Account-Bereichs angezeigt, solange eben die URL mit `/account` beginnt.

### Matching einschränken via Prop

Um das Matching zwischen dem `path` und der URL bewusst einzuschränken bietet uns React Router die `exact` Prop auf der `Route`-Komponente. Wird diese Boolean-Prop angegeben, wird eine Route nur noch dann gerendert wenn ihre `path`-Prop auch exakt mit der aktuellen URL übereinstimmt:

```jsx
<Route exact path="/" component={Home} />
```

Die Reihenfolge wo genau die Prop angegeben wird ist dabei, wie immer in **JSX**, zu vernachlässigen, ich schreibe sie gern direkt vor die `path`-Prop um den Eindruck einer „sprechenden“ Prop zu vermitteln: hier habe ich eine **Route** mit dem **exact path**. In unserem Beispiel mit der Account-Sidebar würde die Sidebar bei Verwendung der `exact`-Prop nun nur noch dann gerendert wenn die URL `/account` entspricht, jedoch nicht mehr bei `/account/edit`, `/account/images` oder `/account/settings`.

### Matching auf eine Route limitieren via Switch-Komponente

Die `exact`-Prop bezieht sich dabei immer nur auf eine **einzelne Route** und lässt andere Routen davon gänzlich unberührt. Haben wir eine Reihe von URLs von denen in bestimmten Fällen mehrere Routen matchen können, wird das mitunter mühsam jeder einzelnen dieser Routen eine weitere Prop hinzuzufügen. Hier hilft uns der nächste Import aus dem **React Router** Paket weiter: `Switch`.

Mit der `Switch`-Komponente, die sich um eine Reihe von `<Route />`-Elementen legt, sorgen wir dafür, dass jeweils immer nur die **erste** Route deren `path` mit der aktuellen URL übereinstimmt gerendert wird. Oft ist es keine schlechte Idee Routen grundsätzlich in einem `Switch`-Element zu verpacken, außer man möchte eben explizit, dass mehrere Routen gerendert werden. Statt der Verwendung der `exact`-Prop wie im obigen Beispiel, wäre auch die Verwendung der `Switch`-Komponente möglich:

```jsx
<Router>
  <Switch>
    <Route path="/account" component={Account} />
    <Route path="/" component={Home} />
  </Switch>
</Router>
```

Beim Aufruf der `/account` URL würde nun gleich die erste Route zutreffen und alle folgenden Routen würden ignoriert, wir würden also nur die `Account`-Komponente rendern. Dabei prüft die `Switch`-Komponente auch stets nur ihre **direkten** Kind-Elemente auf ein Matching mit der URL. Enthält die `Account`-Komponente wiederum eigene Routen, was problemlos möglich ist, sind diese von der Switch-Komponente unbeeindruckt und werden gerendert wenn ihr `path` mit der aktuellen URL übereinstimmt.

Dies ermöglicht uns darüber hinaus die Erstellung einer 404 Fehlerseite als Fallback-Route. Lassen wir die `path`-Prop aus, bedeutet dies, dass diese Route auf **jede** URL zutrifft. Nutzen wir sie also innerhalb eines `Switch`-Elements als letzte Komponente sagen wir dem **React Router** damit: _rendere diese Komponente immer dann wenn keine andere Route zutrifft, und zwar nur dann!_ Und das sieht dann so aus:

```jsx
const Error404 = () => <h1>404 – Seite nicht gefunden</h1>;

const App = () => (
  <Router>
    <Switch>
      <Route path="/account" component={Account} />
      <Route path="/contacts" component={Contacts} />
      <Route path="/inbox" component={Inbox} />
      <Route exact path="/" component={Home} />
      <Route component={Error404} />
    </Switch>
  </Router>
);
```

In diesem Fall ist zusätzlich zum `Switch`-Element auch noch eine `exact`-Prop bei der `/` Route notwendig, da sonst diese immer zutreffen würde wenn vorher keine andere der Routen auf die aktuelle URL zutrifft, da der Router eben so funktioniert, dass bspw. auch `/existiert-nicht` unter der `/`-Route gefunden werden würde. Mit der `exact`-Prop auf der `Home`-Route passiert eben genau dies nicht und stattdessen sehen wir unsere `Error404`-Komponente die am Ende unseres `Switch`-Blocks alle Aufrufe abfängt. Vergleichbar mit dem `default`-Fall in einem `switch`-Statement in JavaScript.

### Parameter in URLs

Kaum eine Anwendung kommt ohne URLs aus, die Parameter enthält. Auch dieser Fall wird natürlich vom React Router abgedeckt und in einer Form die jedem der schon einmal mit anderen Routing-Mechanismen gearbeitet hat durchaus vertraut vorkommen dürfte, nämlich durch die Verwendung eines Parameter-Namens mit vorangestellten Doppelpunkt \(`:`\). 

```jsx
<Route path="/users/:userid" component={UserProfile} />
```

Dabei kann hier eingeschränkt werden was genau der Parameter als solchen erkennen soll. Möchte ich bspw. auf einer Route die Sortierung auf _aufsteigend_ \(asc\) oder _absteigend_ \(desc\) beschränken, kann ich dies durch einen regulären Ausdruck in Klammern, die ich unmittelbar an den Parameter anhänge:

```jsx
<Route path="/products/:order(asc|dec)"
```

Die obige Route würde dann nur zutreffen wenn die URL `/products/asc` oder `/products/desc` lautet.

Möchte ich im ersten Beispiel, dass nur numerische Werte als `:userid` erlaubt sind, so kann ich dafür die Route definieren: `/users/:userid(\d*)` oder `/users/:userid([0-9]*)` und somit würde die URL `/users/123` die `UserProfile`-Komponente rendern, `/users/abc` hingegen nicht.

Findet der **React Router** eine solche URL mit einem Parameter, extrahiert er dessen Wert und übergibt ihn in einer `match`-Prop an die gerenderte Komponente.

### Verwendung der Router Props

Jede Komponente die vom React Router gerendert wird weil sie als `component`-Prop einer `Route`-Komponente angegeben wurde bekommt automatisch drei für das Routing relevante Props übergeben:

* `match` 
* `location` 
* `history`

Auf diese kann innerhalb der jeweiligen Komponente wie auf jede andere Prop auch über `this.props` in **Klassen-Komponenten** oder `props` in **Function Components** zugegriffen werden:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router, Route } from "react-router-dom";

const Example = (props) => {
  console.log(props);
  return <p>Example</p>;
}

const App = () => (
  <Router>
    <Route path="/users/:userid" component={Example} />
  </Router>
);

ReactDOM.render(<App />, document.getElementById("root"));
```

Schauen wir uns die Konsolen-Ausgabe dieser Komponente an, sehen wir beim Aufruf der URL `/users/123` ein Resultat das diesem entspricht \(gekürzt\):

```javascript
{
  history: { /* ... */ },
  location: { /* ... */ },
  match: {
    path: "/:userid",
    url: "/users/123",
    isExact: true,
    params: {
      userid: "123"
    }
  }
}
```

Ohne bereits jetzt im Detail auf jede einzelne Eigenschaft einzugehen, bekommen wir hier einen Eindruck auf welche Eigenschaften des Routers wir hier zugreifen können. Für den Moment interessiert uns erst einmal die `match`-Eigenschaft. Diese enthält bei einer zutreffenden URL in `match.params` die Parameter, die wir im `path` definiert haben mitsamt ihrer Werte. In diesem Fall also `match.params.userid` mit dem Wert `123`.

In einer Benutzerprofil-Komponente könnten wir nun etwa hergehen und einen API-Request starten um uns alle relevanten Daten für den Benutzer mit der BenutzerID `123` zu besorgen und die Profilansicht dieses Benutzers darzustellen.

**React Router** stellt dabei sicher dass in jeder mit dem Router verbundenen Komponente die `match` Eigenschaft immer existiert und diese auch immer eine `params`-Eigenschaft hat, die entweder die Parameter enthält \(falls vorhanden\) oder aber ein leeres Objekt ist. Es ist daher **sicher** auf `props.match.params` zuzugreifen, ohne befürchten zu müssen, dass eine dieser Eigenschaften `undefined` ist und somit einen Fehler wirft.

### Navigation zwischen einzelnen Routen

Haben wir erst einmal eine Anwendung in mehrere Routen unterteilt, wollen wir natürlich auch zwischen diesen URLs verlinken. Dies ginge natürlich mit dem HTML `<a href="...">...</a>` Element. Keine Frage. Allerdings lösen wir damit einen komplett neuen „harten“ Seitenaufruf im Browser aus, verlassen die aktuelle Seite **komplett** und rufen die neue Seite komplett neu auf. 

Das heißt wir fordern ein HTML-Dokument an, das HTML-Dokument lädt CSS und das JavaScript mit unserer React-Anwendung erneut vom Server \(oder holt es idealerweise aus dem Browser-Cache\), initialisiert alles neu und entscheidet auf Basis der geänderten URL welche Route gerendert werden soll. Etwaiger State den wir zuvor global gesetzt haben würde dadurch zurückgesetzt. 

Das ist aber eben nicht das Verhalten das wir uns in einer Single Page Applikation wünschen. Hier möchten wir schließlich das HTML mitsamt seinem CSS und JavaScript nur einmal vom Server laden. Wir möchten den globalen State beim Navigieren zwischen den einzelnen Routen persistieren und nur die Teile der Seite neu rendern, die sich auch basierend auf der Route ändern sollen.

Hierzu bringt uns **React Router** die `Link`-Komponente mit. Diese wird wie alle anderen Komponenten aus dem `react-router-dom`-Paket importiert und besitzt eine `to`-Prop die man grob mit dem `href`-Attribut bei HTML `a`-Elementen vergleichen kann:

```jsx
<Link to="/account">Account</Link>
```

Intern nutzt **React Router** für die Darstellung dieser Links ebenfalls ein ganz gewöhnliches `<a href />`, jedoch werden Klicks auf diese Links abgefangen und an eine Router-interne Funktion weitergeleitet, die sich dann darum kümmert neue Seiteninhalte auf Basis der neuen URL darzustellen, ohne dabei einen vollständigen Pageload auszulösen.

Neben der `to`-Prop kann ein `Link` auch eine `innerRef` besitzen die eine über `createRef()` oder `useRef()` erzeugte Ref erhalten kann, sowie eine `replace`-Prop mit der wir festlegen können, dass wir die aktuelle URL in der Browser-History _ersetzen_ wollen statt einen neuen History-Eintrag zu erzeuge. Beim Klick auf den Zurück-Button im Browser kann dann nicht mehr auf die vorherige Route zurückgesprungen werden.

Alle weiteren Props die dem `<Link />`-Element übergeben werden werden an das erzeugte `a`-Element weitergereicht. So würde `<Link to="/" title="Homepage">Home</Link>` in der Ausgabe entsprechend das folgende Markup erzeugen: `<a href="/" title="Homepage">Home</a>`.

### Sonderausprägung: NavLink

Eine besondere Ausprägung des `Link`-Elements stellt der `NavLink` dar. Neben den Props die auch die `Link`-Komponente erhalten kann, kennt der `NavLink` darüber hinaus seinen aktuellen Zustand in dem Sinne, dass er weiß ob er gerade auf die aktuelle Seite verlinkt. Ist dies der Fall können ihm mittels `activeClassName` und `activeStyle` ein alternatives Erscheinungsbild verpasst werden.

Ein klassisches Beispiel ist hier eine Seitennavigation, bei der der jeweils aktuelle Menüpunkt farblich hervorgehoben wird.

```jsx
<NavLink to="/" activeClassName="active">Home</NavLink>
<NavLink to="/account" activeClassName="active">Account</NavLink>
<NavLink to="/contacts" activeClassName="active">Kontakte</NavLink>
```

Hier würde der Link der jeweils aktuellen Seite, und nur dieser, die Klasse `active` erhalten. Befinden wir uns bspw. auf der `/account` URL, wäre das Markup dementsprechend:

```markup
<a href="/">Home</a>
<a href="/account" class="active">Account</a>
<a href="/contacts">Kontakte</a>
```

Weitere **Props** die ein `NavLink`-Element enthalten kann sind `exact` und `strict`, analog zu den entsprechenden **Props** bei der `Route`-Komponente, sowie `isActive`. Letztere erwartet eine Funktion, die entweder `true` \(aktive Seite entspricht dem `NavLink`\) oder `false` \(aktive Seite entspricht ihm nicht\) zurückgibt. Die Funktion selbst bekommt vom Router als erstes Argument das `match` Objekt und als zweites Argument das `location` Objekt übergeben. Die Funktion kann anhand dieser Information nun ermitteln ob sie den jeweiligen `NavLink` als aktiv markiert oder nicht.

### Programmatisch Navigieren mit der History API

Nun wissen wir, wie wir durch `Route`-Elemente auf verschiedene URLs mit dem Rendering entsprechender Komponenten reagieren können und wir wissen, wie wir einen harten Pageload durch die Verwendung des `<Link />`-Elements verzichten. Doch in manchen Fällen ist es notwendig einen Wechsel der URL programmatisch zu forcieren. Etwa um den Benutzer auf eine andere Seite weiter zu leiten nachdem ein asynchroner Request erfolgreich war.

Zu diesem Zweck dient die `history` Eigenschaft, die der Router allen als Route verwendeten Komponenten in den `Props` übergibt:

```javascript
{
  history: {
    action: "POP"
    block: Function(prompt),
    go: Function(number),
    goBack: Function(),
    goForward: Function(),
    length: 1
    push: Function(path, state)
    replace: Function(path, state)
  },
  location: { /* ... */ },
  match: { /* ... */ }
}
```

In unserem besonderen Interesse liegen hier die beiden Funktionen `push()` sowie `replace()`. Mittels `props.history.push('/ziel')` können wir die URL im Browser auf `/ziel` ändern, sowie ein Rerendering auslösen. Dabei wird wie auch bei der Verwendung der `Link`-Komponente ein neuer Eintrag in der Browser-History erzeugt. Möchten wir hingegen keinen neuen Eintrag in der Browser-History erzeugen, können wir stattdessen die `props.history.replace('/ziel')` Funktion verwenden.

Die Funktion `props.history.go()` ermöglicht es uns in der Browser-History programmatisch vor- und zurück zu blättern. Die Funktion erwartet dabei einen Integer als Parameter der angibt um wie viele Schritte in der History geblättert werden soll. Negative Werte blättern dabei zurück, während positive Werte vorwärts blättern. Die beiden Funktionen `goBack()` und `goForward()` sind dabei Shortcuts, die jeweils einen Schritt in der History zurück- bzw. vorblättern. Sie entsprechen also `go(-1)` bzw. `go(1)`.

Die `action` Eigenschaft verrät uns durch welche Aktion ein Benutzer auf der aktuellen Route gelandet ist. Sie ist entweder `POP`, `PUSH` oder `REPLACE`, wobei `POP` sowohl bedeuten kann, dass es sich um den initialen Seitenaufruf handelt oder der Benutzer den Zurück-Button im Browser gedrückt hat, `PUSH` bedeutet, dass die `history.push()` Funktion aufgerufen wurde, was auch beim Klick auf einen `<Link />` der Fall ist. Ist der Wert der `action`-Eigenschaft `REPLACE` - ihr könnt es euch denken - wurde `history.replace()` aufgerufen oder ein `<Link />`-Element geklickt, dass eine `replace`-Prop besitzt.

### Komponenten mit dem Router über die HOC verbinden



