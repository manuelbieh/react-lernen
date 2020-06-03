# Functions as a Child und Render Props

**Functions as a Child** \(kurz: _FaaC_\) und **Render Props** werden in der offiziellen React-Dokumentation jeweils separat beschrieben, wobei **Functions as Children** im Kapitel zu **Render Props** ebenfalls erwähnt werden. Da beide vom Prinzip her ziemlich identisch funktionieren, möchte ich die beiden Konzepte in einem Kapitel beschreiben. Doch erst einmal: Worum geht es überhaupt?

Bei **Functions as a Child** \(in der offiziellen Doku auch als **Function as Children** bezeichnet\) und bei **Render Props** handelt es sich um Patterns, die ähnlich wie **Higher Order Components** Business- oder Applikations-Logik in einer Art übergeordneten Komponente bündeln. Anders als bei **Higher Order Components** wird jedoch keine neue Komponente zurückgegeben, dieser dann entsprechende Daten als Props übergeben werden, sondern es wird eine Funktion aufgerufen, die die entsprechenden Daten als Parameter übergeben bekommt. Beim **Function as Children-**Pattern ist diese Funktion ein Kind-Element der Komponente \(also: `this.props.children`\), beim **Render Props-**Pattern eine Prop die in den meisten Fällen den Namen `render` \(also: `this.props.render`\) hat, aber auch jeden anderen Namen haben könnte.

Wir wissen bereits, dass der Wert einer Prop in JSX jeder beliebige valide Ausdruck in JavaScript sein kann. Auch aufgerufene Funktionen können Ausdrücke zurückgeben und so können wir nicht nur Strings, Booleans, Arrays, Objekte, andere React-Elements oder `null` als Wert für unsere Props verwenden, sondern eben auch den return-Wert einer aufgerufenen Funktion. Wir haben auch gelernt, dass `children` in React nur eine Art Sonderform einer Prop sind und so haben die folgenden beiden Zeilen jeweils das gleiche Rendering-Ergebnis zur Folge:

```jsx
<MyComponent>Ich bin ein Child-Element</MyComponent>
<MyComponent children="Ich bin ein Child-Element" />
```

In der `MyComponent`-Komponente kann dann mittels `props.children` auf den _Ich bin ein Child-Element_ Text zugegriffen werden.

Dies können wir uns zu nutze machen und eben auch Funktionen übergeben, die dann in der `render()`-Methode einer Komponente aufgerufen werden. Die Idee dahinter ist, dass auf diesem Weg beliebige Daten von einer Komponente in die nächste übergeben werden können. Ähnlich wie bei **Higher Order Components**, jedoch mit etwas mehr Flexibilität. So müssen wir z.B. nicht die ganze Komponente mit  einer **Higher Order Component** „verbinden“, sondern können dies einfach mittendrin im JSX unserer Komponente tun. Denken wir zurück an die `withFormatting` HOC aus dem vorherigen Kapitel. Eine entsprechende als **Function as a Child \(FaaC\)**-Komponente implementiert könnte etwa so aussehen:

```jsx
const bold = (string) => {
  return <strong>{string}</strong>
}

const italic = (string) => {
  return <em>{string}</em>
}

const Formatter = (props) => {
  if (typeof props.children !== 'function') {
    console.warn('children prop must be a function!');
    return null;
  }

  return props.children({ bold, italic });
}
```

Wir definieren also wieder eine `bold`- und eine `italic`-Funktion, prüfen in der `Formatter`-Komponente, ob die übergebene `children`-Prop eine Funktion ist und rufen diese Funktion auf. Weiter übergeben wir ihr als einzigen Parameter ein Objekt mit den Eigenschaften `bold` mit der `bold`-Funktion als Wert sowie `italic` mit der `italic`-Funktion als Wert. Gleichzeitig geben wir die aufgerufene Funktion aus der Komponente zurück.

Bei der Verwendung dieser **Function as Children**-Komponente wird dann eben eine Funktion im JSX als Kind-Element übergeben. Dies funktioniert wie folgt:

```jsx
<div>
  <p>Dieser Text hat keinerlei Kenntnis der Formatter-Funktionen</p>
  <Formatter>
  {({ bold }) => (
    <p>Dieser Text hingegen {bold('sehr wohl')}</p>
  )}
  </Formatter>
</div>
```

Der Nutzen dieses Ansatzes ist die besagte Flexibilität, da wir nun nicht mehr die ganze Komponente selbst mit einer Higher Order Funktion verbinden müssen, nur um vielleicht an einer einzigen Stelle auf dessen wiederverwendbare Funktionalität zurückgreifen zu können. Anders als bei Higher Order Components ist es auf diese Art auch möglich, Parameter direkt aus dem JSX an die Function as a Child Komponente zu übergeben und so mit dieser zu kommunizieren.

Schauen wir uns dazu noch einmal unser zweites Beispiel aus dem Kapitel über Higher Order Components an, die Preisliste der Kryptowährungen, und implementieren diese als Function as a Child:

```jsx
class CryptoPrices extends React.Component {
  state = {
    isLoading: true,
    items: []
  };

  componentDidMount() {
    this.loadData();
  }

  loadData = async () => {
    this.setState(() => ({
      isLoading: true
    }));

    const { limit } = this.props;

    try {
      const cryptoTicker = await fetch(
        `https://api.coingecko.com/api/v3/coins/markets?vs_currency=eur&per_page=${limit || 10}`
      );
      const cryptoTickerResponse = await cryptoTicker.json();

      this.setState(() => ({
        isLoading: false,
        items: cryptoTickerResponse
      }));
    } catch (err) {
      this.setState(() => ({
        isLoading: false
      }));
    }
  };

  render() {
    const { isLoading, items } = this.state;
    const { children } = this.props;

    if (typeof children !== "function") {
      return null;
    }

    return children({
      isLoading, 
      items, 
      loadData: this.loadData 
    });
  }
}
```

Auf den ersten Blick sieht die Komponente gar nicht mal so anders aus als die Higher Order Component aus dem vorherigen Kapitel. Doch wer genau hinschaut erkennt:

* Es wird keine weitere Komponente mehr erzeugt und zurückgegeben, sondern es wird direkt mit der Komponente gearbeitet
* Die `loadData`-Methode greift auf `this.props` zu, um daraus die `limit`-Prop abzulesen. Diese wird als Parameter für den API Call verwendet.
* Die `render()`-Methode gibt nun keine in die Komponente hereingegebene Komponente mehr zurück, sondern ruft stattdessen die `children`-Funktion aus, die sie aus ihren eigenen Props bekommt.
* Die `children`-Funktion hingegen bekommt den Lade-Status \(`isLoading`\) sowie letztendlich die items zurück.

Die Verwendung dieser Komponente ist dann ähnlich zu der aus dem ersten Beispiel mit dem kleinen Unterschied, dass wir der Komponente optional eine `limit`-Prop übergeben können:

```jsx
<div>
  <h1>Current Crypto Currency Prices</h1>
  <CryptoPrices limit={5}>
    {({ isLoading, items }) =>
      isLoading ? (
        <p>Loading prices. Please be patient.</p>
      ) : (
        <ul>
          {items.map((item) => (
            <li>
              {item.name} ({item.symbol}): EUR {item.current_price}
            </li>
          ))}
        </ul>
      )
    }
  </CryptoPrices>
</div>
```

An dieser Stelle kommt ebenfalls auch wieder die `PriceTable`-Komponente ins Spiel. Diese erwartete genau die drei Props, die wir aus der `CryptoPrices`-Komponente zurückgeben. Na, so ein Zufall! Schauen wir uns das doch einmal an, wie wir die beiden miteinander verbinden können:

```jsx
<CryptoPrices limit={5}>
  {({ isLoading, items, loadData }) => (
    <PriceTable isLoading={isLoading} items={items} loadData={loadData} />
  )}
</CryptoPrices>
```

Oder um es mittels der Spread-Syntax kurz zu machen:

```jsx
<CryptoPrices limit={5}>
  {(props) => <PriceTable {...props} />}
</CryptoPrices>
```

Auf diese Art und Weise haben wir eine sehr hohe Flexibilität gewährleistet, müssen Komponenten jedoch nicht starr über eine HOC mit der Logik verbinden, was uns einiges an „organisatorischem Aufwand“ erspart. 

Doch Vorsicht: **Functions as a Child-Komponenten haben auch eine Einschränkung die Higher Order Components** nicht haben. Nämlich können die Daten, die wir aus einer **FaaC**-Komponente beziehen **nur innerhalb von JSX** verwendet werden! Möchten wir also relativ abstrakte Methoden über eine höher in der Komponenten-Hierarchie stehende Logik-Komponente bereitstellen, ist dies mit einer **FaaC**-Komponente erst einmal nicht oder nur umständlich möglich!

### Render Props

Doch Moment mal, wie war das jetzt eigentlich mit den **Render Props** und was ist das jetzt genau und wie unterscheiden sich diese von **Function as Children**-Komponenten?

Vereinfacht gesagt: nur durch den Namen der Prop. Einige populäre Libraries aus der React-Welt hatten irgendwann damit angefangen `render` als Name für Props zu benutzen, die Funktionen als Wert erwarten. Und so würde unsere `CryptoPrices`-Komponente, benutzten wir statt `children` eine `render`-Prop, folgendermaßen aussehen:

```jsx
<CryptoPrices limit={5} render={(props) => <PriceTable {...props} />} />
```

Innerhalb der `CryptoPrices`-Komponente muss es dann natürlich heißen:

```jsx
render() {
  const { isLoading, items } = this.state;
  const { render } = this.props;

  if (typeof render !== "function") {
    return null;
  }

  // Achtung: dieses render() hat nichts mit der gleichnamigen Komponenten Methode
  // zu tun sondern kommt über this.props.render in die Komponente hinein!
  return render({
    isLoading, 
    items, 
    loadData: this.loadData 
  }); 
}
```

Ist also ein Stück weit auch Geschmackssache. Dabei seid ihr natürlich auf den Namen `render` nicht festgelegt, sondern könnt einfach jeder beliebigen Prop einfach eine Funktion übergeben und diese somit zu einer _„Render Prop“_ machen. 

Dabei ist es auch möglich, beliebig viele solcher Props in einer Komponente zu haben. Wollt ihr beispielsweise eine Komponente implementieren, die euch eine Tabelle ausgibt, welche sowohl einen Tabellenkopf als auch einen Body besitzt, die beide jeweils Daten aus der Komponente beziehen ist auch dies kein Problem!

### Render Props und FaaCs in Verbindung mit Higher Order Components

Zum Abschluss noch ein kleiner Trick. Solltet ihr tatsächlich einmal eine **Higher Order Component** benötigen und ihr habt bereits eine **FaaC-** oder **Render Prop**-Komponente, könnt ihr diese auch zu einer HOC machen:

```jsx
function withCryptoPrices(WrappedComponent) {
  return class extends React.Component {
    render() {
      return (
        <CryptoPrices>
          {(cryptoPriceProps) => (
            <WrappedComponent {...this.props} {...cryptoPriceProps} />
          )}
        </CryptoPrices>
      );
    }
  };
}
```

In der Praxis wird dieser Fall erfahrungsgemäß eher selten eintreffen.

{% hint style="info" %}
Das **Function as a Child**-Pattern und das nahezu identische **Render-Props**-Pattern werden verwendet, um Business Logik von Darstellungs-Komponenten zu trennen. Sie sind eine sehr leichtgewichtige Alternative zu **Higher Order Components**, die so ziemlich den gleichen Anwendungsfall bedienen. 

Sie lassen sich anders als HOCs jedoch auch innerhalb einer `render()`-Methode von Komponenten verwenden und müssen nicht starr mit einer Komponente „verknüpft“ werden. Dies macht sie noch etwas flexibler in ihren Einsatzmöglichkeiten als **Higher Order Components,** ohne dabei an Übersichtlichkeit einzubüßen.
{% endhint %}

