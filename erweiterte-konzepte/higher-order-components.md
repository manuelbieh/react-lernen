# Higher Order Components

**Higher Order Components** \(meist abgekürzt: **HOC** oder **HOCs**\) waren und sind ein sehr zentrales Konzept bei der Arbeit mit React. Sie erlauben es Komponenten mit wiederverwendbarer Logik zu implementieren und sind angelehnt an **Higher Order Functions** aus der funktionalen Programmierung. Das Prinzip hinter derartigen Funktionen ist, dass sie eine Funktion als Parameter entgegennehmen und eine neue Funktion zurückgeben. Im Fall von React wird das Prinzip auf Komponenten angewandt. Daher der von den **Higher Order** _**Functions**_ abgeleitete Name **Higher Order** _**Component**_.

Zum leichteren Verständnis gleich ein erstes einfaches Beispiel:

```jsx
const withFormatting = (WrappedComponent) => {
    return class extends React.Component {
        bold = (string) => {
            return <strong>{string}</strong>
        }
        italic = (string) => {
            return <em>{string}</em>
        }
        render() {
            return <WrappedComponent bold={this.bold} italic={this.italic} />
        }
    }
};
```

Hier haben wir eine Funktion `withFormatting` definiert, die eine React-Komponente entgegen nimmt. Die Funktion gibt dabei eine neue React-Komponente zurück welche die in die Funktion herein gegebene Komponente rendert und ihr dabei die Props `bold` und `italic` übergibt. Die hereingegebene Komponente kann nun auf diese Props zugreifen:

```jsx
const FormattedComponent = withFormatting(({ bold, italic }) => (
  <div>
    Dieser Text ist {bold('fett')} und {italic('kursiv')}.
  </div>
));
```

Typischerweise werden **Higher Order Components** benutzt um Logik darin zu kapseln. In diesem Zusammenhang ist auch oft die Rede von **Smart** und **Dumb Components** also schlaue und dumme Komponenten. Die Smart Components \(zu denen Higher Order Components zählen\) sind dann dazu da um Business Logik abzubilden, mit APIs zu kommunizieren oder Verhaltenslogik zu verarbeiten. Dumme Komponenten hingegen bekommen weitestgehend statische Props übergeben und beschränken den Logik-Teil auf reine Darstellungslogik. Also bspw. ob ein Benutzerbild oder, falls dieses nicht vorhanden ist, stattdessen ein Platzhalterbild angezeigt wird. In diesem Zusammenhang fällt auch oft der Begriff **Container Component** \(für _Smart_ Components\) und **Layout Components** \(für _Dumb_ Components\).

Doch wozu das ganze überhaupt? Eine solche strikte Unterteilung in Business Logik und Darstellungslogik macht wirklich Komponenten getriebene Entwicklung erst einmal möglich. Sie erlaubt es Layout Komponenten zu erstellen die keinerlei Kenntnis von etwaigen APIs haben und nur stumpf die Daten darstellen die ihnen übergeben werden, völlig egal woher diese kommen. Gleichzeitig erlaubt sie es auch den Business Logik Komponenten sich um die reine Business Logik zu kümmern, völlig gleichgültig wie die Daten letzten Endes dargestellt werden.

Stellen wir uns ein gängiges Beispiel aus der Interface Entwicklung einmal vor: die Umschaltung zwischen einer Listen und einer Karten-Ansicht. Hier würde sich eine **Container-Komponente** darum kümmern die Daten zu beschaffen die relevant sind für den Benutzer. Sie würde die beschafften Daten dann an die frei konfigurierbare **Layout-Komponente** übergeben. Solange beide Komponenten sich an das vom Entwickler vorgebene Interace \(Stichwort PropTypes\) halten sind beide Daten beliebig austauschbar und können vollkommen unabhängig voneinander entwickelt und **getestet** werden!

Genug Theorie. Zeit für ein weiteres Beispiel. Wir wollen uns eine Liste mit den 10 größten Kryptowährungen laden und ihren momentanen Preis anzeigen. Dazu erstellen wir eine Higher Order Component die sich diese Daten über die freie Coinmarketcap API beschafft und an eine Layout-Komponente übergibt.

```jsx
const withCryptoPrices = (WrappedComponent) => {
  return class extends React.Component {
    state = {
      isLoading: true,
      items: []
    };

    componentDidMount() {
      this.loadData();
    }

    loadData = async () => {
      this.setState(() => ({
        isLoading: true,
      }))
      try {
        const cryptoTicker = await fetch(
          'https://api.coinmarketcap.com/v2/ticker/?limit=10&convert=EUR'
        );
        const cryptoTickerResponse = await cryptoTicker.json();

        this.setState(() => ({
          isLoading: false,
          items: this.convertResponseToArray(cryptoTickerResponse)
        }));
      } catch (err) {
        this.setState(() => ({
          isLoading: false,
        }));
      }
    };

    convertResponseToArray = (response) => {
      return Object.entries(response.data).map(([id, item]) => item);
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

Voilà, fertig ist die **HOC** für die Abfrage der Crypto-Preise auf coinmarketcap.com. Doch die Higher Order Component allein reicht wie gesagt noch nicht. Wir benötigen nun auch noch eine Layout-Komponente, der wir die Verantwortung übertragen die Daten entsprechend anzuzeigen. 

Hierzu erstellen wir eine generische `PriceTable`-Komponente, die selbst keinerlei Kenntnis davon hat ob sie nun die aktuellen Joghurtpreise aus dem örtlichen Supermarkt darstellt oder Preise von Kryptowährungen auf irgendeiner beliebigen Börse. Entsprechend nennen wir sie auch sehr generisch `PriceTable`:

```jsx
const PriceTable = ({ isLoading, items, loadData }) => {
  if (isLoading) {
    return <p>Loading prices. Please be patient.</p>;
  }

  if (!items || items.length === 0) {
    return (
      <p>
        No data available. <button onClick={loadData}>Try again</button>
      </p>
    );
  }

  return (
    <table>
      {items.map(item => (
        <tr key={item.id}>
          <td>
            {item.name} ({item.symbol})
          </td>
          <td>EUR {item.quotes.EUR.price}</td>
        </tr>
      ))}
      <tr>
        <td colSpan="2">
          <button onClick={loadData}>Refresh</button>
        </td>
      </tr>
    </table>
  );
};
```

Die Komponente kennt drei Props: `isLoading`, um ihr mitzuteilen, dass die Daten die sie einmal darstellen soll aktuell geladen werden, `items`, was ein Array aus „Artikeln“ mit Preisen repräsentiert und `loadData`, eine Funktion um erneut einen API-Request zu starten um die neuen Daten zu beziehen.

Beide Komponenten funktionieren vollkommen unabhängig voneinander. Die PriceTable kann nicht nur Crypto-Preise anzeigen, die withCryptoPrices-Komponente muss ihre Daten nicht zwangsweise in einer PriceTable darstellen. Wir haben hier also zwei vollständig gekapselte und wiederverwendbare Komponenten erstellt!

Doch wie bringen wir die beiden nun zusammen? Ganz einfach indem wir die `PriceTable`-Komponente als Parameter an die `withCryptoPrices`-Komponente übergeben. Aha! Und das sieht wie folgt aus:

```jsx
const CryptoPriceTable = withCryptoPrices(PriceTable);
```

That's it! Rendern wir nun eine Instanz der `CryptoPriceTable`, stößt die HOC beim `componentDidMount()` einen API-Request an und übergibt das Ergebnis dieses Requests an die ihr übergebene `PriceTable`-Komponente. Diese kümmert sich anschließend nur noch um die entsprechende Darstellung.

```jsx
ReactDOM.render(
  <CryptoPriceTable />, 
  document.getElementById("root")
);
```



