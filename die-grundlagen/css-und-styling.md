# CSS und Styling

Styling in React ist ein relativ eigenes Thema. So bietet React keine Hausmittel an, um das Styling von Anwendungen mittels CSS zu erleichtern, mit CSS-in-JS hat sich jedoch eine ganz eigene, teilweise sehr kontrovers diskutierte Bewegung zu der Thematik gebildet. Dabei wird der Styling-Teil ebenfalls in JavaScript umgesetzt um nicht mit dem Paradigma der komponentenbasierten Entwicklung zu brechen. Doch eins nach dem anderen, starten wir erst einmal mit den Basics und arbeiten uns dann Stück für Stück an das Thema heran.

### Styling mittels style-Attribut

Die wahrscheinlich einfachste Möglichkeit um Komponenten zu stylen in React ist durch die Verwendung des `style`-Attributs auf HTML-Elementen. Dieses funktioniert jedoch etwas anders als in HTML und so erwartet React ein **Objekt** in der Form `Eigenschaft: Wert`, wobei als Eigenschaft die JavaScript-Schreibweise einer CSS-Eigenschaft erwartet wird. Also etwa `zIndex` statt `z-index`, `backgroundColor` statt `background-color` oder `marginTop` statt `margin-top`. Bei Werten die Pixel-Angaben akzeptieren ist die Verwendung von `px` als Einheit optional:

```jsx
<div style={{border: '1px solid #ccc', marginBottom: 10}}>
  Ein div mit einem grauen Rahmen und 10 Pixel Abstand nach unten
</div>
```

Werte die ohne Einheit verwendet werden \(bspw. `z-index`, `flex` oder `fontWeight`\) sind davon nicht betroffen, sie können bedenkenlos benutzt werden, ohne dass React sie um `px` ergänzt. 

Durch die Verwendung eines Objekts anstelle eines Strings ist React konsistent zur `style`-Eigenschaft von DOM-Elementen \(`document.getElementById('root').style` ist ebenfalls ein Objekt!\) und sorgt außerdem dafür, dass keine Sicherheitslücken durch XSS entstehen können.

Nun ist die Verwendung von Inline-Styles nicht unbedingt wünschenswert, sie kann aber in manchen Situationen hilfreich sein, etwa wenn das Styling eines Elements von bestimmten variablen Werten im State abhängt.

### Verwendung von CSS-Klassen in JSX

Wesentlich angenehmer und sauberer ist die Verwendung von echten CSS-Klassen in JSX, wie es eben auch schon aus der Verwendung on HTML bekannt ist. Der entscheidende Unterschied hier: anders als in HTML lautet der Name der entsprechenden Prop in JSX nicht `class`, sondern `className`:

```jsx
<div className="item">...</div>
```

React rendert hier schließlich gewohnte Syntax mit der jeweiligen HTML-Entsprechung:

```markup
<div class="item">...</div>
```

Wie in JSX üblich kann der Wert für die `className`-Prop auch dynamisch zusammengesetzt werden:

```jsx
render() {
  let className = 'item';
  if (this.state.selectedItem === this.props.itemId) {
    className += ' item--selected';
  }
  return (
    <div className={className}>
      ...
    </div>
  );
}
```

Hier wäre der Wert für `className` in jedem Fall `item` und für den Fall, dass das ausgewählte Item dem aktuellen Item entspricht `item item--selected`.

Als de facto Standard für derartige Fälle hat sich das Paket `classnames` etabliert. Mit diesem ist es möglich Klassen abhängig von einer Bedingung zu machen. Installiert werden kann es über die CLI, mittels:

```bash
npm install classnames
```

```bash
yarn add classnames
```

Anschließend müssen wir es nur noch in den Komponenten importieren, in denen wir es nutzen wollen. Dies passiert ganz unkompliziert via:

```javascript
import classNames from 'classnames';
```

Damit importieren die Funktion und weisen ihr den Namen `classNames` zu. Die Funktion erwartet dann beliebig viele Parameter die entweder ein String sein können oder ein Objekt in der Form `{ Klasse: true|false }`. Die Funktionsweise ist dabei ebenfalls recht simpel: ist der Wert für eine Eigenschaft `true`, setzt `classNames` eine Klasse mit dem selben Eigenschaftsnamen. Im obigen Beispiel wäre das also:

```jsx
render() {
  return (
    <div className={classNames('item', {
      'item--selected': this.state.selectedId === this.props.itemId
    })}>
      ...
    </div>
  );
}
```

Auch die Verwendung eines Objekts mit mehreren Eigenschaften ist möglich, hier werden dann sämtliche Eigenschaften deren Wert `true` ist als Klasse gesetzt:

```jsx
render() {
  return (
    <div className={classNames({
      'item': true,
      'item--selected': this.state.selectedId === this.props.itemId
    })}>
      ...
    </div>
  );
}
```

Bei der Verwendung von Klassen sollte natürlich sichergestellt sein, dass das Stylesheet mit den jeweiligen Klassen auch im HTML-Dokument entsprechend eingebunden wird, da sich React darum prinzipiell nicht kümmert.

### Modulares CSS mit CSS Modules

**CSS Modules** sind eine Art Vorstufe zu **CSS-in-JS** und vereinen einige Eigenschaften von CSS und JavaScript-Modulen in sich. Wie der Name es suggeriert befindet sich das CSS hier in eigenen importierbaren **Modulen**, die jedoch aus reinem CSS bestehen und zumeist unmittelbar zu einer Komponente gehören. Entwickeln wir etwa eine Komponente um ein Profilbild darzustellen in `ProfileImage.js`, existiert beim **CSS Modules** Ansatz oft auch eine `ProfileImage.module.css`  dazu. Beim Import eines solchen CSS Moduls wird dann sichergestellt, dass ein einmaliger, oftmals etwas kryptischer Klassenname generiert wird, um so sicherzustellen, dass diese Klasse nur in der jeweiligen Komponente verwendet wird. Konflikte mit anderen Komponenten die gleiche Klassennamen verwenden sollen so ausgeschlossen werden.

Dabei sind **CSS Modules** selbst sind jedoch erstmal nur ein **Konzept** und noch keine konkrete Implementierung. Die Implementierung übernehmen dann Tools wie bspw. `css-loader` für Webpack.

Die Endung `.module.css` ist dabei keineswegs zwingend und auch ein bloßes `.css` wäre möglich, `.module.css` hat sich aber bei einigen bekannten Tools mit großer Reichweite als Konvention entwickelt und wird auch von **Create React App** ohne die Notwendigkeit weitere Konfigurationseinstellungen vorzunehmen unterstützt. CRA greift dabei auf die eben angesprochene `css-loader` Implementierung für Webpack zurück.

In den CSS-Dateien befindet sich reguläres, **standardkonformes CSS** in seiner gewohnten Schreibweise. Dieses kann dann in einer Komponente wie ein JavaScript-Modul importiert werden:

```javascript
import css from './ProfileImage.module.css';
```

Hier kommt dann die Besonderheit von **CSS Modules** zum Tragen: in der Variable `css` befindet sich nun ein Objekt mit Eigenschaften, die den Klassennamen aus unserem CSS Modul entsprechen. Existiert in der CSS-Datei eine CSS-Klasse mit dem Namen `image`  haben wir nun im `css`-Objekt ebenfalls eine Eigenschaft mit dem Namen `image`. Dabei ist der Wert ein einmaliger, generierter String wie z.B. `ProfileImage_image_2cvf73`. 

Diesen generierten Klassennamen können wir nun in unserer Komponente als `className` vergeben:

```jsx
<img src={props.imageUrl} className={css.image} />
```

Und das Resultat beim Rendering wäre:

```markup
<img src="..." className="ProfileImage_image_2cvf73" />
```

Nutzen wir nun die Klasse `image` noch in einer anderen Komponente und importieren dort ebenfalls eine CSS-Datei mit einer `image`-Klasse würde es hier, anders als in gewöhnlichem CSS, **nicht** zu einem Konflikt kommen, da der _generierte_ Klassenname ein anderer wäre. 

Dabei ist alles erlaubt was auch in CSS erlaubt ist und auch die Kaskade \(also das **C** in **Cascading Style Sheets**\) bleibt dabei intakt:

```css
.imageWrapper img {
  border: 1px solid red;
}

.imageWrapper:hover img {
  border-color: green;
}
```

Das obige CSS würde bei folgender JSX-Struktur auf das Bild angewendet werden:

```jsx
const ProfileImage = () => {
  return (
    <div className={css.imageWrapper}>
      <img src="..." />
    </div>
  );
};
```

Der Hintergrund für **CSS Modules** ist eben der, dass mögliche Konflikte bei der Entwicklung von sehr isolierten Komponenten eben ausgeschlossen werden sollen, während das eigentliche Styling selbst weiterhin in CSS-Dateien und ganz ohne JavaScript-Kenntnisse vorgenommen werden kann. Ein idealer Kompromiss insbesondere in Teams, in denen die Entwicklung von JavaScript und CSS bislang recht strikt getrennt wurde und es Experten für beide Disziplinen im Team gibt.

### CSS-in-JS – die Verlagerung von Styles ins JavaScript

In der Einleitung hatte ich es bereits angesprochen: **CSS-in-JS** wird in einigen Entwicklerkreisen mitunter sehr rege und kontrovers diskutiert. Gegner führen das Argument ins Feld, dass die Befürworter von **CSS-in-JS** einfach nicht verstanden hätten die Kaskade zu verwenden, um skalierbares CSS zu schreiben, während Befürworter meist dagegen argumentieren dass es nicht um die Kaskade geht, sondern man ganz einfach einen sicheren Weg benötige, um höchstmögliche Isolation von Komponenten sicherzustellen. Nun, ich persönlich bin da eher diplomatisch und denke, dass es für beide Seiten Argumente gibt, doch vor allem glaube ich, dass **CSS-in-JS** durchaus seine Daseinsberechtigung hat. Doch worum geht es überhaupt genau?

Beim **CSS-in-JS**-Ansatz wird der Styling-Teil, der in der klassischen Web-Entwicklung immer die Aufgabe von CSS war, in den JavaScript-Teil verschoben. Wie auch bei der **CSS Modules** „Hybrid“-Lösung steht hier vor allem die Isolation und konfliktfreie Wiederverwendbarkeit von JavaScript-Komponenten im Vordergrund. Anders als bei **CSS Modules** findet das Styling in **CSS-in-JS** jedoch vollständig in JavaScript-Dateien statt, allerdings noch immer unter Verwendung von CSS oder zumindest CSS-naher Syntax. 

**CSS-in-JS** ist dabei ebenfalls nur ein Oberbegriff für das Konzept als solches, die konkrete Implementierung findet dann durch die Verwendung von einer von mittlerweile sehr vielen Libraries statt. Eine Übersicht über die zur Auswahl stehenden Optionen bietet diese Liste: [http://michelebertoli.github.io/css-in-js/](https://michelebertoli.github.io/css-in-js/) – hier werden insgesamt über 60 verschiedene Libraries gelistet, die als **CSS-in-JS** Implementierung benutzt werden können.

In diesem Kapitel wollen wir uns auf **Styled Components** als kurzes Beispiel beschränken, das mit über 23.000 Stars auf GitHub die populärste Option darstellt. Dazu müssen wir **Styled Components** zunächst  als Abhängigkeit in unser Projekt über die Kommandozeile installieren:

```bash
npm install styled-components
```

```bash
yarn add styled-components
```

Ist das Paket erst einmal installiert, können wir es importieren und dann gleich mit einer ersten **Styled Component** beginnen. Dazu importieren wir die `styled`-Funktion und nutzen das jeweilige HTML-Element, das wir stylen wollen, als Eigenschaft. Anschließend nutzen wir die **Template Literal** Syntax aus ES2015 um das CSS für unsere **Styled Component** zu definieren. 

Um einen schwarz-gelben Button zu erzeugen, sähe das in einem vollständigen Beispiel etwa so aus:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import styled from 'styled-components';

const Button = styled.button`
  background: yellow;
  border: 2px solid black;
  color: black;
  padding: 8px;
`;

const App = () => {
  return (
    <div>
      <h1>Beispiel für eine Styled Component</h1>
      <Button>Ein schwarz-gelber Button ohne Funktion</Button>
    </div>
  );
};

ReactDOM.render(<App />, document.getElementById('root'));
```

Durch die Verwendung von `const Button = styled.button` erzeugt **Styled Components** dabei eine neue `Button`-Komponente und weist der Komponente die CSS-Eigenschaften zu, die wir im darauf folgenden **Template Literal** definieren. Innerhalb dieses **Template Literals** wird dann wiederum herkömmliches CSS verwendet. 

Möchten wir Pseudo-Selektoren oder -Elemente verwenden, nutzen wir diese einfach ohne vorangestellten Selektor. Im obigen Beispiel sieht das für einen `:hover`-Status etwa so aus:

```jsx
const Button = styled.button`
  background: yellow;
  border: 2px solid black;
  color: black;
  padding: 8px;
  :hover {
    background: gold;
  }
`;
```

Auch der Zugriff auf die **Props** einer **Styled Component** ist möglich. Dazu wird im Template String eine Funktion aufgerufen, die als ersten Parameter alle **Props** des jeweiligen Elements übergeben bekommt:

```jsx
const Button = styled.button`
  background: yellow;
  border: 2px solid black;
  color: black;
  cursor: ${(props) => props.disabled ? 'not-allowed' : 'pointer'};
  padding: 8px;
  :hover {
    background: gold;
  }
`;
```

Hier würde etwa der Mauszeiger zu einem `not-allowed` Symbol werden wenn der Button eine `disabled`-Eigenschaft besitzt. Darüber hinaus bietet Styled Components auch Unterstützung für Themes, serverseitiges Rendering, CSS Animationen und vieles mehr. Für eine \(sehr\) detaillierte Übersicht empfehle ich an dieser Stelle einen Blick auf die [gute und umfassende Dokumentation](https://www.styled-components.com/docs/basics) zu werfen.

Die positiven Seiten von **CSS-inJS** generell und **Styled Components** im Speziellen liegen hier auf der Hand und die Dokumentation fasst sie ziemlich gut zusammen:

* kritisches CSS, also das, was für die aktuell aufgerufene Seite relevant ist wird automatisch generiert, da **Styled Components** genau weiß, welche Komponenten auf einer Seite verwendet werden und welche Styles diese benötigen
* durch die automatische Generierung von Klassennamen wird das Risiko von Konflikten auf ein Minimum reduziert
* dadurch, dass CSS unmittelbar an eine Komponente gebunden wird, ist es sehr einfach nicht mehr benutztes CSS zu finden. Wird die erzeugte **Styled Component** nicht mehr in einer Anwendung verwendet, wird auch das CSS nicht mehr benötigt und die Komponente kann mitsamt ihres CSS sicher gelöscht werden
* Komponenten-Logik \(JavaScript\) und Komponenten-Styling \(CSS-in-JS\) befindet sich unmittelbar am selben Ort, teilweise sogar in der gleichen Datei. Der Entwickler muss nicht erst mühsam die Stelle finden, die er ändern muss bei einer Style-Änderung
* **Styled Components** erzeugt ohne jede weitere Konfiguration automatisch CSS, das Vendor-Prefixes für alle Browser enthält

Nun ist **Styled Components** nur eine von wie bereits angesprochen sehr vielen Implementierungen des Konzepts **CSS-in-JS**. Sie ist ohne Zweifel die bekannteste und am weitesten adaptierte und auch für mich hat sie ihren Job tadellos erfüllt, in einigen meiner vergangenen Projekte. Dennoch schlage ich vor, vielleicht einen Blick auf andere, verfügbare Alternativen zu werfen um die für den jeweiligen eigenen Verwendungszweck optimale Lösung zu finden. Zu den ebenfalls bekannteren, erwähnenswerten Beispielen gehören:

* [emotion](https://github.com/emotion-js/emotion)
* [styled-jsx](https://github.com/zeit/styled-jsx)
* [react-jss](https://github.com/cssinjs/jss/tree/master/packages/react-jss)
* r[adium](https://github.com/FormidableLabs/radium)
* l[inaria](https://github.com/callstack/linaria)

