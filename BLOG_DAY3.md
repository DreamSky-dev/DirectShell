# Blog 19.02.26

## Migraine, Stress und die Suche nach der Nadel im Heuhaufen.

Es ist eine interessante Situation. Auf der einen Seite trudeln hier und da interessante Kommentare ein:

> **Ozz** - *Feb 18*
>
> This is absurdly cool! Fast as lightning. Managed to build a macOS version of it in an hour :) THANKS! I'm sure this is an idea that will not go back to the bag. would be cool to see how this gets integrated to "everything"... but for now it makes claude code so much smarter.
>
> THANKS! :)
>
> *(Dev.to Kommentar)*

Auf der anderen Seite hat man das Gefuehl nichts bewegt sich. 100 Downloads. Einige Hundert Reads auf den Papern aber es scheint fast wie ein kollektives "das ist interessant aber was machen wir jetzt damit?" zu sein.

Vielleicht liegt das einfach daran das noch effektiv die Anwendungen fehlen. DirectShell verspricht Fortschritt aber das ist natuerlich erst dann greifbar wenn die ersten Anwendungen auf Basis von DS entstanden sind die auch praktisch funktionieren.

Ich selbst fokussiere mich auf den KI-Agenten Usecase weil ich glaube das es einer der vielversprechendsten ist aber man merkt auch einfach das man da ploetzlich vor der Situation steht etwas zu bauen zu dem es keine Referenz gibt. Kein "mal kurz Googlen wie mache ich das". Keine Best Practice. Reines Try and Error.

Und ich kann euch das sagen: das ist anstrengend.

## Wo stehe ich?

Den Gemini-Ansatz der einige der Rohdaten in "Haeppchen zerlegt" habe ich wieder eingestampft. Gestern gedacht es ist klug, heute gemerkt: Nein, ich will deterministische Loesungen auch wenn sie schwerer sind.

Ich habe nun also eine Mechanik in den MCP Server gebaut der die relevanten Daten aus dem CDP Port des Browsers holt, aufbereitet und daraus automatisch die notwendigen Tools dynamisch abbildet.

Also: Kein zusaetzlicher API Call. Keine weiteren Kosten. Das ist gut.

Die groesste ungeloeste Frage vor der ich nach wie vor stehe ist wie ich es realisiere das Lernfortschritte der KI direkt in einen Learning Loop gegossen werden koennen. Das Problem ist klar — kannst du der KI sagen "merk dir das". Aber das ist nicht zuverlaessig.

Es muss eine Methode her die autonom, deterministisch und zuverlaessig Learnings extrahiert und diese wieder als permanenten Loop-Kontext bewusst in die KI zurueck subventioniert.

Ansonsten gabs einige coole PNs und Anfragen. Einige Meetings sind geplant. Aber noch nichts Konkretes.

In dem Sinne — euch einen wunderschoenen Donnerstag <3 der erste von gestern sollte schon irgendwo sein.
