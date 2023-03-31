# AT Kommando Referenz
## Nutzungsbeispiel

Das System ist typischerweise ca. 20 Sekunden nach dem Einschalten der Spannung zur Ausführung von AT-Kommandos bereit.

Bitte beachten Sie, dass Zeichen vor dem AT zu einem Fehler führen, da nur Kommandos ausgeführt werden, die mit AT beginnen. Sie müssen deshalb das Kommando ggf. mehrmals ausführen, wenn Sie nicht sicher sind, ob vorher Zeichen gesendet wurden. Auch vorher mit falscher Baudrate gesendete Zeichen führen zu einem ERROR. Auch in dem Fall, wenn das `AT<CR><LF>` mit korrekter Baudrate gesendet wird. Es wird empfohlen, das `AT<CR><LF>` nach dem ersten Fehler erneut zu senden, um diesen Effekt nicht als Fehler zu betrachten.

Generell können alle Kommandos in Klein- oder Grosbuchstaben gesendet werden.

Ein typischer Befehlsablauf ist

```
TX: AT
RX: OK
TX: ATD 192.168.0.1
RX: CONNECT
... Senden und Empfangen von Daten ...
TX: +++
RX: OK
TX: ATH
RX: NO CARRIER
RX: OK
```

## Verfügbare Kommandos

Folgende AT-Kommandos stehen zur Verfügung. Nicht implementierte Befehle oder falsch formatierte Befehle werden mit der Rückgabe `ERROR` quittiert.

* [AT](#at)
* [ATA](#ata)
* [ATD](#atd)
* [ATDT](#atdt)
* [ATDU](#atdu)
* [ATH](#atu)
* [ATI](#ati)
* [ATP](#atp)
* [ATT](#att)
* [ATX](#atx)
* [AT**BAUD](#atbaud)
* [AT**DEFAULT](#atdefault)
* [AT**FLASH](#atflash)
* [AT**FORMAT](#atformat)
* [AT**GPRSAPN](#atgprsapn)
* [AT**GSMNET](#atgsmnet)
* [AT**IMEI](#atimei)
* [AT**IMSI](#atimsi)
* [AT**KEEP](#atkeep)
* [AT**LASTCON](#atlastcon)
* [AT**PIN](#atpin)
* [AT**PPPAUTH](#atpppauth)
* [AT**PPPIPCHECK](#atpppipcheck)
* [AT**PPPISP](#atpppisp)
* [AT**PPPPW](#atppppw)
* [AT**PPPQSP](#atpppqsp)
* [AT**PPPUSER](#atpppuser)
* [AT**PROFILE](#atprofile)
* [AT**RROVIDER](#atrrovider)
* [AT**RESET](#atreset)
* [AT**SAVE](#atsave)
* [AT**TCPAGG](#attcpagg)
* [AT**TCPBLOCK](#attcpblock)
* [AT**TCPLISTEN](#attcplisten)
* [AT**TCPLISTENPORT](#attcplistenport)
* [AT**VERSION](#atversion)
* [AT**DEBUGCLI](#atdebugcli)
* [AT**TRANS](#attrans)

---
### AT

Befehl ohne Funktion. Kann genutzt werden, um zu prüfen, ob das Modul einsatzbereit ist.

#### Ausführen

```
TX: AT
RX: OK
```

---
### ATA

Eingehenden Ruf annehmen. Nach dem CONNECT ist das Modul im Datenmodus.

#### Ausführen

```
TX: ATA
RX: CONNECT
```

Dieser Befehl bewirkt, dass ein ankommender Ruf angenommen wird, der über `RING` angezeigt wurde.

Liegt kein ankommender Ruf an, ergibt der Befehl einen `ERROR`.

Bei S0=1 oder größer wird der Ruf nach entsprechender Anzahl Klingelzeiehcn (RING) automatisch angenommen.

---
### ATD

TCP oder UDP-Verbindung zu einer Gegenstelle aufbauen. Nach dem CONNECT ist das Modul im Datenmodus.

#### Ausführen

```
TX: ATD<dest>:<port>
RX: CONNECT
```

|Parameter|Beschreibung|
|--------|------------|
|`<dest>`|Adresse der Gegenstelle entweder als IP-Adresse oder Domain Name.<br>Beispiel:<br>- IP-Adresse: ATD213.4.8.123:1234<br>- Domain Name: ATDexample.org:1234|
|`<port>`|Port der Gegenstelle im Bereich von 1..65535|

Mit ATT oder ATP wird TCP als Standard eingestellt. Mit ATU wird UDP als Standard eingestellt.

Falls der Domainname mit einem Buchstaben L, P, T oder U beginnt, muss zwischen ATD und dem Domainnamen ein Leerzeichen stehen, z.B. `ATD test.com:1234`.

Wurde der Modus ATX0 eingestellt, erscheint nur CONNECT. Bei ATX1 erscheint CONNECT mit <Baudrate>.

---
### ATDT

TCP-Verbindung zu einer Gegenstelle per IP-Adresse aufbauen. Nach dem CONNECT ist das Modul im Datenmodus.

#### Ausführen

```
TX: ATDT<dest>:<port>
RX: CONNECT
```

|Parameter|Beschreibung|
|--------|------------|
|`<dest>`|Adresse der Gegenstelle entweder als IP-Adresse oder Domain Name.<br>Beispiel:<br>- IP-Adresse: ATDT213.4.8.123:1234<br>- Domain Name: ATDTexample.org:1234|
|`<port>`|TCP-Port der Gegenstelle im Bereich von 1..65535|

---
### ATDU

UDP-Verbindung zu einer Gegenstelle per IP-Adresse aufbauen. Nach dem CONNECT ist das Modul im Datenmodus.

#### Ausführen

```
TX: ATDU<dest>:<port>
RX: CONNECT
```

|Parameter|Beschreibung|
|--------|------------|
|`<dest>`|Adresse der Gegenstelle entweder als IP-Adresse oder Domain Name.<br>Beispiel:<br>- IP-Adresse: ATDU213.4.8.123:1234<br>- Domain Name: ATDUexample.org:1234|
|`<port>`|UDP-Port der Gegenstelle im Bereich von 1..65535|

---
### ATH

Trennen der Verbindung

#### Ausführen

```
TX: ATH
RX: NO CARRIER
    OK
```
Hinweis: Wenn eine Datenverbindung besteht (nach CONNECT-Meldung), muss erst mit `+++` in den Kommandomodus gewechselt werden. Vor und nach dem `+++` darf mindestens 1 Sekunden lang kein Zeichen gesendet werden.

Zum Trennen der Verbindung kann auch die Hardware-Leitung DTR auf HIGH-Pegel gesetzt werden (Invertierte Logic).

---
### ATI

Abfragen von Informationen.

#### Ausführen

```
TX: ATI<n>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<n>`|0,1,2 = Anzeige Produkt Code "SSV GATEWAY"<br>3 = Version der Emulationssoftware<br>4 = Version des Linux Betriebssystems|

---
### ATP

Umstellen auf TCP als Standart für den Befehl ATD.

#### Ausführen

```
TX: ATP
RX: OK
```

Siehe auch ATT und ATU.

---
### ATT

Umstellen auf TCP als Standart für den Befehl ATD.

#### Ausführen

```
TX: ATT
RX: OK
```

Siehe auch ATP und ATU.

---
### ATU

Umstellen auf UDP als Standart für den Befehl ATD.

#### Ausführen

```
TX: ATU
RX: OK
```

Siehe auch ATP und ATT.

---
### ATX

Das Verhalten der erweiterten Meldungen beim CONNECT einstellen.

#### Ausführen

```
TX: ATX<n>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<n>`|0 = keine erweitere Meldungen<br>1 = Baudate nach dem CONNECT anzeigen|

---
### AT**BAUD

Setzen und Lesen der Baudrate der seriellen Schnittstelle.

Beim Setzen wird die Einstellung nach dem "OK" übernommen.

#### Lesen

```
TX: AT**BAUD?
RX: BAUD: <baud>
    OK
```

#### Schreiben

```
TX: AT**BAUD=<baud>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<baud>`|Baudrate.<br>Mögliche Werte: 300, 600, 1200, 2400, 4800, 9600, 19200, 38400, 57500, 115200

Nicht unterstützte Baudraten: 14400, 28800

#### Default

```
AT**BAUD=115200
```

---
### AT**DEFAULT

Lädt die Werksvoreinstellungen. Siehe auch AT&F.

Das Modul führt anschließend einen Neustart (Reconnect) für das LTE-Modem durch.

#### Schreiben

```
TX: AT**DEFAULT
RX: OK
```

---
### AT**FLASH

Einspielen einer neuen Firmware.

#### Ausführen

```
TX: AT**FLASH
RX: Send *.HEX file now.
TX: <hexfile>
RX: Upload OK. Updating FW...
```

|Parameter|Beschreibung|
|--------|------------|
|`<hexfile>`|Firmware-Datei im Hex-Format. Bereitgestellt durch SSV.

---
### AT**FORMAT

Setzen und Lesen des Datenformats der seriellen Schnittstelle.

Beim Setzen wird die Einstellung nach dem "OK" übernommen.

#### Lesen

```
TX: AT**FORMAT?
RX: FORMAT: <format>
    OK
```

#### Schreiben

```
TX: AT**FORMAT=<format>
RX: OK
```

|Parameter|Beschreibung|
|---------|------------|
|`<format>`|8N1 = 8 Datenbits, 1 Stop-Bit, keine Parität<br>8E1 = 8 Datenbits, 1 Stop-Bit, gerade Parität<br>8O1 = 8 Datenbits, 1 Stop-Bit, ungerade Parität<br>8N2 = 8 Datenbits, 2 Stop-Bit, keine Parität<br>7N1 = 7 Datenbits, 1 Stop-Bit, keine Parität<br>7E1 = 7 Datenbits, 1 Stop-Bit, gerade Parität<br>7O1 = 7 Datenbits, 1 Stop-Bit, ungerade Parität<br>7N2 = 7 Datenbits, 2 Stop-Bit, keine Parität<br>7E2 = 7 Datenbits, 2 Stop-Bit, gerade Parität<br>7O2 = 7 Datenbits, 2 Stop-Bit, ungerade Parität

#### Default

```
AT**FORMAT=8N1
```

---
### AT**GPRSAPN

Zugriff auf die Einstellung des APN

#### Lesen

```
TX: AT**GPRSAPN?
RX: GPRSAPM: <apn>
    OK
```

#### Schreiben

```
TX: AT**GPRSAPN=<apn>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<apn>`|APN (Access Point Name) des GPRS Providers.

Ein leerer APN ist nicht erlaubt.

Der neue APN wird erst beim nächsten AT**RESET aktiv.

#### Default

```
AT**GPRSAPN=
```

---
### AT**GSMNET

Anzeigen der aktuellen Netzparameter

#### Lesen
```
TX: AT**GSMNET?
RX: STATE: <state>
    CONNECT: <connect>
    SIGNAL: <quality>
    OPER: <oper>
    MCC: <mcc>
    MNC: <mnc>
    LAC: <lac>
    CID: <cid>
    OK
```

|Ergebnis|Beschreibung|
|---------|------------|
|`<state>`|Verbindungsstatus (running, pending, stopped)
|`<connect>`|Details über die Verbindung (CONNECTED, CONNECTING, ENABLING, DISCONNECTING, ...)
|`<quality>`|Empfangsqualität in Prozent 0..100%
|`<oper>`|Angabe des Netzbereibers
|`<mcc>`|Mobil Country Code, dezimal
|`<mnc>`|Mobil Network Code, dezimal
|`<lac>`|Location Area Code, hexadezimal
|`<cid>`|Cell ID der aktuellen Mobilfunkzelle, hexadezimal

Die Zelleninformationen MCC, MNC, LAC und CID stehen bei aktiver LTE-Internet-Verbindung nicht zur Verfügung.

---
### AT**IMEI

Anzeige der Modem IMEI

#### Lesen

```
TX: AT**IMEI?
RX: IMEI: <imei>
    OK
```

|Ergebnis|Beschreibung|
|---------|------------|
|`<imei>`|International Mobile Equipment Identity des Modems

---
### AT**IMSI

Anzeige der SIM-Karte IMSI

#### Lesen

```
TX: AT**IMSI?
RX: IMSI: <imsi>
    OK
```

|Ergebnis|Beschreibung|
|---------|------------|
|`<imsi>`|International Mobile Subscriber Identity der SIM-Karte

---
### AT**KEEP

TCP Keep-Alive

#### Lesen

```
TX: AT**KEEP?
RX: KEEP: <intervall>
    OK
```

#### Schreiben

```
TX: AT**KEEP=<intervall>
RX: OK
```

|Parameter|Beschreibung|
|---------|------------|
|`<intervall>`|0 = keine TCP-Keep-Alive Pakete senden<br>1..255 = Zeit zwischen zwei Paketen in Minuten

Die Anzahl Wiederholungen, bis eine Verbindung abgebrochen wird stehen im Register S70 (default 5). Das Keepalive Intervall steht gleichfalls im Register S72.

#### Default

```
AT**KEEP=6
```

---
### AT**LASTCON

Anzeige von Informationen zur letzten Verbindung

#### Lesen

```
TX: AT**LASTCON?
RX: LAST CONNECTION:
    REMOTE IP:         123.123.123.123
    REMOTE MAC:        12:12:12:12:12:12
    REMOTE PORT:       12345
    LOCAL PORT:        6400

    OK
```

---
### AT**PIN

PIN der SIM-Karte

#### Lesen

```
TX: AT**PIN?
RX: PIN: <status>
    OK
```

|Ergebnis|Beschreibung|
|--------|------------|
|`<status>`|active = PIN hinterlegt<br>inactive = Keine PIN hinterlegt

#### Schreiben

```
TX: AT**PIN=<pin>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<pin>`|4- oder 5-Stellige SIM-PIN<br>Alternativ kann leer sein, um die PIN zu löschen

#### Default

```
AT**PIN=
```

---
### AT**PPPAUTH

PPP-Authentifizierungsart

#### Lesen

```
TX: AT**PPPAUTH?
RX: PPPAUTH: <auth>
    OK
```

#### Schreiben

```
TX: AT**PPPAUTH=<auth>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<auth>`|0 = keine Authentifizierungsart<br>1 = PAP<br>2 = CHAP

#### Default

```
AT**PPPAUTH=0
```

---
### AT**PPPIPCHECK

Verbindungscheck

Siehe auch AT#PING

#### Lesen

```
TX: AT**PPPIPCHECK?
RX: PPPIPCHECK: <timeout>,<mode>
    OK
```

#### Schreiben

```
TX: AT**PPPIPCHECK=<timeout>,<mode>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<timeout>`|Wird ignoriert. Test erfolgt sofort.
|`<mode>`|0 = Test per DNS-Abfrage<br>1 = Test per Ping-Abfrage and die per AT**AUTOIP hinterlegten Adressen.

#### Ausführen

```
TX: AT**PPPIPCHECK
RX: +PPP-IP <result>
    OK
```

|Ergebnis|Beschreibung|
|--------|------------|
|`<result>`|OK = Verbindung erfolgreich getestet. Verbindung ist in Ordnung.<br>RELEASE = Verbindung nicht in Ordnung.

ERROR erscheint, wenn keine AUTOIP gesetzt wurde.

#### Default

```
AT**PPPIPCHECK=0,0
```

---
### AT**PPPISP

Mobile network operator

#### Lesen

```
TX: AT**PPPISP?
RX: PPPISP: <operator>
    OK
```

#### Schreiben

```
TX: AT**PPPISP=<operator>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<operator>`|Mobile network operator: auto, d1, d2, eplus, o2, user

Bei `auto` wird an Hand der SIM-Karte der Provider d1, d2, eplus oder o2 zu ermitteln, und dann werden für diesen Provider die Standardeinstellungen für APN und Benutzer eingetragen.
Mit `user` müssen APN, Benutzername, Password, Authentifizeirungsmodus (PAP, CHAP) sowie PPPQSP (falls nötig) manuell eingestellt werden.

#### Default

```
AT**PPPPISP=auto
```

---
### AT**PPPPW

PPP-Passwort

#### Lesen

```
TX: AT**PPPPW?
RX: PPPPW: <pw>
    OK
```

#### Schreiben

```
TX: AT**PPPPW=<pw>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<pw>`|PPP-Passwort

#### Default

```
AT**PPPPW=
```

---
### AT**PPPQSP

Quality of service profile

#### Lesen

```
TX: AT**PPPQSP?
RX: PPPQSP: <qsp>
    OK
```

#### Schreiben

```
TX: AT**PPPQSP=<qsp>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<qsp>`|Quality of service profile request

#### Default

```
AT**PPPQSP=
```

---
### AT**PPPUSER

PPP-Benutzername

#### Lesen

```
TX: AT**PPPUSER?
RX: PPPUSER: <user>
    OK
```

#### Schreiben

```
TX: AT**PPPUSER=<user>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<user>`|PPP-Benutzername

#### Default

```
AT**PPPUSER=
```

---
### AT**RESET

Das Modul führt einen Reset durch.

Gespeicherte Einstellungen und Werte bleiben erhalten.

#### Schreiben

```
TX: AT**RESET
RX: OK
```

---
### AT**SAVE

Speichern aller Einstellungen. Das System konfiguriert sich mit den neuen Werten. Anschliessend sollte ein AT**RESET durchgeführt werden.

#### Schreiben

```
TX: AT**SAVE
RX: OK
```

---
### AT**TCPAGG

TCP-Blockbildungszeit

#### Lesen

```
TX: AT**TCPAGG?
RX: TCPAGG: <aggtime>
    OK
```

#### Schreiben

```
TX: AT**TCPAGG=<aggtime>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<aggtime>`|Wartezeit in Millisekunden. Werte im Bereich von 1..65535.<br>1 ms bedeutet keine Blockbildung.

Siehe auch Register S8.

#### Default

```
AT**TCPAGG=50
```

---
### AT**TCPBLOCK

TCP-Blockgröße

#### Lesen

```
TX: AT**TCPBLOCK?
RX: TCPBLOCK: <blocksize>
    OK
```

#### Schreiben

```
TX: AT**TCPBLOCK=<blocksize>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<blocksize>`|Anzahl von Bytes zur Bildung von TCP-Datenpaketen. Werte im Bereich 10..512.

Hinweis: Über den Befehl AT#MSS kann die Blcokgröße über einen größeren Bereich eingestellt oder auch abgeschaltet werden.

#### Default

```
AT**TCPBLOCK=1460
```

---
### AT**TCPLISTEN

Aktiviert das mobile Modem.

#### Lesen

```
TX: AT**TCPLISTEN?
RX: TCPLISTEN: <mode>
    OK
```

#### Schreiben

```
TX: AT**TCPLISTEN=<mode>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<mode>`|0 = TCP Listen aus<br>1 = TCP Listen ein

Der geänderte Modus wird erst beim nächsten AT**RESET übernommen.
Wenn mit `0` das TCP Listen deaktiviert wird, wird die LTE-Internet-Verbindung beendet.

#### Default

```
AT**TCPLISTEN=1
```

---
### AT**TCPLISTENPORT

TCP Datenport für eingehende TCP-Verbindungen in Betriebsart TCP Listen

#### Lesen

```
TX: AT**TCPLISTENPORT?
RX: TCPLISTENPORT: <port>
    OK
```

#### Schreiben

```
TX: AT**TCPLISTENPORT=<port>
RX: OK
```

|Parameter|Beschreibung|
|--------|------------|
|`<port>`|TCP-Port, auf dem auf eingehende TCP-Verbindungen gewartet wird. Werte von 1...65535.

#### Default

```
AT**TCPLISTENPORT=6400
```

---
### AT**VERSION

Abfrage der Software-Version

Siehe auch ATI3, ATI4

#### Lesen

```
TX: AT**VERSION?
RX: Manufacturer: SSV GATEWAY
    Model: eDNP/8331
    Firmware: 2022.09.20-1509, 0.3.9
    SW-Version: 1.1.0rc0
```

---
### AT**DEBUGCLI

Debug-Schnittstelle auf dem seriellen Interface.

#### Ausführen

```
TX: AT**DEBUGCLI
RX: OK

    +DEBUGCLI: ON

    emblinux login:
```

Nach dem OK ist die Debug-Schnittstelle bereit, und es erscheint ein Linux Login Prompt.
Der Login und die Linux-Kommandos sind nur für die Entwicklung vom Hersteller zu benutzen.

Nach Beendigung der Debug-Console wird der Modem-Emulator wieder gestartet. Die Debug-Console beendet sich automatisch nach 60 Sekunden oder nach zu vielen fehlerhaften Logins.

---
### AT**TRANS

Schaltet das LTE-Modul transparent.

#### Ausführen

```
TX: AT**TRANS
RX: OK
    
    +TRANS: ON
```

Eine bestehende mobile Verbindung (PPP) wird beendet.

Nach dem OK werden alle Befehle transparent an das LTE-Modem weiter geleitet. Die aktuelle Baudrate wird beibehalten. Die Schnittstellenparameter sind fest auf 8N1 eingestellt. Änderungen der Schnittstellenparameter (Baud, Bitanzahl, Parität) sind nicht zulässig und führen zum Verlust des Kommando-Interfaces.

Sämtliche direkt an das LTE-Modem ausgeführten Befehle können zu Funtionsbeeiträchtigung der Firmware führen.
Dieser Modus kann nur durch Hard- oder Software-Reset beendet werden.

Falls das Modem deaktiviert ist (Power off) erscheint eine Fehlermeldung.
