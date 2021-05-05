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

In diesen Webinaren lernen die Teilnehmer in der Regel unterschiedliche Sensordatenformate für Digitalisierungsaufgaben (z. B. JSON, CSV/TSV, Merkmalsvektoren) plus einige Erweiterungen (ISO 8601-Zeitstempel für Zeitreihendaten, Sensorfusion usw.) kennen. Aber auch die Sensordatenvorverarbeitung (z. B. Z-Score-Normalisierung, Detrending, FFT) und Datenanalysen per Machine Learning kommen zur Sprache.

In dem Hands-on eines solchen Webinars geht es darum, den Teilnehmern unter fachlicher Anleitung die Möglichkeit zu geben, mit echten Sensordaten zu arbeiten und einige zuvor erlernte Dinge selbst auszuprobieren. Die Sensoren und Teilnehmer befinden sich dabei allerdings an unterschiedlichen Standorten (die Sensoren z. B. am Arbeitsplatz eines SSV-Mitarbeiters in Hannover, die Teilnehmer an beliebigen Orten irgendwo auf der Welt). 
 
![Node-RED-Flow](https://ssv-comm.de/forum/bilder/co_200421.png)

Insofern werden die Sensordaten für die Übungen in Echtzeit an einen MQTT Broker im Internet übermittelt. Dort kann jeder Teilnehmer die Daten mit einem MQTT-Subscribe abonnieren und auf dem eigenen Rechner weiterbearbeiten. Dafür benötigt der Teilnehmer lediglich ein kostenloses Google-Konto mit einem Webbrowser-Zugang zu Google Colaboratory  (Colab). Die für die Übungen erforderlichen Python-Codebausteine werden jeweils vom SSV-Trainer zur Verfügung gestellt.

### Was ist ein PyDSlog-Docker?

Der Maschinensensor MLS/160A liefert ein relativ komplexes Datenbild mit verschiedenen Kanälen (bis zu sechs Achsen mit Beschleunigung und Winkelgeschwindigkeit) sowie einstellbarer Abtastfrequenz innerhalb eines definierbaren Zeitfensters. Als Zubehör ist nun ein MLS/160A-Support Docker verfügbar.

![Node-RED-Flow](https://ssv-comm.de/forum/bilder/py_230421.png) 

Mit diesen PyDSlog Docker wollen wir die Datenqualitätsanforderungen einer Machine-Learning-Applikation so präzise wie möglich umsetzen und Standardschnittstellen zu anderen Anwendungen schaffen. Der Docker unterstützt nicht nur die Sensorkonfiguration. Er dient auch zum sicheren Remote Update der MLS/160A-Firmware. Ein PyDSlog Docker ist auf einem SSV-Gateway und auf anderen geeigneten Plattformen in der Edge einsetzbar.

### Virtueller Hands-on am 06.05.2021: Code-Beispiele für Colob

1) MQTT-Bibliothek in Colab installieren

```python
!pip install paho-mqtt
```

2) Sensordaten per MQTT empfangen uns ausgeben

```python
import paho.mqtt.client as mqtt

# MQTT broker connect callback function ...
def on_connect(client, userdata, flags, rc):  
    print("Connected with result code {0}".format(str(rc)))
    client.subscribe("/xxx")

# Message RX callback function ...
def on_message(client, userdata, msg):
    print("Message received-> " + msg.topic + " " + str(msg.payload))

# Connect to the MQTT broker and receive messages forever ...
client = mqtt.Client("ssv_mqtt_test")
client.on_connect = on_connect
client.on_message = on_message
client.connect("test.mosquitto.org", 1883, 60)
client.loop_forever()
```

3) Sensordaten per MQTT empfangen und in CSV-Datei speichern

```python
import paho.mqtt.client as mqtt 
import pandas as pd
import datetime
import json

# MQTT broker connect callback function ...
def on_connect(client, userdata, flags, rc):  
    file = open('/content/test.csv', 'w')
    file.write("Timestamp;" + "Temperature;" + "Pressure;" + "Humidity" + '\n')
    file.close()
    print("Connected with result code {0}".format(str(rc)))
    client.subscribe("/xxx")

# Message RX callback function ...
def on_message(client, userdata, msg):
    
    print("Message received-> " + msg.topic + " " + str(msg.payload))

    data = json.loads(msg.payload)
    data_ts = data['ts']
    data_ts = pd.to_datetime(data_ts, unit='s') # ISO 8601 timestamp, GMT 
    data_ts = data_ts.isoformat()
    #data_ts = datetime.datetime.now()
    #data_ts = data_ts.isoformat(timespec='seconds')
    data_temp = round(data['ssvSensorBox']['temperature'],1)
    data_pres = data['ssvSensorBox']['pressure'] * 10
    data_pres = round(data_pres, 1)
    data_hum = round(data['ssvSensorBox']['humidity'],1)

    file = open('/content/test.csv', 'a')
    file.write(str(data_ts) + ';' + str(data_temp) + ';' + str(data_pres) + ';' + str(data_hum) + '\n')
    file.close() 

# Connect to the MQTT broker and receive messages forever ...
client = mqtt.Client("ssv_mqtt_test")
client.on_connect = on_connect
client.on_message = on_message
client.connect("test.mosquitto.org", 1883, 60)
client.loop_forever()
```

4) CSV-Datei mit Sensordaten auswerten

```python
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime

url = '/content/test.csv'

csv=pd.read_csv(url, parse_dates=True, sep=';')
csv["Timestamp"]=pd.to_datetime(csv["Timestamp"])  # ISO 8601 time stamp
csv.index=csv["Timestamp"]
del csv["Timestamp"]
print(csv.head(10))
print(':')
print(csv.tail(10))
print(' ')
print(csv.describe())
print(csv.isnull().sum().sum())

csvRS = csv
start_time_str = csv.index[0]
start_time = str(start_time_str.strftime('%Y-%m-%d %H:%M'))
end_time_str = csv.index[csv.shape[0]-1]
end_time = str(end_time_str.strftime('%Y-%m-%d %H:%M'))

fig, ax = plt.subplots(figsize=(12, 4))
ax = plt.axes()
ax.set(title="CSV Log from " + start_time + " to " + end_time)
plt.plot(csvRS["Temperature"])
#plt.plot(csvRS["Pressure"])
#plt.plot(csvRS["Humidity"])
plt.grid()
plt.show()
```

<a href="https://github.com/kdw1000/Test/blob/master/_161120.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>

<a href="https://github.com/kdw1000/Test/blob/master/_250321.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>
