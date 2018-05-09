# Formulare

Formulare besitzen in React eine kleine Sonderstellung von funktionieren etwas anders als andere DOM-Elemente, da Formulare eine Art **eigenen State** besitzen, der erst einmal nichts mit dem React-State gemein hat. 

Der State von Textfeldern besteht bspw. aus dem eingegebenen Wert, der State von Checkboxen oder Radiobuttons resultiert aus der Tatsache ob diese ausgewählt sind oder nicht, Auswahllisten \(`<select></select>`\) halten als State den ausgewählten Wert bzw. bei Mehrfachauswahl die ausgewählten Werte. React ändert an diesem Verhalten grundsätzlich erstmal nichts. Wer möchte, kann das so beibehalten und muss sich um nichts weiter kümmern. Im React-Jargon ist dann die Rede von **Uncontrolled Components**, also **unkontrollierten Komponenten**. Unkontrolliert deshalb, weil React sich nicht um das State-Management dieser Komponenten kümmert. 

Dabei ist es immer noch möglich Event-Handler zu definieren die den Wert eines Eingabefelds bspw. in den React-State der umgebenden Komponente schreiben:

```jsx
class Uncontrolled extends React.Component {
  state = {};

  submitForm = (e) => {
    e.preventDefault();
    alert(`Username: ${this.state.username}`);
  }

  changeUsername = (e) => {
    const { value } = e.target;
    this.setState(() => ({
      username: value,
    }));
  }

  render() {
    return (
      <form method="post" onSubmit={this.submitForm}>
        <input type="text" name="username" onChange={this.changeUsername} />
        <input type="submit" />
      </form>
    )
  }
}
```

Hier sehen wir ein einfaches Textfeld in das der Benutzer einen gewünschten Benutzernamen eintragen kann. Die `Uncontrolled` Komponente wird mittels `onChange`-Event von jeder Änderung in Kenntnis gesetzt und kann den Benutzernamen weiterverarbeiten. Da React hier nur **passiv** agiert, also bei einer Änderung am Textfeld über den neuen Wert in Kenntnis gesetzt wird, bewegen wir uns immer noch im Bereich der **Uncontrolled Components**.



