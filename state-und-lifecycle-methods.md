# State und Lifecycle-Methods

Kommen wir zu dem, was die Arbeit mit React erstmal wirklich effizient macht: **State**, **stateful Components** und die sogenannten **Lifecycle-Methods**.

Wie im Kapitel über Komponenten bereits angesprochen können **stateful Components** einen eigenen **State** halten, verwalten und verändern. Dabei gilt der Grundsatz: **ändert sich der State einer Komponente, löst dies immer auch ein Re-Rendering der Komponente aus!** Dieses Verhalten kann auch explizit unterbunden werden, was in einigen Fällen sinnvoll ist. Aber der Grundsatz bleibt unverändert: eine State-Änderung löst ein Re-Rendering einer Komponente und ihrer Kind-Komponenten aus.

Das ist insofern hilfreich, dass wir nicht mehr manuell ReactDOM.render\(\) aufrufen müssen wann immer wir meinen dass ich etwas an unserem Interface geändert hat, sondern die Komponenten dies selbst entscheiden können.

