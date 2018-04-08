Die Codes können z.B. mit dem ConnAir Home Automation Gateway von Simple Solutions getestet werden.

  $ echo $CODE | nc -w 1 -u <ip> 49880 
  
## REV ##
Hersteller: REV-Ritter GmbH

**Übersetzung**

```
0 = 1,3,1,3,
1 = 3,1,3,1,
F = 1,3,3,1,
```

**Beginn:**

`TXP:0,0,10,5600,350,25,`

**für Master:**

```
A = 1FFF
B = F1FF
C = FF1F
D = FFF1
```

**für Slave:**

```
1 = 1FF
2 = F1F
3 = FF1
```

**fester Zwischenteil:**

`0FF`

**On** `FF` oder **Off** `00`

**Abschluss:**
`1,16;`

[Quelle](http://simple-solutions.de/forum/viewtopic.php?p=4072#p4072)

### REV Typ 8342L ###
**B1**

ON: `TXP:0,0,10,5600,350,25,1,3,3,1,3,1,3,1,1,3,3,1,1,3,3,1,3,1,3,1,1,3,3,1,1,3,3,1,1,3,1,3,1,3,3,1,1,3,3,1,1,3,3,1,1,3,3,1,1,16;`

OFF: `TXP:0,0,10,5600,350,25,1,3,3,1,3,1,3,1,1,3,3,1,1,3,3,1,3,1,3,1,1,3,3,1,1,3,3,1,1,3,1,3,1,3,3,1,1,3,3,1,1,3,1,3,1,3,1,3,1,16;`

**B2**

ON: `TXP:0,0,10,5600,350,25,1,3,3,1,3,1,3,1,1,3,3,1,1,3,3,1,1,3,3,1,3,1,3,1,1,3,3,1,1,3,1,3,1,3,3,1,1,3,3,1,1,3,3,1,1,3,3,1,1,16;`

OFF: `TXP:0,0,10,5600,350,25,1,3,3,1,3,1,3,1,1,3,3,1,1,3,3,1,1,3,3,1,3,1,3,1,1,3,3,1,1,3,1,3,1,3,3,1,1,3,3,1,1,3,1,3,1,3,1,3,1,16;`

**B3**

ON: `TXP:0,0,10,5600,350,25,1,3,3,1,3,1,3,1,1,3,3,1,1,3,3,1,1,3,3,1,1,3,3,1,3,1,3,1,1,3,1,3,1,3,3,1,1,3,3,1,1,3,3,1,1,3,3,1,1,16;`

OFF: `TXP:0,0,10,5600,350,25,1,3,3,1,3,1,3,1,1,3,3,1,1,3,3,1,1,3,3,1,1,3,3,1,3,1,3,1,1,3,1,3,1,3,3,1,1,3,3,1,1,3,1,3,1,3,1,3,1,16;`

## Quigg ##
Hersteller: Globaltronics GmbH

Weitere Modelle:

- GT-7008BS
- GT-FSI-04
- DMV-7008S 
- Powerfix RCB-I 3600

Diese Dosen können einfach angelernt werden. Dazu die Lernen-Taste gedrückt halten und ein ON-Signal senden.

Das Protokoll sieht so aus:
Die Taktfrequenz beträgt 1.5 kHz. Ein Bit dauert 3 Takte.
1-Bit: 2 Takte aus 1 Takt ein.
0-Bit: 1 Takt aus 2 Takte ein.

Die Sequenz besteht aus einer 1 als Startbit gefolgt von 12 Bit Hauscode und 8 Bit Funktionscode:

```
#1 EIN 00010001
#1 AUS 00000000
#1 HELL 00001010
#1 DUNKEL 00011011
#2 EIN 10010011
#2 AUS 10000010
#2 HELL 10001000
#2 DUNKEL 10011001
#3 EIN 11010010
#3 AUS 11000011
#3 HELL 11001001
#3 DUNKEL 11011000
#4 EIN 01010000
#4 AUS 01000001
#4 HELL 01001011
#4 DUNKEL 01011010
ALLE EIN 11110000
ALLE AUS 11100001
```

Es gibt 4 Wiederholungen mit einer Pause von je 80ms, allerdings kann das Conn Air anscheinend keine Pausen über 65535us.
Dieses Script jedenfalls erzeugt eine Sequenz, die mit diesen Dosen funktioniert:

```
#!/bin/bash
CODE=`echo $* | awk '{  
  cnt=0
  for (i=1; i <= length($0); ++i)
  {
    c = substr($0, i, 1)
    if (c=="0" || c=='1')
      ++cnt
  }
  printf("TXP:0,0,5,65535,667,%d,1,",cnt+1)
  for (i=1; i <= length($0); ++i)
  {
    c = substr($0, i, 1)
    if (c=="1")
      printf("2,1,")
    else if (c=="0")
      printf("1,2,")
  }
  printf("16,;\n")
}'`

echo $CODE
echo $CODE | nc -w 1 -u 192.168.0.48 49880 
```

Wenn man es so aufruft, Hauscode 011011100000 , Dose #2 EIN:
./gtfsi04 011011100000 10010011

wird diese Sequenz an das Conn Air gesendet:

```
TXP:0,0,5,65535,667,21,1,1,2,2,1,2,1,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,1,2,2,1,1,2,1,2,2,1,1,2,1,2,2,1,2,1,16,;
```

[Quelle](http://simple-solutions.de/forum/viewtopic.php?p=2527#p2527)

Weitere Hauscodes können wie folgt erstellt werden:

```
printf "%012d\n" `echo "obase=2;<int value>" | bc`
```

### Beispielcodes ###

**B1**

ON: `TXP:0,0,5,65535,667,21,1,1,2,2,1,2,1,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,1,2,2,1,1,2,1,2,2,1,1,2,1,2,2,1,2,1,16,;`

OFF: `TXP:0,0,5,65535,667,21,1,1,2,2,1,2,1,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,1,2,2,1,1,2,1,2,1,2,1,2,1,2,2,1,1,2,16,;`

**B2**

ON: `TXP:0,0,5,65535,667,21,1,1,2,2,1,2,1,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,2,1,1,2,1,2,1,2,2,1,16,;`

OFF: `TXP:0,0,5,65535,667,21,1,1,2,2,1,2,1,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,16,;`

**B3**

ON: `TXP:0,0,5,65535,667,21,1,1,2,2,1,2,1,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,1,2,2,1,2,1,1,2,2,1,1,2,1,2,2,1,1,2,16,;`

OFF: `TXP:0,0,5,65535,667,21,1,1,2,2,1,2,1,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,1,2,2,1,2,1,1,2,1,2,1,2,1,2,2,1,2,1,16,;`

**B4**

ON: `TXP:0,0,5,65535,667,21,1,1,2,2,1,2,1,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,1,2,1,2,2,1,1,2,2,1,1,2,1,2,1,2,1,2,16,;`

OFF: `TXP:0,0,5,65535,667,21,1,1,2,2,1,2,1,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,1,2,1,2,2,1,1,2,1,2,1,2,1,2,1,2,2,1,16,;`

**All**

ON: `TXP:0,0,5,65535,667,21,1,1,2,2,1,2,1,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,1,2,2,1,2,1,2,1,2,1,1,2,1,2,1,2,1,2,16,;`

OFF: `TXP:0,0,5,65535,667,21,1,1,2,2,1,2,1,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,1,2,2,1,2,1,2,1,1,2,1,2,1,2,1,2,2,1,16,;`

### Alternative Codes ###
Einige Quigg-Steckdosensysteme arbeiten mit einem minimal veränderten Protokoll. Die Bits sind hier kodiert:

1 Bit: Ein Takt aus, ein Takt ein, ein Takt aus
0 Bit: Zwei Takte ein, ein Takt aus

Die Taktung ist 1.812 kHz, zwischen je zwei Wiederholungen liegen 72 ms Pause. Die Codes lauten:

```
1 an        1 1 0 1 1 0 1 0 1 1 1 0 1 0 0 0 1 0 0 0 1
1 aus       1 1 0 1 1 0 1 0 1 1 1 0 1 0 0 0 0 0 0 0 0

2 an        1 1 0 1 1 0 1 0 1 1 1 0 1 1 0 0 1 0 0 1 1
2 aus       1 1 0 1 1 0 1 0 1 1 1 0 1 1 0 0 0 0 0 1 0

3 an        1 1 0 1 1 0 1 0 1 1 1 0 1 0 1 0 1 0 0 0 0
3 aus       1 1 0 1 1 0 1 0 1 1 1 0 1 0 1 0 0 0 0 0 1

4 an        1 1 0 1 1 0 1 0 1 1 1 0 1 1 1 0 1 0 0 1 0
4 aus       1 1 0 1 1 0 1 0 1 1 1 0 1 1 1 0 0 0 0 1 1

master an   1 1 0 1 1 0 1 0 1 1 1 0 1 1 1 1 1 0 0 0 0
master aus  1 1 0 1 1 0 1 0 1 1 1 0 1 1 1 1 0 0 0 0 1

(z.B. für Steckdose 4:)
dim an      1 1 0 1 1 0 1 0 1 1 1 0 1 1 1 0 0 1 0 0 1
dim aus     1 1 0 1 1 0 1 0 1 1 1 0 1 1 1 0 1 1 0 0 0
            \-----------------------/ \---/ - - - - -
                  System-ID            id   o d 0 v p
                                        \--------/
                                         Funktion

                                         o == on/off
                                         d == dimmen/nicht dimmen
                                         v == prüfziffer für id (??)
                                         p == paritätsbit
```

Die System-ID ist frei wählbar. Der Funktionscode bestimmt, welche Steckdose
wie geschaltet werden soll. Bit 4 kodiert dabei, ob ein- oder ausgeschaltet
wird. id ist die ID der Steckdose, per 1 1 1 werden alle Steckdosen
angesprochen.  Das zweite Bit von on? wird zum Dimmen gesetzt. v ist scheinbar
ein Paritätsbit für Bits 1 und 3 der id. Das Paritätsbit wird über den gesamten
Code gebildet und invertiert.

## Type Nr. 401 ##
Hersteller: NPCDTM Ltd.

Diese Steckdosen hat Conrad früher einmal vertrieben. An den Steckdosen und der
Fernbedienung ist ein dreistelliger, binärer Systemcode einstellbar. Das
Übertragungsprotokoll entspricht fast dem der Quigg-Steckdosen oben: Es wird mit 1.814 kHz mit 3 Takten
kodiert. Ein 0 Bit wird als "Aus, Ein, Aus" kodiert, eine 1 als "Ein, Ein, Aus". Zwischen den Sequenzen
ist je 20.318ms Pause. Die genaue Einhaltung der Zeiten ist hier sehr wichtig, ggfs. lohnt es sich, eine
Empfängereinheit über den Mikrofoneingang der Soundkarte in Kombination mit einem Aufnahmeprogramm zur
Feineinstellung zu verwenden. ([Siehe z.B.](http://gimlie.net/433-mhz-signale-auslesen-und-auswerten/))

Die Codes lauten:

```
#1 an   0 1 1 1 0 x x x 0 0 0 0 0
#1 aus  0 0 1 1 0 x x x 0 0 0 0 0
#2 an   0 1 1 0 0 x x x 0 0 0 0 0
#2 aus  0 0 1 0 0 x x x 0 0 0 0 0
#3 an   0 1 0 1 0 x x x 0 0 0 0 0
#3 aus  0 0 0 1 0 x x x 0 0 0 0 0
#4 an   0 1 0 0 1 x x x 0 0 0 0 0
#4 aus  0 0 0 0 1 x x x 0 0 0 0 0
        - - \---/ \---/ \-------/
        1 2   3     4       5

       1: Initialisierungsbit
       2: An/Aus
       3: Steckdosen ID wie im Werkszustand
       4: Umgekehrter System-Code (1 0 0 für Schalter auf Aus, Aus, An)
       5: Endbits
```
