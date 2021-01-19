<!--

author:   Sebastian Zug & André Dietrich & Fabian Bär
email:    sebastian.zug@informatik.tu-freiberg.de & andre.dietrich@informatik.tu-freiberg.de & fabian.baer@student.tu-freiberg.de
version:  0.0.4
language: de
narrator: Deutsch Female

import:  https://raw.githubusercontent.com/liascript-templates/plantUML/master/README.md
         https://github.com/LiaTemplates/Pyodide

mark: <span style="background-color: @0;
                                  display: flex;
                                  width: calc(100% + 32px);
                                  margin: -16px;
                                  padding: 6px 16px 6px 16px;
                                  ">@1</span>
red:  @mark(#FF888888,@0)
blue: @mark(#898AE3,@0)
gray: @mark(gray,@0)
-->

# Programmierung CPU

**TU Bergakademie Freiberg - Wintersemester 2020 / 21**

Link auf die aktuelle Vorlesung im Versionsmanagementsystem GitHub

[https://github.com/TUBAF-IfI-LiaScript/VL_EingebetteteSysteme/blob/12_RISC_CISC.md](https://github.com/TUBAF-IfI-LiaScript/VL_EingebetteteSysteme/blob/master/12_RISC_CISC.md)

Die interaktive Form ist unter [diesem Link](https://liascript.github.io/course/?https://raw.githubusercontent.com/TUBAF-IfI-LiaScript/VL_EingebetteteSysteme/master/12_RISC_CISC.md#1) zu finden


---------------------------------------------------------------------

** Fragen an die Veranstaltung**

+ Unter

<!--
style="width: 80%; min-width: 420px; max-width: 720px;"
-->
```ascii

                Abstraktionsebenen

           +----------------------------+ -.   
  Ebene 6  | Problemorientierte Sprache |  |     ╔═══════════════╗  
           +----------------------------+  |  ◀══║ HIER SIND WIR!║  
                                           ⎬     ╚═══════════════╝
           +----------------------------+  |  Anwendungssoftware
  Ebene 5  | Assemblersprache           |  |
           +----------------------------+ -.

           +----------------------------+
  Ebene 4  | Betriebssystem             |     Systemsoftware
           +----------------------------+

           +----------------------------+
  Ebene 3  | Istruktionsset             |     Maschinensprache
           +----------------------------+     

           +----------------------------+  -.
  Ebene 2  | Mikroarchitektur           |   |
           +----------------------------+   |
                                            ⎬ Automaten, Speicher, Logik
           +----------------------------+   |
  Ebene 1  | Digitale Logik             |   |
           +----------------------------+  -.

           +----------------------------+
  Ebene 0  | E-Technik, Physik          |     Analoge Phänomene
           +----------------------------+                                      .
```

---------------------------------------------------------------------

## Motivation

In der vorangegangenen Vorlesung sprachen wir insbesondere über die Erfassung
von digitalen Signalen. Eine Erfassung von analogen Werten ist allerdings notwendig, um Phänomene der Umgebung mit dem notwendigen Detailgrad beobachten zu können.  

![Bild](./images/14_ADC/Herausforderung.png)<!-- style="width: 85%; max-width: 1000px" -->

Dabei wird das zeit- und wertkontinuierliche Eingangssignal in eine zeit- und wertdiskrete Darstellung überführt.

Die Idee besteht darin einem Spannungswert einer Zahlenrepräsentation zuzuordnen, die einen Indexwert innerhalb eines beschränkten Spannungswert repräsentiert.

<!--
style="width: 80%; min-width: 420px; max-width: 720px;"
-->
```ascii

            minimaler                     maximaler
              Wert                           Wert

Analog        0V                   <- Wert -> 8V

               |-------------------------------|                               .

Digital        | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |        3 Bit Auflösung   

               |   0   |   2   |   3   |   4   |        2 Bit        

               |       0       |       1       |        1 Bit           
```

![Bild](./images/14_ADC/ADC_prinzip.png)<!-- style="width: 85%; max-width: 1000px" -->


Daraus ergibt sich die zentrale Gleichung für die Interpretation des Ausgaben eines Analog-Digital-Wandlers

$$
ADC = \frac{V_{in}}{V_{ref}} ADC_{res}
$$

oder

$$
\frac{ADC \cdot V_{ref}} {ADC_{res}} = V_{in}
$$

Für unser Beispiel aus der Grafik zuvor bedeutet die Ausgabe von "5" bei einem 3-Bit-Wandler, also

$$
\frac{5 \cdot 8V} {2^3} = 5 V
$$

Der potentielle Quantisierungsfehler `Q` beträgt

$$
Q = \frac{8V}{2^3} = 1V
$$

Dabei erfolgt die Wandlung in zwei generellen Schritten. Zunächst wird das Signal zeitlich diskretisiert und darauffolgend wertbezogenen gewandelt.

![Bild](./images/14_ADC/Wandlungsprozess.png)<!-- style="width: 85%; max-width: 1000px" -->

### Analog Komperator

Ein Komparator ist eine elektronische Schaltung, die zwei Spannungen vergleicht. Der Ausgang zeigt in binärer/digitaler Form an, welche der beiden Eingangsspannungen höher ist. Damit handelt es sich praktisch um einen 1-Bit-Analog-Digital-Umsetzer.

![Bild](./images/14_ADC/Op-amp_symbol.svg.png)<!-- style="width: 35%; max-width: 300px" -->[^1]
![Bild](./images/14_ADC/Comperator.png)<!-- style="width: 55%; max-width: 600px" -->

[^1]: Wikipedia, Autor Omegatron - Eigenes Werk, CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=983276

<!--
style="width: 80%; min-width: 420px; max-width: 720px;"
-->
```ascii
                   ^
 Ideales       U_a |------------+
 Verhalten         |            |
                   |            |
                   |            |             U_i
                   +------------+------------->
                   |            | U_ref
                   |            |
                   |            |
                   |............+------------                                                .

                   ^
 Reales        U_a |--------+
 Verhalten         |        !\  
                   |        ! \  
                   |        !  \ U_ref        U_i
                   +--------!---+------------->
                   |        !    \  !
                   |        !     \ !  
                   |        !      \!
                   |........!.......+-----------                                                .

                            Hysterese
```

Im AVR findet sich ein Komperator, der unterschiedliche Eingänge miteinander vergleichen kann:
Für "+" sind dies die `BANDGAP Reference` und der Eingang `AIN0` und für "-" der Pin `AIN1` sowie alle analogen Eingänge.  

![Bild](./images/14_ADC/AnalogCompAVR.png)<!-- style="width: 85%; max-width: 1000px" -->[^1]

[^13]: Firma Microchip, megaAVR® Data Sheet, Seite 243, [Link](http://ww1.microchip.com/downloads/en/DeviceDoc/ATmega48A-PA-88A-PA-168A-PA-328-P-DS-DS40002061A.pdf)

Die grundlegende Konfiguration erfolgt über die Konfiguration der Bits / Register :

| Bits                   | Register | Bedeutung                                              |
| ---------------------- | -------- | ------------------------------------------------------ |
| `ACBG`                 | `ACSR`   | Analog Comparator Bandgap Select                       |
| `ACME`                 | `ADCSRB` | Analog Comparator Multiplexer Enable - Bit im Register |
| `ADEN`                 | `ADCSRA` | Analog Digital Enable                                  |
| `MUX2`, `MUX1`, `MUX0` | `ADMUX`  | Mulitiplexer Analog input                              |

Dazu kommen noch weitere Parameterisierungen bezüglich der Interrupts, der Aktivitierung von Timerfunktionalität oder der Synchronisierung.

Weitere Erläuterungen finden Sie im Handbuch auf Seite

> **Aufgabe:** An welchen Pins eines Arduino Uno Boards müssen Analoge Eingänge angeschlossen werden, um die zwei Signale mit dem Komperator zu vergleichen. Nutzen Sie den Belegungsplan (Schematics) des Kontrollers, der unter [Link](https://store.arduino.cc/arduino-uno-rev3) zu finden ist.

Ein Beispiel für den Vergleich eines Infrarot Distanzsensors mit einem fest vorgegebenen Spannungswert findet sich im _Example_ Ordner der Veranstaltung.

![Bild](./images/14_ADC/ComperatorBeispiel.png)<!-- style="width: 85%; max-width: 1000px" -->

```cpp
#define F_CPU 16000000UL
#include <avr/io.h>

int main()
{
  ADCSRB = (1<<ACME);
  DDRB = (1<<PB5);

  while(1)
  {
    if (ACSR & (1<<ACO))/* Check ACO bit of ACSR register */
       PORTB &= ~(1 << PB5); /* Then turn OFF PB5 pin */
    else    /* If ACO bit is zero */
        PORTB = (1<<PB5); /* Turn ON PB5 pin */
  }
}
```

> **Einschränkung von Tinkercad** 2. The ANALOG COMPARE feature, which is a possible interrupt source on the mega328P, does not work at all (output compare result ACO in register ACSR does not seem to change, no matter what voltages are presented at pins 6,7).

> **Die Demo folgt am Donnerstag**

## Analog Digital Wandler

Voraussetzung für den Wandlungsprozess ist die Sequenzierung des Signals. Mit einer spezifischen Taktrate wird das kontinuierliche Signal erfasst.  

![Bild](./images/14_ADC/SampleAndHold.png)<!-- style="width: 85%; max-width: 1000px" -->

Wie sollte die Taktrate für die Messung denn gewählt werden?

```python   PrintSampleResults.py
import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import curve_fit

# Generierung der "Realen Messungen"
f = 40  # Hz
tmin = -0.3
tmax = 0.3
t = np.linspace(tmin, tmax, 400)
s = np.cos(2*np.pi*f*t)

# Abtastung mit einer Frequenz kleiner der Grenzfrequenz
T = 1/35.0
nmin = np.ceil((tmin) / T)
nmax = np.floor(tmax / T)
n = np.arange(nmin, nmax) * T
y = np.cos(2*np.pi*f*n)

# Fitting eines Cosinussignals anhand der Messungen
def test(x, a):
    return np.cos(a * x)

# Berechnung des Signalverlaufes mit der geschätzten Periodendauer
param, param_cov = curve_fit(test, n, y)
ans = np.cos(param[0]*t)

# Ausgabe
fig, ax = plt.subplots()
ax.plot(t, s)
ax.plot(n, y, '.', markersize=8)
ax.plot(t, ans, '--', color='red', label=f"Estimated Signal")
ax.grid(True, linestyle='-.')
ax.tick_params(labelcolor='r', labelsize='medium', width=3)

plt.show()

plot(fig) # <- this is required to plot the fig also on the LiaScript canvas
```
@Pyodide.eval

Wenn ein kontinuierliches Signal, das keine Frequenzkomponenten hat, die über einer Frequenz  $f_c$ liegen mit einer Häufigkeit von größer $2f_c$ abgetatstet wird, kann das Originalsignal aus den gewonnenen Punkten unverzerrt rekonstruiert werden.

Falls dieses Kriterium nicht eingehalten wird, entstehen nichtlineare Verzerrungen, die auch als Alias-Effekt bezeichnet werden (vgl. Python Beispiel).  Die untere Grenze für eine Alias-freie Abtastung wird auch als _Nyquist-Rate_ bezeichnet.  

Als Erklärung kann eine Darstellung des Signals im Frequenzbereich dienen:

![Bild](./images/14_ADC/Nyquist.png)<!-- style="width: 85%; max-width: 1000px" -->

### Flashwandler

![Bild](./images/14_ADC/FlashWandler.png)<!-- style="width: 85%; max-width: 1000px" -->

Vorteil

+ Hohe Geschwindigkeit

Nachteil

+ Energieverbrauch größer
+ Hardwareaufwand für höhere Auflösungen

https://www.youtube.com/watch?v=x7oPVWLD59Y

### Sequenzielle Wandler

Sequenzielle Wandler umgehen die Notwendigkeit mehrerer Komperatoren, in dem das Referenzsignal variabel erzeugt wird.

![Bild](./images/14_ADC/SequenziellerWandler.png)<!-- style="width: 85%; max-width: 1000px" -->

Dafür ist allerdings ein Digital-Analog-Wandler nötig, der die Vergleichsspannung ausgehend von einer digitalen Konfiguration vornimmt.

![Bild](./images/14_ADC/DAWandler.png)<!-- style="width: 85%; max-width: 1000px" -->

**Zählverfahren**

![Bild](./images/14_ADC/Zaehlverfahren.png)<!-- style="width: 85%; max-width: 1000px" -->

Vorteil

+ sehr hohe Auflösungen möglich
+ Schaltung einfach umsetzbar – kritisches Element DAC/Komperator

Nachteil

+ Variierende Wandlungsdauer
+ langsam (verglichen mit dem Flashwandler)

**Sukzessive Approximation/Wägeverfahren**

![Bild](./images/14_ADC/SukzessiveAppro.png)<!-- style="width: 85%; max-width: 1000px" -->

Vorteil

+ Gleiche Wandlungsdauer

## Herausforderungen bei der Wandlung




**Fehlertypen**

![Bild](./images/14_ADC/Fehler.png)<!-- style="width: 85%; max-width: 1000px" -->

+ Quantisierungsfehler sind bedingt durch die Auflösung des Wandlers
+ Offsetfehler ergeben sich aus einer Abweichung der Referenzspannung und führen zu einem konstanten Fehler.
+ Verstärkungsfehler im Analog-Digitalwandler wirken einen wertabhängigen Fehler.
+ Der Linearitätsfehler ist die Abweichung von der Geraden. Linearitätsfehler lassen sich nicht abgleichen.

> **Merke:** Die Fehlerparameter hängen in starkem Maße von der Konfiguration des Wandlers (Sample Frequenz, Arbeitsbreite, Umgebungstemperatur) ab!

![Bild](./images/14_ADC/Linearitätsfehler.png)<!-- style="width: 55%; max-width: 1000px" -->[^2]

[^2]: Firma Texas Instruments, Datenblatt AD-Wandler 8 Bit DIL-8, TLC0831, TLC0831IP

**Referenzspannung**

Bereitstellung einer hinreichend stabilen Referenzspannung

Diode als Referenzspannung

## Parameter eines Analog-Digital-Wandlers

+ Auflösung
+ Messdauer
+ Leistungsaufnahme
+ Stabilität der Referenzspannung
+ Unipolare/Bipolare Messungen
+ Zahl der Eingänge
+ Ausgangsinterfaces (parallele Pins, Bus)
+ Temperaturabhängigkeit und Rauschverhalten (Gain, Nicht-Linearität, Offset)

### Umsetzung im AVR

| Handbuch des Atmega328p                             | Bedeutung                                                                                       |
| --------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| 10-Bit Auflösung                                    |                                                                                                 |
| 0.5 LSB Integral Non-Linearity                      | maximale Abweichung zwischen der idealen und der eigentlichen analogen Signalverlauf am Wandler |
| +/- 2 LSB Absolute Genauigkeit                      | Summe der Fehler inklusive Quantisierungsfehler, Offset Fehler etc.  (worst case Situation)     |
| 13 - 260μs Conversion Time                          | Die Dauer der Wandlung hängt von der Auflösung und der der vorgegebenen Taktrate  ab.                                                                                               |
| Up to 76.9kSPS (Up to 15kSPS at Maximum Resolution) |                                                                                                 |
| 0 - V CC ADC Input Voltage Range                    |  Es sind keine negativen Spannungen möglich.                                                                                               |
| Temperature Sensor Input Channel                    |                                                                                                 |
| Sleep Mode Noise Canceler                                                    |    Reduzierung des Steuquellen durch einen "Sleepmode" für die CPU                                                                                             |

![Bild](./images/14_ADC/AVR_ADC.png)<!-- style="width: 75%; max-width: 1000px" -->[^3]

[^3]: Firma Microchip, megaAVR® Data Sheet, Seite 247, [Link](http://ww1.microchip.com/downloads/en/DeviceDoc/ATmega48A-PA-88A-PA-168A-PA-328-P-DS-DS40002061A.pdf)

**Trigger für den Wandlung**

Grundsätzlich sind 3 Modi für die Wandlung möglich:

+ Programmgetriggerte Ausführung der Wandlung
+ Kontinuierliche Wandlung
+ ereignisgetriebener Start

![Bild](./images/14_ADC/ADCTrigger.png)<!-- style="width: 75%; max-width: 1000px" -->[^3]

[^3]: Firma Microchip, megaAVR® Data Sheet, Seite 247, [Link](http://ww1.microchip.com/downloads/en/DeviceDoc/ATmega48A-PA-88A-PA-168A-PA-328-P-DS-DS40002061A.pdf)



## Beispiele

**Beispiel 1 - Lesen eines Analogen Distanzsensors**

Für das Beispiel wird der AtMega2560 verwendet, der eine interne Referenzspannung von 2.56 V anstatt der des AtMega328 von 1.1 V bereit stellt.

Die Bedeutung ergibt sich beim Blick ins Datenblatt des Sensors GP2D, dessen Maximalwertausgabewert liegt bei etwa 2.5V

```c
#ifndef F_CPU
#define F_CPU 16000000UL // 16 MHz clock speed
#endif

#include <avr/io.h>
#include <util/delay.h>

int readADC(int channel) {
  int i; int result = 0;
  // Den ADC aktivieren und Teilungsfaktor auf 64 stellen
  ADCSRA = (1<<ADEN) | (1<<ADPS2) | (1<<ADPS1);
  // Kanal des Multiplexers & Interne Referenzspannung (2,56 V)
  ADMUX = channel | (1<<REFS1) | (1<<REFS0);
  // Den ADC initialisieren und einen sog. Dummyreadout machen
  ADCSRA |= (1<<ADSC);
  while(ADCSRA & (1<<ADSC));
  //4 Leseoperationen
  for(i=0; i<4; i++) {
    ADCSRA |= (1<<ADSC);
    while(ADCSRA & (1<<ADSC)); // Auf Ergebnis warten...
    result += ADCW; }
  // ADC wieder deaktivieren
  ADCSRA &= ~(1<<ADEN);
  return result>>2;
}

int main(void)
{
  Serial.begin(9600);
  while (1) //infinite loop
  {
    int result = readADC(0);
    Serial.println(result);
    Serial.flush();
    _delay_ms(10); //1 second delay
  }
  return  0; // wird nie erreicht
}
```

**Beispiel 2 - Temperaturüberwachung des Controllers**

> _The temperature measurement is based on an on-chip temperature sensor that is coupled to a single ended ADC8 channel. Selecting the ADC8 channel by writing the MUX3...0 bits in ADMUX register to "1000" enables the temperature sensor. The internal 1.1V voltage reference must also be selected for the ADC voltage reference source in the temperature sensor measurement. When the temperature sensor is enabled, the ADC converter can be used in single conversion mode to measure the voltage over the temperature sensor._ Handbuch Seite 256

```c
#ifndef F_CPU
#define F_CPU 16000000UL // 16 MHz clock speed
#endif

#include <avr/io.h>
#include <util/delay.h>

double getTemp(void)
{
    unsigned int wADC;
    double t;

    // Set the internal reference and mux.
    ADMUX = (1<<REFS1) | (1<<REFS0) | (1<<MUX3));
    ADCSRA |= (1<<ADEN);  // enable the ADC

    // wait for voltages to become stable.
    delay(20);

    // Start the ADC
    ADCSRA |= (1<<ADSC);

    // Detect end-of-conversion
    while (ADCSRA & (1<<ADSC));

    // Reading register "ADCW" takes care of how to read ADCL and ADCH.
    wADC = ADCW;

    // The offset of 324.31 could be wrong. It is just an indication.
    t = (wADC - 324.31 ) / 1.22;

    // The returned temperature is in degrees Celsius.
    return (t);
}

int main(void)
{
  Serial.begin(9600);
  while (1)
  {
    Serial.println(getTemp());
    Serial.flush();
    _delay_ms(10); //1 second delay
  }
  return  0; // wird nie erreicht
}
```