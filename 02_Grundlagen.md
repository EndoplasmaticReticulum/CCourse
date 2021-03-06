<!--

author:   Sebastian Zug & André Dietrich
email:    zug@ovgu.de   & andre.dietrich@ovgu.de
version:  0.0.1
language: de
narrator: Deutsch Female

script:   https://felixhao28.github.io/JSCPP/dist/JSCPP.es5.min.js

@JSCPP.__eval
<script>
  try {
    var output = "";
    JSCPP.run(`@0`, `@1`, {stdio: {write: s => { output += s }}});
    output;
  } catch (msg) {
    var error = new LiaError(msg, 1);

    try {
        var log = msg.match(/(.*)\nline (\d+) \(column (\d+)\):.*\n.*\n(.*)/);
        var info = log[1] + " " + log[4];

        if (info.length > 80)
          info = info.substring(0,76) + "..."

        error.add_detail(0, info, "error", log[2]-1, log[3]);
    } catch(e) {}

    throw error;
    }
</script>
@end


@JSCPP.eval: @JSCPP.__eval(@input, )

@JSCPP.eval_input: @JSCPP.__eval(@input,`@input(1)`)

@output: <pre class="lia-code-stdout">@0</pre>

-->

# Vorlesung II - Grundlagen der Sprache C

**Fragen an die heutige Veranstaltung ...**

* Durch welche Parameter ist eine Variable definiert?
* Erklären Sie die Begriffe Deklaration, Definition und Initialisierung im
  Hinblick auf eine Variable!
* Worin unterscheidet sich die Zahldarstellung von ganzen Zahlen (`int`) von den
  Gleitkommazahlen (`float`).
* Welche Datentypen kennt die Sprache C?
* Erläutern Sie für `char` und `int` welche maximalen und minimalen Zahlenwerte
  sich damit angeben lassen.
* Ist `printf` ein Schlüsselwort der Programmiersprache C?
* Welche Beschränkung hat `getchar`

**Vorwarnung:** Man kann Variablen nicht ohne Ausgaben und Ausgaben nicht ohne
Variablen erklären. Deshalb wird im folgenden immer wieder auf einzelne Aspekte
vorgegriffen. Nach der Vorlesung sollte sich dann aber ein Gesamtbild ergeben.

## 0. Zeichensätze

Sie können in einem C-Programm folgende Zeichen verwenden:

* Dezimalziffern

  `0 1 2 3 4 5 6 7 8 9`

* Buchstaben des englischen Alphabets

  `A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z`

* Grafiksymbole

  `!"%&/()[]{}\?='#+*~-_.:,`

  ```cpp                     SpecialSigns.c
  #include <stdio.h>

  int main() {
      printf("Einen schönen Tag für Sie!");
      return 0;
  }
  ```
  @JSCPP.eval

![C logo](img/PellesCUmlauteEnglish.png)<!-- width="100%" -->


## 1. Variablen

> Ein Rechner ist eigentlich ziemlich dumm, dass aber viele Millionen mal pro
> Sekunde (Zitat - Quelle gesucht!)

```cpp                     Calculator.c
#include<stdio.h>

int main() {
  printf("%d",43 + 17);
	return 0;
}
```
@JSCPP.eval

Unbefriedigenden Lösung, jede neue Berechnung muss in den Source-Code integriert
und dieser dann kompiliert werden. Ein Taschenrechner wäre die bessere Lösung!

```cpp                     QuadraticEquation.c
#include<stdio.h>

int main() {
  int x;
  x = 5;
  printf("f(%d) = %d \n",x, 3*x*x + 4*x + 8);
  x = 9;
  printf("f(%d) = %d ",x, 3*x*x + 4*x + 8);
	return 0;
}
```
@JSCPP.eval

Ein Programm manipuliert Daten, die in Variablen organisiert werden.

Eine Variable ist somit ein **abstrakter Behälter** für Inhalte, welche im
Verlauf eines Rechenprozesses benutzt werden. Im Normalfall wird eine Variable
im Quelltext durch einen Namen bezeichnet, der die Adresse im Speicher
repräsentiert.

Kennzeichen einer Variable:

1. Name
2. Datentyp
3. Wert
4. Adresse
5. Gültigkeitraum


### Zulässige Variablennamen

Der Name kann Zeichen, Ziffern und den Unterstrich enthalten. Dabei ist zu
beachten:

* Das erste Zeichen muss ein Buchstabe sein, der Unterstrich ist auch möglich.
* C betrachte Groß- und Kleinschreibung - `Zahl` und `zahl` sind also
  unterschiedliche Variablentypen
* Schlüsselworte (`auto`, `goto`, `return`, etc.) sind als Namen unzulässig.

| Name            | Zulässigkeit                                   |
|:----------------|:-----------------------------------------------|
| `gcc`           | erlaubt                                        |
| `a234a_xwd3`    | erlaubt                                        |
| `speed_m_per_s` | erlaubt                                        |
| `double`        | nicht zulässig (Schlüsselwort)                 |
| `robot.speed`   | nicht zulässig (`.` im Namen)                  |
| `3thName`       | nicht zulässig (Ziffer als erstes Zeichen)     |
| `x y`           | nicht zulässig (Leerzeichen im Variablennamen) |

Das ganze kann dann noch in entsprechenden Notationen verpackt werden, um einem
Wildwuchs an Bezeichnern vorzubeugen.

| Bezeichnung   | denkbare Variablennamen                            |
|:--------------|:---------------------------------------------------|
| CamelCase     | `youLikeCamelCase`, `humanDetectionSuccessfull`    |
| underscores   | `I_hate_Camel_Case`, `human_detection_successfull` |

Verschiedene Konvention geht noch einen Schritt weiter und verknüpft Datentypen
(an erster Stelle) und Funktion mit dem Namen (hier abgewandelte
*ungarische Notation*).

| Präfix | Datentyp                      | Beispiel     |
|:-------|:------------------------------|:-------------|
| n      | Integer                       | `nSize`      |
| b      | Boolean                       | `bBusy`      |
| sz     | null-terminierter String      | `szLastName` |
| p      | Zeiger                        | `pMemory`    |
| a      | Array                         | `aCounter`   |
| ch     | char                          | `chName`     |
| dw     | Double Word, 32 Bit, unsigned | `dwNumber`   |
| w      | Word, 16 Bit, unsigned        | `wNumber`    |

> Encoding the type of a function into the name (so-called Hungarian notation)
> is brain damaged – the compiler knows the types anyway and can check those,
> and it only confuses the programmer.
>
> (Linus Torvalds)

### Datentypen

Welche Informationen lassen sich mit Blick auf einen Speicherauszug im Hinblick
auf die Daten extrahieren?

    {{0-1}}
| Adresse | Speicherinhalt |
|         | binär          |
| 0010    | 0000 1100      |
| 0011    | 1111 1101      |
| 0012    | 0001 0000      |
| 0013    | 1000 0000      |

    {{1-2}}
| Adresse | Speicherinhalt | Zahlenwert |
|         |                |  (Byte)    |
| 0010    | 0000 1100      | 12         |
| 0011    | 1111 1101      | 253 (-125) |
| 0012    | 0001 0000      | 16         |
| 0013    | 1000 0000      | 128 (-128) |

    {{2}}
| Adresse | Speicherinhalt | Zahlenwert | Zahlenwert | Zahlenwert   |
|         |                |  (Byte)    | (2 Byte)   | (4 Byte)     |
| 0010    | 0000 1100      | 12         |            |              |
| 0011    | 1111 1101      | 253 (-125) | 3325       |              |
| 0012    | 0001 0000      | 16         |            |              |
| 0013    | 1000 0000      | 128 (-128) | 4224       | 217911424    |


<!-- hidden = "true" -->
| Adresse | Speicherinhalt | Zahlenwert | Zahlenwert | Zahlenwert   |
|:--------|:---------------|:-----------|:-----------|:-------------|
|         |                |  (Byte)    | (2 Byte)   | (4 Byte)     |
| 0010    | 0000 1100      | 12         |            |              |
| 0011    | 1111 1101      | 253 (-125) | 3325       |              |
| 0012    | 0001 0000      | 16         |            |              |
| 0013    | 1000 0000      | 128 (-128) | 4224       | 217911424    |


    {{3}}
Der dargestellte Speicherauszug kann aber auch eine Kommazahl (Floating Point)
umfassen und repräsentiert dann den Wert `3.8990753E-31`

    {{4}}
Folglich bedarf es eines expliziten Wissens um den Charakter der Zahl, um eine
korrekte Interpretation zu ermöglichen. Dabei erfolgt die Einteilung nach:

    {{4}}
* Wertebereichen (größte und kleinste Zahl)
* ggf. vorzeichenbehaftet Zahlen
* ggf. gebrochene Werte

#### Generische Datentypen

Ganzzahlen sind Zahlen ohne Nachkommastellen mit und ohne Vorzeichen. In C gibt es folgende Typen für Ganzzahlen:

| Schlüsselwort    | Benutzung                       | Mindestgröße       |
|:-----------------|:--------------------------------|:-------------------|
| `char`           | 1 Byte bzw. 1 Zeichen           | 1 Byte (min/max)   |
| `short int`      | Ganzahl (ggf. mit Vorzeichen)   | 2 Byte             |
| `int`            | Ganzahl (ggf. mit Vorzeichen)   | "natürliche Größe" |
| `long int`       | Ganzahl (ggf. mit Vorzeichen)   |                    |
| `long long int`  | Ganzahl (ggf. mit Vorzeichen)   |                    |
| `float`          | Gebrochener Wert mit Vorzeichen | 4 Byte             |
| `double`         | Gebrochener Wert mit Vorzeichen | 8 Byte             |
| `long double`    | Gebrochener Wert mit Vorzeichen |                    |
| `_Bool`          | boolsche Variable               | 1 Bit              |

``` c
signed char <= short <= int <= long <= long long
```

Gängige Zuschnitte für `char` oder `int`

| Schlüsselwort    | Wertebereich               |
|:-----------------|:---------------------------|
| `signed char`    | -128 bis 127               |
| `char`           | 0 bis 255 (0xFF)           |
| `signed int`     | 32768 bis 32767            |
| `int`            | 65536 (0xFFFF)             |

Wenn die Typ-Spezifizierer (`long` oder `short`) vorhanden sind kann auf die
`int` Typangabe verzichtet werden.

```c
short int a; // entspricht short a;
long int b;  // äquivalent zu long b;
```

Standardmäßig wird von vorzeichenbehafteten Zahlenwerten ausgegangen. Somit wird das Schlüsselwort `signed` eigentliche nicht benötigt

```cpp
int a;  //  signed int a;
unsigned long long int b;
```

#### Sonderfall `char`

Für den Typ `char` ist der mögliche Gebrauch und damit auch die Vorzeichenregel
zwiespältig:

* Wird `char` dazu verwendet einen **numerischen Wert** zu speichern und die
  Variable nicht explizit als vorzeichenbehaftet oder vorzeichenlos vereinbart,
  dann ist es implementierungsabhängig, ob `char` vorzeichenbehaftet ist oder
  nicht.
* Wenn ein Zeichen gespeichert wird, so garantiert der Standard, dass der
  gespeicherte Wert der nicht negativen Codierung im **Zeichensatz** entspricht.

```cpp
char c = 'M';  // =  1001101 (ASCII Zeichensatz)
char c = 77;   // =  1001101
char s[] = "Eine kurze Zeichenkette";
```

> **Achtung:** Anders als bei anderen Programmiersprachen unterscheidet C
> zwischen den verschiedenen Anführungsstrichen.

![C logo](img/ASCII_Zeichensatz.jpeg)<!-- style="width: 80%; display: block; margin-left: auto; margin-right: auto;" -->

http://www.chip.de/webapps/ASCII-Tabelle_50073950.html

    --{{1}}--
Erweiterung erfährt `char` mit der Überarbeitung des C-Standards 1994. Hier
wurde das Konzept eines breiten Zeichens (engl. *wide character*) eingeführt,
das auch Zeichensätze aufnehmen kann, die mehr als 1 Byte für die Codierung
eines Zeichen benötigen (beispielsweise Unicode-Zeichen). Siehe `wchar_t` oder
`wprintf`.


#### Architekturspezifische Ausprägung

Der Operator `sizeof` gibt Auskunft über die Größe eines Datentyps oder einer
Variablen in Byte.

**Integer Datentypen**

```cpp                     sizeof.c
#include <stdio.h>

int main()
{
  int x;
  printf("x umfasst %d Byte.", (unsigned int)sizeof x);
  return 0;
}
```
@JSCPP.eval


```cpp                     sizeof_example.c
#include <stdio.h>
#include <limits.h>   /* INT_MIN und INT_MAX */

int main() {
   printf("int size : %d Byte\n", (unsigned int) sizeof( int ) );
   printf("Wertebereich von %d bis %d\n", INT_MIN, INT_MAX);
   printf("char size : %d Byte\n", (unsigned int) sizeof( char ) );
   printf("Wertebereich von %d bis %d\n", CHAR_MIN, CHAR_MAX);
   return 0;
}
```

```bash @output
▶ gcc sizeof_example.c
▶ ./a.out
int size : 4 Byte
Wertebereich von -2147483648 bis 2147483647
char size : 1 Byte
Wertebereich von -128 bis 127
```

**Fließkommazahlen**

Zur Darstellung von Fließkommazahlen sagt der C-Standard nichts aus. Zur
konkreten Realisierung ist die Headerdatei `float.h` auszuwerten.

Im Gegensatz zu Ganzzahlen gibt es bei den Fließkommazahlen keinen Unterschied
zwischen vorzeichenbehafteten und vorzeichenlosen Zahlen. Alle Fließkommazahlen
sind in C immer vorzeichenbehaftet.

| Schlüsselwort    | Wertebereich                     |
|:-----------------|:---------------------------------|
| `float`          | größte Zahl 3.4028234664e+38     |
| `char`           | kleinste Zahl 1.1754943508e-38   |

> **ACHTUNG:** Fließkommazahlen bringen neben den Maximal und Minimalwerten noch
> einen weiteren Faktor mit - die Unsicherheit

```cpp                     float_precision.c
#include<stdio.h>
#include<float.h>

int main() {
	printf("float Genauigkeit  :%d \n", FLT_DIG);
	printf("double Genauigkeit :%d \n", DBL_DIG);
	float x = 0.1;
  if (x == 0.1) {
    printf("Gleich\n");
  }else{
    printf("Ungleich\n");
  }
  return 0;
}
```

``` bash @output
▶ gcc float_comparism.c
▶ ./a.out
float Genauigkeit :6
double Genauigkeit :15
Ungleich
```


#### Spezifische Datentypen

Ab C99 umfasst die Standard Bibliothek das header file `stdint.h`, das
Ganzzahldatentypen mit exakten bzw. spezifizierten Datenbreiten einführt. Dabei
können die individuellen Realisierungen für verschiedene Compiler auf ein und
der selben Architektur durchaus variieren!

| signed         | unsigned        | Bedeutung                            |
|:---------------|:----------------|:-------------------------------------|
| `intN_t`       | `uintN_t`       | exakte Breite von N Bits (`uint8_t`) |
| `int_leastN_t` | `uint_leastN_t` | garantierte Mindestbreite            |
| `int_fastN_t`  | `uint_fastN_t`  | fokussiert die Ausführungszeit       |


#### Und was soll ich nun verwenden?

... das hängt von der Zielstellung ab. Mögliche Optimierungsansätze sind

* die Programmgröße
* der notwendige Arbeitsspeicher
* die Laufzeit
* (die Compilezeit)

TODO: Link auf measureExecutionTime.py

``` bash @output
▶ python measureExecutionTime.py

                           execution_mean
                    -O0     -O1     -O2    -Os
data_type
uint16_t           3.5170  2.4215  0.7340  2.0100
uint32_t           2.4725  2.4550  0.9025  2.2945
uint_fast16_t      3.5195  2.6060  0.7310  2.6415
unsigned int       3.8970  1.9680  0.9010  2.4775
unsigned long      4.0015  2.2040  0.6910  2.3460
unsigned short     2.9580  2.1010  0.7495  2.3240
```

### Wertspezifikation

Zahlenliterale können in C mehr als Ziffern umfassen!

|  Zahlentyp | Dezimal       | Oktal         | Hexadezimal  |
|:-----------|:--------------|---------------|--------------|
| Eingabe    | x             | x             | x            |
| Ausgabe    | x             | x             | x            |
| Beispiel   | `12`          | `020`         | `0x20`       |
|            | `1234.342`    |               | `0X1a`       |

```cpp                     NumberFormats.c
#include <stdio.h>

int main()
{
  int x=020;
  int y=0x20;
  printf("x = %d\n", x);
  printf("y = %d\n", y);
  printf("Rechnen mit Oct und Hex z = %d", x + y);
  return 0;
}
```
@JSCPP.eval

### Adressen

Um einen Zugriff auf die Adresse einer Variablen zu haben, kann man den Operator `&`
nutzen. Gegenwärtig ist noch nicht klar, warum dieser Zugriff erforderlich ist,
wird aber in einer der nächsten Veranstaltungen verdeutlicht.

```cpp                     Pointer.c
#include <stdio.h>

int main()
{
  int x=020;
  //              cast operator
  //               |  Adressoperator
  //               |   |
  printf("%p\n",(void*)&x);
  return 0;
}
```

``` bash @output
▶ ./a.out
0x7ffef853c3f4
```

### Sichtbarkeit und Lebensdauer von Variablen

**Lebensdauer**

Die Lebensdauer einer Variablen gilt immer bis zum Ende des Anweisungsblockes,
in dem sie definiert wurde.

```cpp                           lifespan.c
#include<stdio.h>

int main(){
  {
    int v;
    v = 2;
    printf("%d", v);
  }
  printf("%d", v);
  return 0;
}
```
@JSCPP.eval

**Sichtbarkeit**

Bis C99 musste eine Variable immer am Anfang eines Anweisungsblocks vereinbart
werden. Nun genügt es, die Variable unmittelbar vor der ersten Benutzung zu
vereinbaren.

Variablen *leben* innerhalb einer Funktion, eine Schleife oder einfach nur ein
durch geschwungene Klammern begrenzter Block von Anweisungen.

Wird eine Variable/Konstante z. B. im Kopf einer Schleife vereinbart, gehört sie
laut C99-Standard zu dem Block, in dem auch der Code der Schleife steht.
Folgender Codeausschnitt soll das verdeutlichen:

```cpp                           visibility.c
#include<stdio.h>

int main()
{
  int v = 1;
  int w = 5;
  {
    int v;
    v = 2;
    printf("%d\n", v);
    printf("%d\n", w);
  }
  printf("%d\n", v);
  return 0;
}
```
@JSCPP.eval

**Globale Variablen**

Muss eine Variable immer innerhalb von `main` definiert werden? Nein, allerdings
sollten globale Variablen vermieden werden.

```cpp                           visibility.c
#include<stdio.h>

int main()
{
  int v = 1;
  printf("%d\n", v);
  return 0;
}
```
@JSCPP.eval


### Definition vs. Deklaration vs. Initialisierung

... oder andere Frage, wie kommen Name, Datentyp, Adresse usw. zusammen?

> Deklaration ist nur die Vergabe eines Namens und eines Typs für die Variable.
> Definition ist die Reservierung des Speicherplatzes. Initialisierung ist die
> Zuweisung eines ersten Wertes.

**Merke:** Jede Definition ist gleichzeitig eine Deklaration aber nicht umgekehrt!

```cpp                     DeclarationVSDefinition.c
extern int a;      // Deklaration
int i;             // Definition + Deklaration
int a,b,c;
i = 5;             // Initialisierung
```

Das Schlüsselwort `extern` in obigem Beispiel besagt, dass die Definition der
Variablen a irgendwo in einem anderen Modul des Programms liegt. So deklariert
man Variablen, die später beim Binden (Linken) aufgelöst werden. Da in diesem
Fall kein Speicherplatz reserviert wurde, handelt es sich um keine Definition.

### Typische Fehler

**Fehlende Initialisierung**

```cpp                     MissingInitialisation.c
#include<stdio.h>

void foo() {
  int a;     // <- Fehlende Initialisierung dynamische Variable
  printf("a=%d ", a);
}

int main() {
  int x = 5;
  printf("x=%d ", x);
  int y;     // <- Fehlende Initialisierung statische Variable
  printf("y=%d ", y);
  foo();
	return 0;
}
```
@JSCPP.eval

``` bash @output
▶ gcc -Wall experiments.c
experiments.c: In function ‘foo’:
experiments.c:5:3: warning: ‘a’ is used uninitialized in this function [-Wuninitialized]
   printf("a=%d ", a);
   ^
experiments.c: In function ‘main’:
experiments.c:12:3: warning: ‘y’ is used uninitialized in this function [-Wuninitialized]
   printf("y=%d ", y);
   ^
```

    {{0}}
Der C++, der für diese Webseite zum Einsatz kommt initialisiert offenbar alle
Werte mit 0 führen Sie dieses Beispiel aber einmal mit einem richtigen Compiler
aus.


**Redeklaration**

```cpp                     Redeclaration.c
#include<stdio.h>

int main() {
  int x;
  int x;
  return 0;
}
```

``` bash @output
▶ gcc -Wall doubleDeclaration.c

doubleDeclaration.c: In function ‘main’:
doubleDeclaration.c:5:7: error: redeclaration of ‘x’ with no linkage
   int x;
       ^
doubleDeclaration.c:4:7: note: previous declaration of ‘x’ was here
   int x;
```

**Falsche Zahlenliterale**

```cpp                     wrong_float.c
#include<stdio.h>

int main() {
  float a=1,5;   /* FALSCH  */
  float b=1.5;   /* RICHTIG */
  return 0;
}
```

**Was passiert wenn der WERT zu groß ist?**

```cpp                     TooLarge.c
#include<stdio.h>

int main() {
  short a;
  a = 0xFFFF + 2;
  printf("Schaun wir mal ... %hi\n", a);
	return 0;
}
```

``` bash @output
▶ gcc -Wall experiments.c
experiments.c: In function ‘main’:
experiments.c:5:7: warning: overflow in implicit constant conversion [-Woverflow]
   a = 0xFFFF + 2;
       ^
▶ ./a.out
Was steckt drin 1
```

## 2. Input-/ Output Operationen

Ausgabefunktionen wurden bisher genutzt, um den Status unserer Programme zu
dokumentieren. Nun soll dieser Mechanismus systematisiert und erweitert werden
[^1].

![C logo](img/EVA-Prinzip.png)<!-- width="100%" -->

[^1]: Programmiervorgang und Begriffe (Quelle:
      https://de.wikipedia.org/wiki/EVA-Prinzip#/media/File:EVA-Prinzip.svg
      (Autor Deadlyhappen))

### `printf`

Die Bibliotheksfunktion `printf` dient dazu, eine Zeichenkette (engl. *String*)
auf der Standardausgabe auszugeben. In der Regel ist die Standardausgabe der
Bildschirm. Über den Rückgabewert liefert `printf` die Anzahl der ausgegebenen
Zeichen. Wenn bei der Ausgabe ein Fehler aufgetreten ist, wird ein negativer
Wert zurückgegeben.

Als erstes Argument von `printf` sind **nur Strings** erlaubt. Bei folgender
Zeile gibt der Compiler beim Übersetzen deshalb eine Warnung oder einen Fehler
aus:

```c
printf(55);      // Falsch
printf("55");    // Korrekt
```

Dabei kann der String um entsprechende Parameter erweitert werden. Jeder Parameter nach dem String wird durch einen Platzhalter in dem selben repäsentiert.

```cpp                             printf_example.c
#include <stdio.h>

int main(){
  //       ______________________________
  //       |                            |
  printf("%i plus %2.2f ist gleich %s.\n", 3, 2.0, "Fuenf");
  //      <------------------------>
  //        String mit Referenzen
  //            auf Parameter
  return 0;
}
```
@JSCPP.eval

                                 {{1-3}}
| Zeichen        | Umwandlung                                              |
|:---------------|:--------------------------------------------------------|
| `%d` oder `%i` | `int`                                                   |
| `%c`           | einzelnes Zeichen                                       |
| `%e` oder `%E` | `double` im Format `[-]d.ddd e±dd` bzw. `[-]d.ddd E±dd` |
| `%f`           | `double` im Format `[-]ddd.ddd`                         |
| `%o`           | `int` als Oktalzahl ausgeben                            |
| `%p`           | die Adresse eines Zeigers                               |
| `%s`           | Zeichenkette ausgeben                                   |
| `%u`           | `unsigned int`                                          |
| `%x` oder `%X` | `int` als Hexadezimalzahl ausgeben                      |
| `%%`           | Prozentzeichen                                          |


    {{2-3}}
Welche Formatierungmöglichkeiten bietet `printf` noch?

    {{2-3}}
+ die Feldbreite
+ ein Flag
+ durch einen Punkt getrennt die Anzahl der Nachkommstellen (Längenangabe) und
  an letzter Stelle schließlich
+ das Umwandlungszeichen selbst (siehe Tabelle oben)

#### Feldbreite

Die Feldbreite definiert die Anzahl der nutzbaren Zeichen, sofern diese nicht
einen größeren Platz beanspruchen.

```cpp                             printf_numbercount.c
#include <stdio.h>

int main(){
  printf("rechtsbuendig      : %5d, %5d, %5d\n",34, 343, 3343);
  printf("linksbuendig       : %-5d, %-5d, %-5d\n",34, 343, 3343);
  printf("Zu klein gedacht   : %5d\n", 234534535);
  //printf("Ohnehin besser     : %*d\n", 12, 234534535);
  return 0;
}
```
@JSCPP.eval

#### Formatierungsflags

| Flag              | Bedeutung                           |
|:------------------|:------------------------------------|
| `-`               | linksbündig justieren               |
| `+`               | Vorzeichen ausgeben                 |
| Leerzeichen       | Leerzeichen                         |
| `0`               | numerische Ausgabe mit 0 aufgefüllt |
| `#`               | für `%x` wird ein x in die Hex-Zahl eingefügt, für `%e` oder `%f` ein Dezimaltrenner (.) |

```cpp                             printf_flag_examples.c
#include <stdio.h>

int main(){
  printf("012345678901234567890\n");
  printf("Integer: %+11d\n", 42);
	printf("Integer: %+11d\n", -4242);
  printf("Integer: %-e\n", 42.2345);
  printf("Integer: %-g\n", 42.2345);
  printf("Integer: %#x\n", 42);
  return 0;
}
```
@JSCPP.eval

#### Genauigkeit

 Bei %f werden ansonsten standardmäßig 6 Nachkommastellen ausgegeben. Mit %a.bf kann eine genauere Spezifikation erfolgen. a steht für die Feldbreite, b für die Nachkommastellen

```cpp                             printf_flag_examples.c
#include <stdio.h>

int main(){
  float a = 142.23443513452352;
  printf("Float Wert: %f\n", a);
  printf("Float Wert: %9.3f\n", a);
  return 0;
}
```
@JSCPP.eval

#### Escape Sequenzen

| Sequenz | Bedeutung             |
|:--------|:----------------------|
| `\n`    | newline               |
| `\b`    | backspace             |
| `\r`    | carriage Return       |
| `\t`    | horizontal tab        |
| `\\\`   | Backslash             |
| `\'`    | Single quotation mark |
| `\"`    | Double quotation mark |

```cpp                             printf_esc_sequences.c
#include <stdio.h>

int main(){
  printf("123456789\r");
  printf("ABCD\n\n");
	printf("Vorname \t Name \t\t Alter \n");
	printf("Andreas \t Mustermann\t 42 \n\n");
	printf("Manchmal braucht man auch ein \"\\\"\n");
  return 0;
}
```
@JSCPP.eval

### `getchar`

Die Funktion `getchar()` liest Zeichen für Zeichen aus einem Puffer. Dies
geschieht aber erst, wenn die Enter-Taste gedrückt wird.

```cpp                     GetChar.c
#include <stdio.h>

int main(){
	char c;
	printf("Mit welchem Buchstaben beginnt ihr Vorname? ");
	c = getchar();
	printf("\nIch weiss jetzt, dass Ihr Vorname mit '%c' beginnt.\n", c);
	return 0;
}
```
``` text                  stdin
T
```
@JSCPP.eval_input

Problem: Es lassen sich immer nur einzelne Zeichen erfassen, die durch `Enter`
bestätigt wurden. Als flexiblere Lösung lässt sich `scanf` anwenden.

### `scanf`

```cpp                     scanf_getNumbers.c
#include <stdio.h>

int main(){
  int i;
  float a;
  char b;
  printf("Bitte folgende Werte eingeben %%c %%f %%d: ");
  scanf(" %c %f %4d", &b, &a, &i);
  printf("%c %f %d\n", b, a, i);
  return 0;
}
```

``` bash @output
▶ gcc sizeof_example.c
▶ ./a.out
Bitte folgende Werte eingeben %c %f %d: A -234.24324 234562
A -234.243240 2345
```


## Ausblick

```cpp                     GoodBy.c
#include<stdio.h>

int main() {
  printf("... \t bis \n\t\t zum \n\t\t\t");
  printf("naechsten \n\t\t\t\t\t mal! \n");
	return 0;
}
```
@JSCPP.eval
