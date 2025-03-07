---
title: "Testen von regulären Ausdrücken mit Python und doctest"
date: 2023-07-01 10:00:10 +0200
categories: Python
---

Mit regulären Ausdrücken kannst du schnell und präzise wichtige Informationen in großen Textmengen finden und extrahieren. Ihre Syntax ist allerdings komplex, und Fehler können leicht passieren. Der häufigste Vorschlag für das zuverlässige Erstellen von regulären Ausdrücken ist die Verwendung einer der zahlreichen Online-Testplattform, die sich mit diesem Thema befassen. Ich nutze persönlich folgende:

* [regexr](https://regexr.com/)
* [regex101](https://regex101.com/)

Diese Plattformen eignen sich gut für die schnelle Erstellung eines regulären Ausdrucks. Allerdings lassen sich die Resultate nur schwer reproduzieren, dokumentieren und archivieren. Daher wähle ich mittlerweile immer häufiger einen anderen Weg.

Das in Python integrierte Paket [re](https://docs.python.org/3/library/re.html) bietet ausgezeichnete Unterstützung für reguläre Ausdrücke. Zusätzlich ermöglicht das Python-Paket [doctest](https://docs.python.org/3/library/doctest.html) das Erstellen kleiner Unit-Tests in den Kommentaren (docstrings) von Funktionen und Modulen, die dann ganz einfach ausgeführt werden können.

Die Kombination von [re](https://docs.python.org/3/library/re.html) und [doctest](https://docs.python.org/3/library/doctest.html) ist für mich der ideale Weg, um komplexe reguläre Ausdrücke zu entwickeln und zu testen. 

## Kleines Test-Framework für reguläre Ausdrücke

Am besten lässt sich der Ablauf anhand des folgenden Skripts erklären:

```python {linenos=true}
""" 
The various "doctest" tests that are performed on 
the regular expression "REGEX_UNDER_TEST". For 
this the auxiliary function test_regex is used

>>> test_regex('11.12.13.14')
(True, '11.12.13.14')

>>> test_regex("255.10.10.10")
(True, '255.10.10.10')

>>> test_regex("10.12.13")
(False, '')

>>> test_regex("256.10.10.10")
(False, '')

>>> test_regex("   10.10.10.10")
(False, '')

>>> test_regex("10.10.10.10    ")
(False, '')
"""

# The regular expression which should be testet.
REGEX_UNDER_TEST = "^((?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$"

import re

def test_regex(test_str: str) -> tuple[bool, str]:
    """
    This function helps to test the regular expression REGEX_UNDER_TEST.

    :param str test_str: The string against which the regular expression REGEX_UNDER_TEST is applied.

    :returns:
        - is_match: True if the regular expression matches the string.
        - match: The partial string that was found.
    """
    regex = re.compile(REGEX_UNDER_TEST)
    ret = regex.search(test_str)
    if ret is None:
        return False, ""
    
    return True, ret.string


if __name__ == "__main__":
    import doctest
    doctest.testmod()
```

Exemplarisch wird hier ein regulärer Ausdruck getestet, der Ip Adressen erkennt. Dies dient allerdings nur der Veranschaulichung der Arbeitsweise.

Um die Funktionsweise des Skriptes zu verstehen, ist die letzte Zeile des Skriptes, also der Aufruf `doctest.testmod()` entscheidend. Dieser sorgt dafür, dass ``doctest`` Modul das aufrufende Modul testet. Das bedeutet, alle Docstrings im Modul werden nach der Zeichenkette ``>>>`` durchsucht. Alles, was danach folgt, wird im Python Interpreter ausgeführt. Der Rückgabewert jedes dieser Funktionsaufrufe wird anschließend mit dem Text in der nächsten Zeile verglichen.

Hierzu ein Beispiel:

```python
>>> test_regex('11.12.13.14')
(True, '11.12.13.14')
```
Dieser Abschnitt ist aus dem vorherigen Skript entnommen. Die Funktion `test_regex` wird mit dem Argument `'11.12.13.14'` durch ``doctest`` aufgerufen. Es folgt der zuvor angesprochene Vergleich. Wenn das Ergebnis stimmt, geht `doctest` zum nächsten Test über. Findet ``doctest`` aber eine Abweichung, wird diese als Fehler auf der Konsole ausgegeben.

Um dies zu verdeutlichen, führen wir das obige Skript aus.
```
>> python test_regex.py
```

Zunächst passiert nichts. Das liegt daran, dass das `doctest` Modul bei erfolgreichen Tests eben einfach nichts auf der Konsole ausgibt.

{: .notice--info} 
**Information:** Wenn dir die Ausgabe zu knapp ist, kannst du auch die erfolgreich abgeschlossenen Tests anzeigen lassen. Dazu musst du das Argument `verbose=True` dem Aufruf `testmod` übergeben.

Aber was passiert, wenn wir einen Test absichtlich scheitern lassen? Ändern wir beispielsweise den String des ersten Tests auf `11.12.13.15` und führen das Skript erneut aus. 

```
>> python test_regex.py
**********************************************************************
File "./test_regex.py", line 6, in __main__
Failed example:
    test_regex('11.12.13.15')
Expected:
    (True, '11.12.13.14')
Got:
    (True, '11.12.13.15')
**********************************************************************
1 items had failures:
   1 of   6 in __main__
***Test Failed*** 1 failures.
```

Nun ist die Ausgabe etwas interessanter. Es wird angezeigt, dass einer unserer Tests fehlgeschlagen ist. Praktisch dabei ist, dass direkt untereinander der Soll- und Ist-Zustand des Tests ausgegeben wird. So können wir den Fehler leicht lokalisieren und beseitigen.

# Meine Vorgehensweise

Mithilfe des vorgestellten Templates kann man nun auch sehr komplexe reguläre Ausdrücke einfach und verlässlich testen. Meist gehe ich dabei nach folgender Vorgehensweise vor:

1. Kopiere die oben angegebene Vorlage in eine eigene Datei mit der Endung `.py`
2. Ersetze den regulären Ausdruck definiert in ``REGEX_UNDER_TEST`` durch den, den du testen oder entwickeln möchtest.
3. Lösche alle Tests, bis auf einen.
4. Passe diesen so an, dass er einen Aspekt deines regulären Ausdrucks testet und fehlerfrei abschließt.
5. Entwickle von hier aus weitere Tests oder passe den regulären Ausdruck an, bis du das gewünschte Endergebnis erreichst.

Abschließend habe ich mit dem so entwickelten Test-Skript auch eine gute Möglichkeit das Ergebnis zu dokumentieren und im jeweiligen Projekt abzulegen. 


# Zusammenfassung

In diesem Artikel haben wir uns mit dem Testen von regulären Ausdrücken in Python befasst. Wir haben gelernt, dass die Kombination der Python-Pakete [re](https://docs.python.org/3/library/re.html) und [doctest](https://docs.python.org/3/library/doctest.html) ein effektives Werkzeug für die Erstellung und das Testen von regulären Ausdrücken ist. Durch die Nutzung von doctest können wir unsere Tests direkt in unseren Skripten und Modulen integrieren und einfach ausführen. Mit diesem Ansatz können wir reguläre Ausdrücke zuverlässig erstellen und testen.
