### Wie funktioniert SDU (Secure Device Update)?

Eine SDU-Anwendung soll eine Komponente (Device) hinter dem RMG/941 über eine IoT-Anbindung mit Software-, Konfigurations- und Machine-Learning-Modell-Updates versorgen. Dabei liegt der Schwerpunkt, neben der Remote-Update-Funktion, im Bereich der IT-Sicherheit. Um eine dem Stand der Technik entsprechende Sicherheit zu gewährleisten, nutzt SDU eine Public-Key-Infrastruktur (PKI) für digitale Signaturen mit privaten und öffentlichen Schlüsseln, Zertifikaten und einer Sperrliste (Blacklist).

![Übersicht: Zusammenhänge SDU](https://ssv-comm.de/forum/bilder/sdu_141220.png) 

Der gesamte Update-Prozess wurde für den vollautomatischen Betrieb entwickelt und lässt sich in Bezug auf die Device-Anzahl nahezu beliebig skalieren. Die Abbildung zeigt die wichtigsten SDU-Funktionsbausteine.

**1:** SDU Server. Dieser Serverprozess läuft im Internet und dient für die Weitergabe der Updates als Bindeglied zwischen dem PC des Softwareentwickler (bzw. zuständigen Maintainer) und der Device hinter dem RMG/941. Ein SDU Server wird als Docker Container Image ausgeliefert und ist daher nahezu in jeder geeigneten Docker-Laufzeitumgebung ausführbar.

**2:** RMG/941 mit SDU Agent und SDU Gateway Update Client (SDU GUC). Der SDU GUC ist für die Kommunikation mit dem SDU Server zuständig. Er sorgt dafür, dass eine gültige Update-Datei bei Bedarf vom Server zum RMG/941 heruntergeladen und dem SDU Agent zur Verfügung gestellt wird. Ein anwendungsbezogener SDU Agent übernimmt die Kommunikation mit der jeweiligen Device hinter dem RMG/941, z. B. um die Update-Datei an die Device zu übertragen. Der SDU Agent muss daher projektspezifisch an das Protokoll der jeweiligen Device angepasst werden. Beide Softwarekomponenten werden in der Regel von SSV werksseitig vorinstalliert und für eine bestimmte Anwendung konfiguriert.

**3:** Entwickler-PC mit SDU Signing Tool und SDU Maintainer Update Client (SDU MUC). Mit dem Signing Tool werden die für den Update erforderlichen einzelnen Dateien auf dem PC zu einer SDU-Update-Datei zusammengefügt und mit einer digitalen Unterschrift (digital Signature) versehen. Mit Hilfe des SDU MUC wird diese signierte Datei dann zum SDU Server hochgeladen. Sowohl das SDU Signing Tool als auch der SDU Maintainer Update Client wurden als Electron-Cross-Plattform-Desktop-Anwendung entwickelt. Sie lassen sich auf Linux-, macOS- und Windows-Rechnern ausführen. 

**Device:** Das eigentliche Zielsystem für den Update. Es ist per CAN, Modbus (RTU, TCP), Nahbereichs-Funkschnittstelle usw. mit dem RMG/941 verbunden. Über den RMG/941-SDU-Agent wurde das Gateway an das jeweilige Update-Protokoll der Device angepasst.

### Wie extrahiert man Messwerte aus SSV/SFS-Daten?

Die Daten eines Smart Factory Sensors (SFS) werden vielfach als JSON-Objekt verschickt. Für die weitere Verarbeitung müssen die jeweils benötigten Daten aus diesen Objekten extrahiert werden. Der folgende Textblock zeigt als Beispiel das JSON-Objekt eines SFS mit drei Sensorelementen:

```javascript
{
  "gw" : "94:54:93:56:57:8c",
  "address" : "bc:dd:c2:c4:7c:1e",
  "name" : "SSV-SFS",
  "rssi" : -47,
  "ts" : 1618660439,
  "type" : "ssvSensorBox",
  "ssvSensorBox" : {
    "temperature" : 23.280000686645508,
    "pressure" : 101.8153076171875,
    "humidity" : 34.5595703125
  }
}
```

Insgesamt liefert der SFS in diesem Beispiel-JSON-Objekt die aktuellen Messwerte für Temperatur, Luftdruck und relative Luftfeuchte. Der hier abgebildete Node-RED-Flow verdeutlicht, wie die JSON-Daten auseinandergenommen werden, um die Rohdaten der einzelnen Sensorelemente gemäß den jeweiligen Anforderungen zu nutzen. 

![Node-RED-Flow](https://ssv-comm.de/forum/bilder/nr_170421.png)

Der Dateneingang des Flows erhält die SFS-Daten von einem MQTT Broker (MQTT Subscribe). Danach wird das per MQTT erhaltende String-Objekt in ein reines JSON-Objekt umgewandelt und an den Function Node „Measurement Extraction“ weitergeleitet.    

```javascript
var d = msg.payload.ssvSensorBox.temperature;
d = d.toFixed(1);
var msg1 = {payload:d};

d = msg.payload.ssvSensorBox.pressure;
d = d.toFixed(1);
var msg2 = {payload:d};

d = msg.payload.ssvSensorBox.humidity;
d = d.toFixed(1);
var msg3 = {payload:d};

return [msg1, msg2, msg3];
```
Der JavaScript-Code des Function Node extrahiert die Messwerte für Temperatur, Luftdruck und relative Luftfeuchte aus dem JSON-Objekt, begrenzt die Zahlenlänge auf eine Nachkommastelle und stellt die drei Sensormesswerte jeweils an einem eigenen Ausgang zur Verfügung. Dort übernimmt sie in diesem Beispiel ein Debug Node und erzeugt eine Messwertausgabe im Debug Sidebar.

### Wie funktioniert der virtuelle Hands-on in einem SSV-Sensorik-Webinar?

Die Sensoren und Teilnehmer befinden sich an unterschiedlichen Standorten (Sensoren z. B. am Arbeitsplatz eines SSV-Mitarbeiters in Hannover, Teilnehmer an belibeigen Orten irgendwo auf der Welt). 

![Node-RED-Flow](https://ssv-comm.de/forum/bilder/co_200421.png)

<a href="https://github.com/kdw1000/Test/blob/master/_161120.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>

<a href="https://github.com/kdw1000/Test/blob/master/_250321.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>
