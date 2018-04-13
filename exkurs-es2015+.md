# Exkurs ES2015+

## Das „neue“ JavaScript

ES2015 ist kurz gesagt eine modernisierte, aktuelle Version von JavaScript mit vielen neuen Funktionen und Syntax-Erleichterungen. Auf die wichtigsten möchte ich hier kurz eingehen, da ich sie in den nachfolgenden Beispielen verwenden werde. ES2015 hieß ursprünglich einmal ES6 und wird auch in einigen Blogs und Artikeln immer noch so bezeichnet. Stößt du also beim Lesen von Artikeln zu React auf den Begriff ES6 ist damit ES2015 gemeint. Ich schreibe hier meist von ES2015+ und meine damit Änderungen die seit 2015 in JavaScript eingeflossen sind.

{% hint style="info" %}
Das **ES** in ES2015 und ES6 steht für **ECMAScript**. Die ECMA International ist die Organisation, die hinter der Standardisierung der **ECMA-262** Spezifikation steht, auf der JavaScript basiert. Seit 2015 werden jährlich neue Versionen der Spezifikation veröffentlicht die aus historischen Gründen erst eine fortlaufende Versionsnummer beginnend ab Version 1 hatten, dann jedoch für mehr Klarheit die Jahreszahl ihrer Veröffentlichung angenommen haben. So wird ES6 heute offiziell als **ES2015** bezeichnet, ES7 als **ES2016**, usw.
{% endhint %}

Wenn du bereits Erfahrung mit ES2015 und den nachfolgenden Versionen hast kannst du dieses Kapitel überspringen. 

Wer mit React arbeitet nutzt in vermutlich 99% der Fälle auch Babel als Transpiler um sein JSX entsprechend in `createElement()`-Aufrufe zu transpilieren. Doch Babel transpiliert nicht nur JSX in ausführbares JavaScript, sondern hieß ursprünglich mal **6to5** und hat genau das gemacht: mit ES6-Syntax geschriebenes JavaScript in ES5 transpiliert, so dass neuere, zukünftige Features und Syntax-Erweiterungen auch in älteren Browsern ohne Unterstützung für „das neue“ JavaScript genutzt werden konnten.

## Neue Variablen-Deklarationen mit let und const

Gab es bisher nur `var` um eine Variable zu deklarieren in JavaScript, kommen in ES2015 zwei neue Schlüsselwörter dazu mit denen Variablen deklariert werden können: `let` und `const`. Eine Variablendeklaration mit `var` wird dadurch in fast allen Fällen überflüssig, meist sind `let` oder `const` die sauberere Wahl. Doch wo ist der Unterschied? 

Anders als `var` existieren mit `let` oder `const` deklarierte Variablen **nur innerhalb des Scopes in dem sie deklariert wurden!** Ein solcher Scope kann eine Funktion sein wie sie bisher auch schon bei `var` einen neuen Scope erstellt hat aber auch Schleifen oder gar `if` Statements!

**Grobe Merkregel:** überall dort wo man eine öffnende geschweifte Klammer findet, wird auch ein neuer Scope geöffnet. Konsequenterweise schließt die schließende Klammer diesen Scope wieder. Dadurch sind Variablen deutlich eingeschränkter und gekapselter, was für gewöhnlich eine gute Sache ist. 

Möchte man den Wert einer Variable nochmal überschreiben, beispielsweise in einer Schleife, ist die Variable dafür mit `let` zu deklarieren. Möchte man die Referenz der Variable unveränderbar halten, sollte `const` benutzt werden.

Doch Vorsicht: anders als bei anderen Sprachen bedeutet `const` nicht, dass der komplette Inhalt der Variable konstant bleibt. Bei Objekten oder Arrays kann deren Inhalt auch bei mit `const` deklarierten Variablen noch verändert werden. Es kann lediglich das Referenzobjekt auf welche die Variable zeigt nicht mehr verändert werden.

### Beispiele



