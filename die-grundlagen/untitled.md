# Listen, Refs und Conditional Rendering

Bis hier her habt ihr schon eine ganze Menge über React gelesen. Ihr wisst wofür die Props sind, was der State ist, wie eine React-Komponente implementiert wird, was der Unterschied einer React-Komponente und einem React-Element ist und wie ihr mit JSX einen Elementenbaum beschreibt, der später in eurer Anwendung gerendert wird. Damit habt ihr auch schon alles beisammen um eine simple React-Anwendung zu entwickeln. 

Allerdings gibt es noch einige Details, die in den vorherigen Kapiteln bisher gar keine Erwähnung fanden oder ohne weitere Erklärung in Beispielen benutzt wurden, die aber, gerade wenn eure Anwendung anfängt komplexer zu werden zunehmend wichtiger werden.

Im Speziellen sind das sogenannte **Refs**, damit sind Referenzen zu DOM-Repräsentationen von React-Elementen gemeint, **Fragments**, eine spezielle Art Komponente, die keine Spuren im gerenderten Output hinterlässt, **Listen**, also Arrays mit Daten und **Conditional Rendering**, also Unterscheidungsmöglichkeiten, wann ihr was rendert, basierend auf **Props** und **State**.

## Listen

Mit Listen sind hier tatsächlich stumpfe JavaScript-Arrays gemeint, also einfache Daten, durch die iteriert werden kann. Sie sind bei der Arbeit mit React alltäglich und keine Anwendung kommt ohne sie aus. **ES2015+** bietet uns mit `Array.map()`, `Array.filter()` und `Array.find()` schöne Methoden, die wir als Ausdrücke in JSX innerhalb von geschweiften Klammern `{}` weiterverarbeiten können.

Welche Rolle Ausdrücke in JSX spielen und wie wir Ausdrücke in JSX nutzen können, habe ich bereits im Kapitel über JSX angesprochen. Kurz aufgefrischt: Arrays können als Ausdruck in JavaScript genutzt werden und somit auch in JSX. Das heißt sie können in geschweiften Klammern stehen und werden dann von beim Transpiling von JSX als Childnode behandelt.



## Refs

## Conditional Rendering



