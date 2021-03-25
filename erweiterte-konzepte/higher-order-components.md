# Higher Order Components

**Higher Order Components** \(meist abgekürzt: **HOC** oder **HOCs**\) waren und sind ein sehr zentrales Konzept bei der Arbeit mit React. Sie erlauben es, Komponenten mit wiederverwendbarer Logik zu implementieren und sind angelehnt an **Higher Order Functions** aus der funktionalen Programmierung. Das Prinzip hinter derartigen Funktionen ist, dass sie eine Funktion als Parameter entgegennehmen und eine neue Funktion zurückgeben. Im Fall von React wird das Prinzip auf Komponenten angewandt. Daher der von den **Higher Order** _**Functions**_ abgeleitete Name **Higher Order** _**Component**_.

Zum leichteren Verständnis gleich ein erstes einfaches Beispiel:

```jsx
const withFormatting = (WrappedComponent) => {
  return class extends React.Component {
    bold = (string) => {
      return <strong>{string}</strong>;
    };
    italic = (string) => {
      return <em>{string}</em>;
    };
    render() {
      return <WrappedComponent bold={this.bold} italic={this.italic} />;
    }
  };
};
```

Hier haben wir eine Funktion `withFormatting` definiert, die eine React-Komponente entgegen nimmt. Die Funktion gibt dabei eine neue React-Komponente zurück, welche die _in die Funktion herein gegebene Komponente_ rendert und ihr dabei die Props `bold` und `italic` übergibt. Die hereingegebene Komponente kann nun auf diese Props zugreifen:

```jsx
const FormattedComponent = withFormatting(({ bold, italic }) => (
  <div>
    Dieser Text ist {bold('fett')} und {italic('kursiv')}.
  </div>
));
```

Typischerweise werden **Higher Order Components** benutzt um Logik darin zu kapseln. In diesem Zusammenhang ist auch oft die Rede von **Smart** und **Dumb Components** also: **schlaue** und **dumme** Komponenten. Die Smart Components \(zu denen Higher Order Components zählen\) sind dann dazu da, Business Logik abzubilden, mit APIs zu kommunizieren oder Verhaltenslogik zu verarbeiten. _Dumme_ Komponenten hingegen bekommen weitestgehend statische Props übergeben und beschränken den Logik-Teil auf reine Darstellungslogik. Also bspw. ob ein Benutzerbild, oder falls dieses nicht vorhanden ist, stattdessen ein Platzhalterbild angezeigt wird. In diesem Zusammenhang fällt auch oft der Begriff **Container Component** \(für _Smart_ Components\) und **Layout Components** \(für _Dumb_ Components\).

Doch wozu das überhaupt? Eine solche strikte Unterteilung in Business Logik und Darstellungslogik macht echte komponentenbasierte Entwicklung erst einmal möglich. Sie erlaubt es, Layout-Komponenten zu erstellen die keinerlei Kenntnis von etwaigen APIs haben und nur stumpf die Daten darstellen, die ihnen übergeben werden, völlig egal woher diese kommen. Gleichzeitig erlaubt sie es auch den Business Logik Komponenten, sich um die reine Business Logik zu kümmern, völlig gleichgültig, wie die Daten letzten Endes dargestellt werden.

Stellen wir uns ein gängiges Beispiel aus der Interface Entwicklung einmal vor: die Umschaltung zwischen einer **Listen-** und einer **Karten-Ansicht**. Hier würde sich eine **Container-Komponente** darum kümmern, die Daten zu beschaffen die relevant für den Benutzer sind. Sie würde die beschafften Daten dann an die frei konfigurierbare **Layout-Komponente** übergeben. Solange beide Komponenten sich an das vom Entwickler vorgebene Interface \(Stichwort **PropTypes**\) halten, sind beide Komponenten beliebig austauschbar und können vollkommen unabhängig voneinander entwickelt und **getestet** werden!

Genug Theorie. Zeit für ein weiteres Beispiel. Wir wollen uns eine Liste mit den 10 größten Kryptowährungen laden und ihren momentanen Preis anzeigen. Dazu erstellen wir eine **Higher Order Component,** die sich diese Daten über die frei zugängliche Coingecko API beschafft und an eine Layout-Komponente übergibt.

```jsx
const withCryptoPrices = (WrappedComponent) => {
  return class extends React.Component {
    state = {
      isLoading: true,
      items: [],
    };

    componentDidMount() {
      this.loadData();
    }

    loadData = async () => {
      this.setState(() => ({
        isLoading: true,
      }));

      try {
        const cryptoTicker = await fetch(
          'https://api.coingecko.com' +
            '/api/v3/coins/markets?vs_currency=eur&per_page=10'
        );
        const cryptoTickerResponse = await cryptoTicker.json();

        this.setState(() => ({
          isLoading: false,
          items: cryptoTickerResponse,
        }));
      } catch (err) {
        this.setState(() => ({
          isLoading: false,
        }));
      }
    };

    render() {
      const { isLoading, items } = this.state;
      return (
        <WrappedComponent
          isLoading={isLoading}
          items={items}
          loadData={this.loadData}
        />
      );
    }
  };
};
```

Voilà, fertig ist die **HOC** für die Abfrage der Crypto-Preise auf coingecko.com. Doch die **Higher Order Component** allein reicht noch nicht. Wir benötigen nun auch noch eine Layout-Komponente, an die wir die Verantwortung delegieren, die Daten entsprechend anzuzeigen.

Hierzu erstellen wir eine möglichst generische `PriceTable`-Komponente, die selbst keinerlei Kenntnis davon hat, ob sie nun die aktuellen Joghurtpreise aus dem örtlichen Supermarkt darstellt oder Preise von Kryptowährungen auf irgendeiner beliebigen Börse. Entsprechend nennen wir sie auch sehr generisch `PriceTable`:

```jsx
const PriceTable = ({ isLoading, items, loadData }) => {
  if (isLoading) {
    return <p>Preise werden geladen. Bitte warten.</p>;
  }

  if (!items || items.length === 0) {
    return (
      <p>
        Keine Daten vorhanden.{' '}
        <button onClick={loadData}>Erneut versuchen</button>
      </p>
    );
  }

  return (
    <table>
      {items.map((item) => (
        <tr key={item.id}>
          <td>
            {item.name} ({item.symbol})
          </td>
          <td>EUR {item.current_price}</td>
        </tr>
      ))}
      <tr>
        <td colSpan="2">
          <button onClick={loadData}>Neu laden</button>
        </td>
      </tr>
    </table>
  );
};
```

Die Komponente kennt drei Props: `isLoading`, um ihr mitzuteilen, dass die Daten, die sie einmal darstellen soll, aktuell noch geladen werden; `items`, was ein Array aus „Artikeln“ mit Preisen repräsentiert und `loadData`, eine Funktion, die erneut einen API-Request startet um die neuen Daten zu beziehen.

Beide Komponenten funktionieren vollkommen unabhängig voneinander. Die `PriceTable` kann nicht nur Crypto-Preise anzeigen, die `withCryptoPrices`-Komponente muss ihre Daten nicht zwangsweise in einer `PriceTable` darstellen. Wir haben hier also zwei vollständig gekapselte und wiederverwendbare Komponenten erstellt!

Doch wie bringen wir die beiden nun zusammen? Ganz einfach indem wir die `PriceTable`-Komponente als Parameter an die `withCryptoPrices`-Komponente übergeben. Aha! Und das sieht wie folgt aus:

```jsx
const CryptoPriceTable = withCryptoPrices(PriceTable);
```

Rendern wir nun eine Instanz der `CryptoPriceTable`, stößt die **Higher Order Component** beim `componentDidMount()` einen API-Request an und übergibt das Ergebnis dieses Requests an die ihr übergebene `PriceTable`-Komponente. Diese kümmert sich anschließend nur noch um die entsprechende Darstellung.

```jsx
ReactDOM.render(<CryptoPriceTable />, document.getElementById('root'));
```

Dadurch ergeben sich großartige Möglichkeiten für uns. Erst einmal sind beide Komponenten unabhängig voneinander testbar. Mehr dazu gibt es im entsprechenden Kapitel, wo wir uns nochmal gezielt anschauen, wie einfach man insbesondere Layout-Komponenten mittels Snapshot-Tests testen kann.

Weiter haben wir nun eben die Möglichkeit, auch andere Layout-Komponenten mit der `withCryptoPrices`-HOC zu „verbinden“. Um dieses mächtige Konzept einmal anhand eines Beispiels zu verdeutlichen, geben wir die Preise nun im CSV-Format aus. Unsere HOC bleibt dabei völlig unverändert. Unsere Layout-Komponente könnte wie folgt implementiert werden:

```jsx
const PriceCSV = ({ isLoading, items, loadData, separator = ';' }) => {
  if (isLoading) {
    return <p>Preise werden geladen. Bitte warten.</p>;
  }

  if (!items || items.length === 0) {
    return (
      <p>
        Keine Daten vorhanden.{' '}
        <button onClick={loadData}>Erneut versuchen</button>
      </p>
    );
  }

  return (
    <pre>
      {items.map(
        ({ name, symbol, current_price }) =>
          `${name}${separator}${symbol}${separator}${current_price}\n`
      )}
    </pre>
  );
};
```

Und damit haben wir auch schon unsere CSV-Layout-Komponente implementiert. Wieder schauen wir zuerst ob noch Daten geladen werden; anschließend schauen wir erneut, ob `items` vorhanden sind. Hier könnte man anfangen darüber nachzudenken, auch dies in einer weiteren HOC zu bündeln, denn HOCs lassen sich beliebig ineinander schachteln, sind es doch am Ende lediglich Funktionen, die als Parameter an andere Funktionen weitergegeben werden.

Zuletzt rendern wir den tatsächlichen Output: wir iterieren durch die Liste der `items`, picken uns über die **Object Destructuring** Syntax die für uns relevanten Eigenschaften `name`, `symbol` und `current_price` heraus und umschließen die einzelnen Zeilen mit einem `pre`-Element um den Zeilenumbruch am Ende der Zeile korrekt darzustellen.

Anders als bei der `PriceTable` haben wir hier allerdings noch eine weitere \(optionale\) Prop eingeführt: `separator` - um der Render-Komponente mitzuteilen welches Trennzeichen sie bei der Darstellung der Daten verwenden soll. Die Prop kann wie in JSX üblich bei der Verwendung der Komponente als simple Prop angegeben werden:

```jsx
const CryptoCSV = withCryptoPrices(PriceCSV);

ReactDOM.render(<CryptoCSV separator="," />, document.getElementById('root'));
```

Allerdings wird dafür eine Änderung an unserer ursprünglichen `withCryptoPrices`-HOC notwendig. Momentan werden lediglich die fest definierten Props `isLoading`, `items` und `loadData` an die Kind-Komponente \(`WrappedComponent`\) übergeben:

```jsx
return (
  <WrappedComponent
    isLoading={isLoading}
    items={items}
    loadData={this.loadData}
  />
);
```

Damit die in `<CryptoCSV separator="," />` angegebene `separator`-Prop auch korrekt an die `PriceCSV`-Komponente übergeben wird, müssen wir unserer **HOC** mitteilen, dass sie auch alle weiteren Props an die `WrappedComponent` übergibt. Je nach Einsatzzweck können weitere erlaubte Props entweder explizit übergeben werden, oder aber es werden einfach **sämtliche** weiteren Props übergeben:

```jsx
return (
  <WrappedComponent
    {...this.props}
    isLoading={isLoading}
    items={items}
    loadData={this.loadData}
  />
);
```

Entscheidend ist hier Zeile 3: mittels `{...this.props}`, die Spread-Syntax aus ES2015+, leiten wir hier sämtliche Props an die Kind-Komponente weiter.

{% hint style="info" %}
**Higher Order Components** sind ein schönes Mittel, um Logik zu „zentralisieren“ und seine Anwendung somit übersichtlicher zu strukturieren. Logik kann dabei sehr unkompliziert aus den Komponenten entfernt werden, die nur für das Rendering, also die Darstellungslogik sorgen sollten. Obwohl sie ein sehr zentrales Konzept in React waren und noch immer sind, sind sie gleichzeitig ein sehr altes Konzept.

Zwar werden **Higher Order Components** noch immer häufig verwendet und gegen ihre Verwendung ist nichts einzuwenden. Allerdings gibt es mittlerweile neuere Konzepte und seit den neuesten Updates vor allem neue Wege um eine ähnliche Funktionalität in vielen Fällen in noch übersichtlicher Form zu erreichen. Dazu gehören **Functions as a Child**, die neue **Context API**, die in Version 16.3.0 ihren Weg in React gefunden hat, sowie **Hooks**, die seit Version 16.8.0 in Function Components verwendet werden können. Diese Möglichkeiten werden in den folgenden Kapiteln beschrieben.
{% endhint %}
