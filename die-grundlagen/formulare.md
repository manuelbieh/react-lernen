# Formulare

Formulare besitzen in React eine kleine Sonderstellung und funktionieren etwas anders als andere DOM-Elemente, da Formulare eine Art **eigenen State** besitzen, der erst einmal nichts mit dem React-State gemein hat. 

Der State von Textfeldern besteht bspw. aus dem eingegebenen Wert, der State von Checkboxen oder Radio-Buttons resultiert aus der Tatsache ob diese ausgewählt sind oder nicht, Auswahllisten \(`<select></select>`\) halten als State den ausgewählten Wert bzw. bei Mehrfachauswahl die ausgewählten Werte. React ändert an diesem Verhalten grundsätzlich erstmal nichts. Wer möchte, kann das so beibehalten und muss sich um nichts weiter kümmern. 

Im React-Jargon ist dann die Rede von **Uncontrolled Components**, also **unkontrollierten Komponenten**. Unkontrolliert deshalb, weil React sich nicht um das State-Management dieser Komponenten kümmert. Das State-Handling ist entweder vollständig unabhängig von React oder funktioniert nur aus Richtung der DOM Formular-Elemente hin zum React State, jedoch **nicht in die entgegengesetzte Richtung**. Von einem Update am React-State bekommt ein Formular-Element also nichts mit und zeigt weiter den gleichen Wert \(oder Status bei Checkboxen, Selects und Radiobuttons\) wie zuvor.

Demgegenüber stehen die **Controlled Components**, also **kontrollierte Komponenten**. Hier aktualisiert ein Update am React-State den Wert \(oder Status\) des Formular-Elements und ebenso aktualisiert ein Update am jeweiligen Formular-Element den React-State. **Controlled Components** sind etwas aufwändiger in der Implementierung, sind zugleich jedoch auch die „sicherere“ Variante, da wir nicht Gefahr laufen, dass beide States voneinander abweichen.

## Uncontrolled Components / unkontrollierte Komponenten

**Unkontrollierte Komponenten** können dabei im Wesentlichen in zwei verschiedenen Formen auftreten. Bei der ersten Variante werden einfach nur Formular-Elemente gerendert, die beim Abschicken bspw. rein serverseitig verarbeitet werden und in keiner Weise mit React interagieren. Ein komplett statisches Formular wenn man so will. React kümmert sich dabei **nicht von alleine** um die Anbindung an den React-State sondern **lässt dem Entwickler hier sämtliche Freiheiten**!

Bei der zweiten Variante werden Änderungen an einem Formular-Element **in den React-State** geschrieben um bspw. im Hintergrund eine Validierung der Daten vorzunehmen oder die eingegebenen Daten an anderer Stelle auszugeben. Eine Änderung am React-State an anderer Stelle der Anwendung hat dabei keinerlei direkten Einfluss auf die Formularfelder. 

Ein Beispiel für eine solche unkontrollierte Komponente: 

```jsx
class Uncontrolled extends React.Component {
  state = {
    username: '',
    isValid: false,
  };

  changeUsername = e => {
    const { value } = e.target;
    this.setState(() => ({
      username: value,
      isValid: value.length > 3,
    }));
  };

  submitForm = e => {
    e.preventDefault();
    alert(`Hallo ${this.state.username}`);
  };

  render() {
    return (
      <form method="post" onSubmit={this.submitForm}>
        <p>Dein Benutzername: {this.state.username}</p>
        <p>
          <input 
            type="text" 
            name="username" 
            onChange={this.changeUsername} />
          <input type="submit" disabled={!this.state.isValid} />
        </p>
      </form>
    );
  }
}
```

Hier sehen wir ein einfaches Textfeld in das der Benutzer einen gewünschten Benutzernamen eintragen kann. Die `Uncontrolled` Komponente wird mittels `onChange`-Event von jeder Änderung in Kenntnis gesetzt und kann den Benutzernamen weiterverarbeiten. Da React hier nur **passiv** agiert, also bei einer Änderung am Textfeld über den neuen Wert in Kenntnis gesetzt wird, bewegen wir uns immer noch im Bereich der **Uncontrolled Components**.

Dies ist in einigen Fällen  ausreichend, insbesondere wenn die Formulare noch nicht all zu komplex sind. Allerdings ist der React-State hier vom DOM State **entkoppelt** bzw. funktioniert nur **in eine Richtung**. Der React State wird aktualisiert sobald der `onChange`-Event des Textfelds ausgelöst wird. Allerdings bedeutet dies, dass nicht gleichzeitig auch unser Textfeld aktualisiert wird wenn der Wert im React State an anderer Stelle verändert wurde, bspw. weil der Response eines asynchronen Requests nach einiger Zeit eintrifft.

Ein Formularfeld gilt als **kontrolliert**, sobald ein `value`-Attribut gesetzt wird. Ab diesem Moment erwartet React, dass wir uns als Entwickler selbst darum kümmern den React-State mit dem Formularfeld zu synchronisieren. Möchten wir allerdings nur einmalig einen initialen Wert setzen ohne gleich die ganze Komponente zu einer **Controlled Component** zu machen, haben wir die Möglichkeit statt des `value`-Attributs das React eigene `defaultValue`-Attribut zu setzen \(`defaultChecked` bei Checkboxen und Radiobuttons\). Das Element bleibt dann weiterhin **unkontrolliert**, zeigt aber dennoch einen vorausgefüllten Wert \(bzw. Status\) an.

## Controlled Components / kontrollierte Komponenten

Um sowohl State-Updates in Formularfeldern zu abzubilden als auch auf der anderen Seite benutzerseitige Änderungen an Formularfeldern in den React-State zu übertragen, benötigen wir eine **Controlled Component**. Hier überlassen wir das State-Handling eines Formular-Elements vollständig React. Dies bedeutet, dass wir das `value`-Attribut mit einem Wert befüllen den wir aus dem React-State beziehen und gleichzeitig auch einen geänderten Wert wieder zurück in den React-State überführen.

Das Ziel bei diesem Ansatz ist es, den React-State \(oder einen anderen State-Container wie z.B. Redux\) als **Single Source of Truth** zu betrachten, also als die _einzige Quelle der Wahrheit_. Relevant ist der Wert der im von React verwalteten State steht, das jeweilige Eingabefeld reflektiert dann zu jedem Zeitpunkt den Wert aus diesem State.

Schauen wir uns auch hierzu mal ein Beispiel an:

```jsx
class Controlled extends React.Component {
  state = {
      username: '',
      isValid: false,
  };

  changeUsername = (e) => {
    const { value } = e.target;
    this.setState(() => ({
      username: value,
      isValid: value.length > 3
    }));
  };

  submitForm = (e) => {
    e.preventDefault();
    alert(`Hallo ${this.state.username}`);
  };

  render() {
    return (
      <form method="post" onSubmit={this.submitForm}>
        <p>{username}</p>
        <p>
          <input
            type="text"
            name="username"
            onChange={this.changeUsername}
            value={this.state.username}/>
          <input type="submit" disabled={!this.state.isValid} />
        </p>
      </form>
    );
  }
}
```

Auf den ersten Blick unterscheidet sich die `Controlled` Komponente gar nicht sonderlich von der `Uncontrolled` Komponente im Absatz oben. Und tatsächlich ist der entscheidende Faktor der die unkontrollierte Komponente zu einer kontrollierten werden lässt einzig und allein das `value`-Attribut des `<input />`-Elements. Ist ein solches vorhanden **kontrolliert** React das Formular-Element und erwartet, dass sich Änderungen am Eingabefeld entsprechend im State widerspiegeln. Wichtig ist außerdem der `onChange`-Handler, um den Wert bei einer Änderung jeweils in den React-State zu übertragen. Ein gern gemachter Fehler wenn das erste Mal mit Formularen in React gearbeitet wird ist, entsprechende Eingabefelder nicht mit dem React-State zu synchronisieren indem der neue Wert in den State geschrieben wird. Das Eingabefeld verändert sich dann nicht und zeigt weiterhin den alten Wert aus `this.state` an.

Hier gibt es noch einige weitere Dinge zu beachten. So darf der Wert des `value`-Attributes immer nur ein **String** sein, niemals `undefined` oder `null`.

![Warnung bei einem kontrollierten Textfeld mit dem value &quot;null&quot;](../.gitbook/assets/react-uncontrolled-null.png)

Eine Ausnahme sind hier `select`-Elemente die ein `multiple`-Attribut besitzen. Hier **muss** das `value`-Attribut ein **Array** sein. 

Moment mal, denkt ihr euch jetzt vielleicht. Welches `value`-Attribut beim `<select>`? Optionen selektiere ich doch, indem ich das `selected`-Attribut bei der jeweiligen `<option>` setze! Und ja, das ist korrekt in HTML, in React funktioniert das ein klein wenig anders. Hier wird der kontrollierte Wert ebenfalls über das `value`-Attribut gesetzt. Dasselbe gilt übrigens auch für das `<textarea>`-Element, dessen Initialwert für gewöhnlich durch seinen `textContent` bestimmt wird. Nicht so in React. 

React vereinheitlicht hier den Mechanismus zum ändern von Werten etwas und erfordert für die drei Elemente `input` \(alle Typen mit Ausnahme `checkbox` und `radio`\), `textarea` und `select` ein `value`-Attribut! Bei einfachen Werten muss dies **immer ein String** sein, bei einer Auswahlliste mit dem `multiple`-Attribut wie eben erwähnt ein **Array bestehend aus Strings**!

Darüber hinaus muss eine Änderung eines Formular-Elements **immer auch zurück in den React-State übertragen werden**. Dies kann mitunter etwas mühsam werden, insbesondere bei Checkboxen und Radiobuttons, bei denen nicht lediglich ein Wert geändert wird sondern der Status \(`checked`\) zu einem Wert.

Im folgenden möchte ich eine vollständig kontrollierte Komponente zeigen, die alle Grundtypen von Formular-Elementen die HTML beinhaltet \(andere `input`-Elemente vom vom Typ `email`, `date`, `range`, etc. funktionieren identisch wie Eingabefelder vom Typ `text`\).

```jsx
class FullyControlledComponent extends React.Component {
  state = {
    text: "",
    textarea: "",
    checkbox: false,
    singleSelect: "",
    multipleSelect: [],
  };

  changeValue = ({ target: { name, value } }) => {
    this.setState(() => ({
      [name]: value,
    }));
  };

  changeCheckbox = ({ target: { name, checked } }) => {
    this.setState(() => ({
      [name]: checked,
    }));
  };

  changeSelect = ({ target: { name, value, selectedOptions, multiple } }) => {
    if (multiple) {
      value = Array.from(selectedOptions).map((option) => option.value);
    }

    this.setState(() => ({
      [name]: value,
    }));
  };

  render() {
    return (
      <form>
        <input
          type="text"
          name="text"
          value={this.state.text}
          onChange={this.changeValue}
        />

        <textarea
          name="textarea"
          value={this.state.textarea}
          onChange={this.changeValue}
        />

        <input
          type="checkbox"
          name="checkbox"
          checked={this.state.checkbox}
          onChange={this.changeCheckbox}
        />

        <input
          type="radio"
          name="radio"
          value="1"
          checked={this.state.radio === "1"}
          onChange={this.changeValue}
        />
        <input
          type="radio"
          name="radio"
          value="2"
          checked={this.state.radio === "2"}
          onChange={this.changeValue}
        />

        <select
          name="singleSelect"
          value={this.state.singleSelect}
          onChange={this.changeValue}
        >
          <option value="">Bitte auswählen</option>
          <option value="1">One</option>
          <option value="2">Two</option>
        </select>

        <select
          name="multipleSelect"
          value={this.state.multipleSelect}
          onChange={this.changeSelect}
          multiple
        >
          <option value="1">One</option>
          <option value="2">Two</option>
        </select>

        <pre>{JSON.stringify(this.state, null, 2)}</pre>
      </form>
    );
  }
}
```

Kern des Formulars sind erst einmal die drei Event-Handler Methoden für die verschiedenen Formular-Element Typen: `changeValue`, `changeCheckbox` und `changeSelect`.

Sie werden jeweils beim onChange-Event der jeweiligen Formular-Elemente aufgerufen und bekommen ein Objekt vom Typ `SyntheticEvent` übergeben. Aus dessen `target`-Eigenschaft picken wir uns mittels **ES2015 Object Destructuring** wiederum einzelne Eigenschaften heraus um damit anschließend den State entsprechend zu aktualisieren.

Bei Elementen vom Typ `<input type="text" />`, `<input type="radio" />` und `<textarea />` sind das `name`und `value`, bei `<input type="checkbox" />` interessiert uns der `name` und die `checked`-Eigenschaft, bei `select`-Elementen interessiert uns in jedem Fall auch der `name` und dann, abhängig davon ob es eine einfache Auswahlliste ist oder eine Auswahlliste mit Mehrfachauswahl, wieder der `value`oder die `selectedOptions`. Ob wir eine es mit einer einfachen oder mehrfachen Auswahlliste zu tun haben finden wir mittels der `multiple`-Eigenschaft heraus, die wir uns ebenfalls aus der `e.target`-Eigenschaft herauspicken.

### Veränderung von Werten

Wird ein Wert verändert, wie es bei Text-Eingabe oder Radiobuttons der Fall ist, setzen wir eine gleichnamige State-Eigenschaft auf den jeweiligen Wert, den der Benutzer eingegeben hat und mit dem er den `onChange`-Event getriggert hat. Da wir uns in einer kontrollierten Komponente befinden, funktioniert nun folgendes:

1. Der Benutzer ändert mittels Texteingabe den Wert
2. Ein `onChange`-Event wird ausgelöst und im Event-Handler verarbeitet
3. Der Event-Handler setzt die State-Eigenschaft auf den neuen Wert
4. React re-rendert das User Interface und setzt die `value`-Eigenschaft des Eingabefelds auf den neuen Wert aus `this.state`.
5. Der Benutzer sieht seinen neu eingegebenen Wert. 

Für den Benutzer ist dies erstmal **Business as Usual**. Er bemerkt nicht, dass das Formular anders funktioniert als er das aus dem Browser kennt. Und doch hat sich hier React um die Logik im Hintergrund gekümmert und einen neuen „Frame“ im User Interface gezeichnet.

### Veränderung von Zuständen bei Checkboxen und Radiobuttons

Checkboxen \(`<input type="checkbox" />`\) funktionieren hier vom Ablauf im Hintergrund genauso, allerdings mit dem Unterschied das ihr Wert grundsätzlich gleich bleibt. Bei Checkboxen ändert sich statt des Werts der Zustand ihrer `checked`-Eigenschaft von `true` auf `false` oder andersherum. Sie gelten daher dann als kontrolliert, wenn ihre `checked`-Eigenschaft durch React gesteuert wird. Der `onChange`-Event bei Checkboxen teilt uns mittels `e.target.checked`  mit, ob die eben geänderte Checkbox nun aktiviert \(`true`\) oder nicht aktiviert \(`false`\) ist. Diesen Status geben wir unverändert an den React-State weiter, React kümmert sich dann im Re-Rendering darum, dass der neue Status der Checkbox angezeigt wird.

Radiobuttons sind eine Art Hybrid-Element. Sie gelten wie Checkboxen ebenfalls als kontrolliert wenn ihr `checked`-Attribut durch React verwaltet wird. Allerdings gibt es für gewöhnlich mehrere Radiobuttons mit dem selben Namen, allerdings mit unterschiedlichen Werten in einem Dokument. Hier würde es also keinen Sinn machen den Wert zu einem Namen auf `true` oder `false` zu setzen, da uns der tatsächliche Wert des ausgewählten Radiobuttons interessiert. Hier schreiben wir also wie bei Text-Elementen den Wert des Radiobuttons in den State und prüfen dann beim Rendering des jeweiligen Radiobuttons selbst, ob der ausgewählte Wert aus dem State dem eigenen Wert entspricht: `checked={this.state.radio === "1"}`. Also in diesem Beispiel: setze `checked` auf `true` wenn der Wert eines Radiobuttons mit dem Namen `radio` gleich `1` ist.

### Veränderung des Status bei einfachen oder mehrfachen Auswahllisten

Fangen wir mit dem einfachen Fall an: einfache `<select>`-Auswahllisten ändern ebenfalls wie Textfelder ihren Wert, lösen damit ein Re-Rendering aus und zeigen den ausgewählten Wert im neu gezeichneten User Interface an. Eine Ausnahme stellen hier mehrfache Auswahllisten dar.

Mehrfache Auswahllisten erwarten anders als Textfelder oder einfache Auswahllisten keinen String als Wert, sondern ein **Array aus Strings**. Diesen müssen wir uns allerdings selbst zusammenbasteln, da `e.target.value` bei Mehrfach-Auswahllisten nur einen einzigen Wert enthält, selbst bei der Auswahl mehrerer Optionen. Hier hilft uns `e.target.selectedOptions` weiter. Diese Eigenschaft ist ein Objekt vom Typ `HTMLCollection`, mit den `<option>`-Elementen die momentan ausgewählt sind. Dieses Objekt können wir mit der statischen Array-Methode `Array.from()` aus ES2015 ziemlich einfach in ein Array umwandeln. Indem wir mittels `Array.map()` über dieses iterieren, können wir außerdem ein neues Array erzeugen, das alle für uns relevanten Werte enthält: `Array.from(selectedOptions).map((option) => option.value);`

Das so erzeugte, neue Array schreiben wir dann als neuen Wert in unseren State. Zuvor schauen wir jedoch erst einmal mittels e.target.multiple ob es sich überhaupt um ein `<select>` mit Mehrfachauswahl handelt, da nur dieses ein Array als `value` erwartet.

Alternativ wäre es natürlich auch möglich einfachen Auswahllisten die `changeValue`-Methode als Event-Handler zu übergeben und nur Mehrfach-Auswahllisten die `changeSelect`-Methode. Dann könnten wir uns in selbiger den Check sparen ob es sich um ein `multiple` Select handelt. Es auf die obigen Weise zu lösen hat aber den Vorteil, dass später der Typ von mehrfach auf einfach geändert werden könnte ohne dass man zusätzlich noch den Event-Handler ändern muss. Aber das bleibt am Ende natürlich euch selbst überlassen.

### Besonderheiten bei kontrollierten Komponenten

Ich habe in den obigen Beispielen jeweils das `name`-Attribut der jeweiligen Elemente als Schlüssel benutzt um deren Wert im State zu speichern. Das ist insbesondere dann praktisch wenn man mit serverseitigem React arbeitet und bspw. Formulare auf Basis eines Datenbankschemas automatisch generiert und wieder verarbeitet. Voraussetzung für eine funktionierende kontrollierte Komponente ist das aber nicht. Es wird theoretisch weder ein `name`-Attribut benötigt noch muss schlussendlich der Name der State-Eigenschaft mit dem `name`-Attribut übereinstimmen. 

Ihr könnt die gespeicherten Werte auch verschachteln, was sich anbietet wenn ihr in einer Komponente mehrere Formulare haben solltet \(**Achtung:** React Antipattern!\). Darüber hinaus muss nicht einmal zwingend der React-State verwendet werden um **Controlled Components** abzubilden. Im Gegenteil, in der Praxis wird stattdessen oftmals auch auf einen externen State-Container wie **Redux**, **Unstated** oder **MobX** zurückgegriffen. 

## Fazit

{% hint style="success" %}
Formulare in React können in \(von React\) kontrollierter oder unkontrollierter Form auftreten.

**Unkontrollierte Komponenten** reichen für simple Formulare oftmals aus, allerdings empfiehlt es sich, Formular-Komponenten von React kontrollieren zu lassen um eine **Single Source of Truth** zu haben. Dazu muss das `value` bzw. `checked`-Attribut von React verwaltet werden. Auf Entwicklerseite muss dann manuell auf Änderungen reagiert werden.

Anders als in herkömmlichem HTML erwartet React den Wert von Textareas, Selects und Inputfeldern mit Texteingabe im `value`-Attribut.
{% endhint %}



