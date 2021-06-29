## Wie funktioniert SDU (Secure Device Update)?

Eine SDU-Anwendung soll eine Komponente (Device) hinter dem RMG/941 über eine IoT-Anbindung mit Software-, Konfigurations- und Machine-Learning-Modell-Updates versorgen. Dabei liegt der Schwerpunkt, neben der Remote-Update-Funktion, im Bereich der IT-Sicherheit. Um eine dem Stand der Technik entsprechende Sicherheit zu gewährleisten, nutzt SDU eine Public-Key-Infrastruktur (PKI) für digitale Signaturen mit privaten und öffentlichen Schlüsseln, Zertifikaten und einer Sperrliste (Blacklist).

![Übersicht: Zusammenhänge SDU](https://ssv-comm.de/forum/bilder/sdu_141220.png) 

Der gesamte Update-Prozess wurde für den vollautomatischen Betrieb entwickelt und lässt sich in Bezug auf die Device-Anzahl nahezu beliebig skalieren. Die Abbildung zeigt die wichtigsten SDU-Funktionsbausteine.

**1:** SDU Server. Dieser Serverprozess läuft im Internet und dient für die Weitergabe der Updates als Bindeglied zwischen dem PC des Softwareentwickler (bzw. zuständigen Maintainer) und der Device hinter dem RMG/941. Ein SDU Server wird als Docker Container Image ausgeliefert und ist daher nahezu in jeder geeigneten Docker-Laufzeitumgebung ausführbar.

**2:** RMG/941 mit SDU Agent und SDU Gateway Update Client (SDU GUC). Der SDU GUC ist für die Kommunikation mit dem SDU Server zuständig. Er sorgt dafür, dass eine gültige Update-Datei bei Bedarf vom Server zum RMG/941 heruntergeladen und dem SDU Agent zur Verfügung gestellt wird. Ein anwendungsbezogener SDU Agent übernimmt die Kommunikation mit der jeweiligen Device hinter dem RMG/941, z. B. um die Update-Datei an die Device zu übertragen. Der SDU Agent muss daher projektspezifisch an das Protokoll der jeweiligen Device angepasst werden. Beide Softwarekomponenten werden in der Regel von SSV werksseitig vorinstalliert und für eine bestimmte Anwendung konfiguriert.

**3:** Entwickler-PC mit SDU Signing Tool und SDU Maintainer Update Client (SDU MUC). Mit dem Signing Tool werden die für den Update erforderlichen einzelnen Dateien auf dem PC zu einer SDU-Update-Datei zusammengefügt und mit einer digitalen Unterschrift (digital Signature) versehen. Mit Hilfe des SDU MUC wird diese signierte Datei dann zum SDU Server hochgeladen. Sowohl das SDU Signing Tool als auch der SDU Maintainer Update Client wurden als Electron-Cross-Plattform-Desktop-Anwendung entwickelt. Sie lassen sich auf Linux-, macOS- und Windows-Rechnern ausführen. 

**Device:** Das eigentliche Zielsystem für den Update. Es ist per CAN, Modbus (RTU, TCP), Nahbereichs-Funkschnittstelle usw. mit dem RMG/941 verbunden. Über den RMG/941-SDU-Agent wurde das Gateway an das jeweilige Update-Protokoll der Device angepasst.

## Wie extrahiert man Messwerte aus SSV/SFS-Daten?

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

## Wie funktioniert der virtuelle Hands-on in einem SSV-Sensorik-Webinar?

In diesen Webinaren lernen die Teilnehmer in der Regel unterschiedliche Sensordatenformate für Digitalisierungsaufgaben (z. B. JSON, CSV/TSV, Merkmalsvektoren) plus einige Erweiterungen (ISO 8601-Zeitstempel für Zeitreihendaten, Sensorfusion usw.) kennen. Aber auch die Sensordatenvorverarbeitung (z. B. Z-Score-Normalisierung, Detrending, FFT) und Datenanalysen per Machine Learning kommen zur Sprache.

In dem Hands-on eines solchen Webinars geht es darum, den Teilnehmern unter fachlicher Anleitung die Möglichkeit zu geben, mit echten Sensordaten zu arbeiten und einige zuvor erlernte Dinge selbst auszuprobieren. Die Sensoren und Teilnehmer befinden sich dabei allerdings an unterschiedlichen Standorten (die Sensoren z. B. am Arbeitsplatz eines SSV-Mitarbeiters in Hannover, die Teilnehmer an beliebigen Orten irgendwo auf der Welt). 
 
![Node-RED-Flow](https://ssv-comm.de/forum/bilder/co_200421.png)

Insofern werden die Sensordaten für die Übungen in Echtzeit an einen MQTT Broker im Internet übermittelt. Dort kann jeder Teilnehmer die Daten mit einem MQTT-Subscribe abonnieren und auf dem eigenen Rechner weiterbearbeiten. Dafür benötigt der Teilnehmer lediglich ein kostenloses Google-Konto mit einem Webbrowser-Zugang zu Google Colaboratory  (Colab). Die für die Übungen erforderlichen Python-Codebausteine werden jeweils vom SSV-Trainer zur Verfügung gestellt.

## Virtueller Hands-on am 25.06.2021: Code-Beispiele für Colab

Der Hands-on gehört zum Webinar "IoT-Funksensorik für Machine-Learning-Anwendungen". Die Teilnehmer können per Colab live MQTT-Sensordaten empfangen, anzeigen, in einer Zeitreihen-CSV-Datei speichern und visualisieren. Des Weiteren wird ein einfaches Regressionsmodell per TensorFlow erzeugt.  

### 1) MQTT-Bibliothek in Google-Colab installieren

Um unter Colab das MQTT-Protokoll zu nutzen, muss zunächst per `pip` eine MQTT-Bibliothek installiert werden. Führen Sie daher die folgende Codezeile aus:

```python
!pip install paho-mqtt
```
Für Colab-Sitzungen gibt es ein Session-Timeout. Durch ein solches Timeout geht der Kontext einer Sitzung inkl. der per `pip` installierten Bibliotheken verloren. Insofern ist nach einem solchen Timeout die `pip`-Codezeile erneut auszuführen.      

### 2) Sensordaten per MQTT empfangen und ausgeben

Mit den folgenden Codezeilen werden Sensordaten direkt in Colab per MQTT empfangen und in Zeilenform ausgegeben. Die Daten werden während des Webinars vom einem Bluetooth-Funksensor an ein Edge-Gateway geschickt und von dort aus an den MQTT-Broker gesendet. 

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
**Bitte unbedingt beachten:** Jeder MQTT-Client, der sich mit einem MQTT-Broker verbindet, benötigt eine individuelle Client-ID. Tauschen Sie bitte den Text „ssv_mqtt_test“ in der Codezeile `client = mqtt.Client("ssv_mqtt_test")` gegen eine andere Zeichenfolge Ihrer Wahl aus. Ansonsten kann es sein, dass der Broker Ihren MQTT-Verbindungsaufbau ablehnt, weil zuvor schon jemand anders mit der gleichen ID eine Verbindung aufgebaut hat.

Wie Sie eine Colab-Codezelle mit diesem Code zur Ausführung bringen, läuft der Code praktisch ‚für immer‘ in einer Endlosschleife (also auf jeden Fall bis zum nächsten Session-Timeout). Um die Ausführung zu beenden, benutzen Sie mit im Menü *Laufzeit* den Menüpunkt *Ausführung unterbrechen*.    

### 3) Sensordaten per MQTT empfangen und in einer CSV-Datei speichern

Dieses Beispiel entspricht funktional dem Vorgänger *2) Sensordaten per MQTT empfangen und ausgeben*. Die per MQTT empfangenen Sensordaten werden allerdings nicht nur zeilenweise ausgegeben, sondern unter Colab auch in einer CSV-Datei gespeichert. Beachten Sie bitte, dass auch hier die Client-ID „ssv_mqtt_test“ in der Codezeile `client = mqtt.Client("ssv_mqtt_test")` gegen eine andere Zeichenfolge Ihrer Wahl auszutauschen ist.

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
Um den aktuellen Inhalt der CSV-Datei auszugeben, reicht ein Mausklick auf das Dateisymbol am linken Bildschirmrand des Colab-Fensters. Danach ist im Colab-Dateibereich ein Doppelklick auf den Dateinamen *test.csv* erforderlich. Dadurch entsteht rechts ein weiteres Fenster, indem Colab Ihnen die CSV-Daten anzeigt.

### 4) CSV-Datei mit den Sensordaten auswerten

Die unter *3) Sensordaten per MQTT empfangen und in einer CSV-Datei speichern* erzeugte CSV-Datei *test.csv* lässt sich mit den folgenden Codezeilen auswerten und als Plot darstellen:

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
Bitte beachten: Durch das Session-Timeout einer Colab-Sitzung geht im Colab-Dateibereich auch die Datei *test.csv* verloren. 

### 5) TensorFlow-Modell erzeugen und für Vorhersagen nutzen

Die folgenden Colab-Codebausteine bilden so etwas wie ein „Hallo Welt!“ des Supervised Machine Learning mit TensorFlow. Es wird ein Modell für eine lineare Regression `y = mx + b` erzeugt und für Vorhersagen genutzt.  

Ein Lernalgorithmus soll in diesem Beispiel die Gewichtungen für ein Modell aus den zur Verfügung stehenden Trainingsdaten erlernen. Diese Gewichtungen beschreiben die Wahrscheinlichkeit, dass die Datenmuster, die das Modell aus den Daten erlernt (in unserem Beispiel `x = np.array([…])` und `y = np.array([…])`), die tatsächlichen Beziehungen in diesen Daten widerspiegeln. Mit diesem erlernten Modell kann man anschließend für einen bisher unbekannten x-Wert den jeweiligen y-Wert vorhersagen, wenn für x und y die gleichen Beziehungen wie für die Trainingsdaten gelten.  

### 5.1) Beispiel für ein TensorFlow-Regressionsmodell 

Der hier folgende Code beinhaltet die Trainingsdaten `x = np.array([…])` und `y = np.array([…])` sowie die erforderlichen TensorFlow-Funktionsaufrufe zur Modellbildung. Durch die Codeausführung in einer Colab-Zelle wird die Datei *my_model.h5* im Colab-Dateibereich erzeugt. Diese Datei bildet das neue Modell.

```python
# TensorFlow regression model example ...

import tensorflow as tf
import numpy as np
from tensorflow import keras

x = np.array([-1.0, 0.0, 1.0, 2.0,  3.0,  4.0], dtype=float)
y = np.array([-2.0, 1.0, 4.0, 7.0, 10.0, 13.0], dtype=float)

model = tf.keras.Sequential([keras.layers.Dense(units=1, input_shape=[1])])

model.compile(optimizer='sgd', loss='mean_squared_error')

history = model.fit(x, y, epochs=400)

# Save the Keras model to .h5 file ...
model.save("my_model.h5")
```

### 5.2) Lernkurve des Modells visualisieren 

Der eigentliche Lernvorgang zur Modellbildung einer Supervised Machine Learning-Anwendung erfolgt in einer Trainingsschleife. Dabei entstehen TensorFlow und Keras interne Daten zum Verlauf der Lernkurve (Training loss). Der folgende Code bewirkt die Darstellung eines Diagramms mit dem *Training loss*. 

```python
# ... Show the training loss as diagram

import matplotlib.pyplot as plt

loss = history.history['loss']
epochs = range(1, len(loss) + 1)

plt.plot(epochs, loss, 'bo', label='Training loss')
plt.title('Training loss')
plt.show()
```

### 5.3) Modellparameter ausgeben

Das Machine Learning-Modell wird in unserem Beispiel durch ein künstliches neuronales Netzwerk mit je einem Eingang und Ausgang gebildet. Es gibt für die lineare Regression `y = mx + b` genau zwei „lernfähige“ Parameter: *m* und *b*. Die Detailinformationen zum neuronalen Netz lassen sich mit dem folgenden Code in Textform ausgeben:

```python
# ... Show more model details

import tensorflow as tf
import numpy as np
from tensorflow import keras

model = keras.models.load_model("my_model.h5")
model.summary()
```

### 5.4) Modell unter TensorFlow für Vorhersagen nutzen 

Nachdem ein Machine-Learning-Modell vorliegt, lässt es sich für Vorhersagen nutzen (also, um für einen neuen x-Wert den jeweiligen y-Wert zu bestimmen. Der folgende Code bildet den erfoderlichen Inferenzbaustein:

```python
# ... Load model file and predict something

import tensorflow as tf
import numpy as np
from tensorflow import keras

model = keras.models.load_model("my_model.h5")
print(np.round(model.predict([3]), 1))
```
Der neue x-Wert ist in diesem Beispiel die *3* in der letzten Codezeile `print(np.round(model.predict([3]), 1))`. Sie können hier auch andere Werte eintragen und die Codsezelle unter Colab immer wieder ausführen.

### 5.5) Modell in TensorFlow Lite-Format konvertieren 

Um ein TensorFlow-Modell für die Inferenz in einer OT-Umgebung zu nutzen, ist auf dem Zielsystem auch eine vollständige TensorFlow-Laufzeitumgebung erforderlich. Falls Ihre OT-Hardware dafür nicht die erforderlichen Ressourcen bietet, können Sie das Modell in ein TensorFlow Lite-Modell umwandeln. Der folgende Code führt diese Umwandlung aus: Die Datei *my_model.h5* wird TensorFlow Lite-Modell mit dem Namen *my_model.tflite* konvertiert.

```python
# TensorFlow Lite: Load model file and convert model to *.tflite ...

import tensorflow as tf
import numpy as np
from tensorflow import keras

model = keras.models.load_model("my_model.h5")

# Convert the Keras model to .tflite file ...
converter = tf.lite.TFLiteConverter.from_keras_model(model)
#converter.optimizations = [tf.lite.Optimize.OPTIMIZE_FOR_SIZE]
#converter.optimizations = [tf.lite.Optimize.DEFAULT]
tflite_model = converter.convert()
open('my_model.tflite', 'wb').write(tflite_model)
```

### 5.6) TensorFlow Lite-Interpreter für Vorhersagen nutzen 

Eine Inferenzmaschine, die TensorFlow Lite-Modelle nutzt (also z. B. Dateien im *tflite*-Format), benötigt einen sogenannten Interpreter. Der hier folgende Code dient als Beispiel für einen solchen TensorFlow Lite-Interpreter.

```python
# TensorFlow Lite: load *.tflite model file and predict something ...

import numpy as np
import tensorflow as tf

# Load TFLite model and allocate tensors.
interpreter = tf.lite.Interpreter(model_path="my_model.tflite")
interpreter.allocate_tensors()

# Get input and output tensors.
input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()

# Use model with input data.
input_shape = input_details[0]['shape']
input_data = np.array([[3.0]], dtype=np.float32)
interpreter.set_tensor(input_details[0]['index'], input_data)

interpreter.invoke()

# The function `get_tensor()` returns a copy of the tensor data.
# Use `tensor()` in order to get a pointer to the tensor.
output_data = interpreter.get_tensor(output_details[0]['index'])
print(np.round(output_data, 1))
```

### 5.7) Trainingsdaten für weitere Regressionsmodelle 

Die Trainingsdaten `x = np.array([…])` und `y = np.array([…])` unter *5.1 Beispiel für ein TensorFlow-Regressionsmodell* können Sie gegen eines der beiden folgenden Beispiele austauschen, um anschließend ein neues TensorFlow-Modell zu erzeugen.

```python
# More data to learn (y = mx + b)

x = np.array([-1.0, 0.0, 1.0, 2.0, 3.0, 4.0,  5.0,  6.0], dtype=float)
y = np.array([-0.5, 1.5, 3.5, 5.5, 7.5, 9.5, 11.5, 13.5], dtype=float)

x = np.array([-1.0, 0.0,  1.0, 2.0,  3.0, 4.0,  5.0, 6.0], dtype=float)
y = np.array([0.75, 1.0, 1.25, 1.5, 1.75, 2.0, 2.25, 2.5], dtype=float)
```

### 5.8) Embedded in eine HMTM-Seite per TensorFlow.js 

Die ...

```html
<html>
   <head>
      <title>TensorFlow Regression</title>
      <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@0.6.1"></script>
      <script>
         var tfinterface;
         const model = tf.sequential();
             
         function initTF() {
         
         // 1. Training data
               // y = 10x
               //const xs = tf.tensor2d([1,2,3,4,5,6,7,8,9], [9,1]);
               //const ys = tf.tensor2d([10,20,30,40,50,60,70,80,90], [9,1]);
         
               // y = 3x + 1
               const xs = tf.tensor2d([-1.0, 0.0, 1.0, 2.0,  3.0,  4.0], [6,1])
               const ys = tf.tensor2d([-2.0, 1.0, 4.0, 7.0, 10.0, 13.0], [6,1])
      
         // 2. Build linear regression model
               model.add(tf.layers.dense({units: 1, inputShape: [1]}));
               model.compile({loss: 'meanSquaredError', optimizer: 'sgd'}); 
               tfinterface=model.fit(xs,ys,{epochs: 400});
         }
         
         // 3. Prediction wrapper
         function predict(n) {
               return tfinterface.then(()=> { return model.predict(tf.tensor2d([n],[1,1])); });
         }
         
         // Helper for HTML form
         function formpredict(v,r) {
            predict(v).then(function(res) {
            r.innerHTML=res.get([0]);
            });  
         }
         
            
      </script>      
   </head>  
   <body onLoad='initTF();'>
      <p>
      <div id='res'>Wait</div>
      <p>
      <form name='iForm' onSubmit='formpredict(this.val.value,document.getElementById("res")); return false;')>
         <script>
            document.getElementById("res").innerHTML="&nbsp;";
                 
         </script>
         Input Number: <input name='val'><input type=submit>
      </form>
   </body>
</html>
```

## Was ist ein PyDSlog-Docker?

Der Maschinensensor MLS/160A liefert ein relativ komplexes Datenbild mit verschiedenen Kanälen (bis zu sechs Achsen mit Beschleunigung und Winkelgeschwindigkeit) sowie einstellbarer Abtastfrequenz innerhalb eines definierbaren Zeitfensters. Als Zubehör ist nun ein MLS/160A-Support Docker verfügbar.

![Node-RED-Flow](https://ssv-comm.de/forum/bilder/py_230421.png) 

Mit diesen PyDSlog Docker wollen wir die Datenqualitätsanforderungen einer Machine-Learning-Applikation so präzise wie möglich umsetzen und Standardschnittstellen zu anderen Anwendungen schaffen. Der Docker unterstützt nicht nur die Sensorkonfiguration. Er dient auch zum sicheren Remote Update der MLS/160A-Firmware. Ein PyDSlog Docker ist auf einem SSV-Gateway und auf anderen geeigneten Plattformen in der Edge einsetzbar.

<a href="https://github.com/kdw1000/Test/blob/master/_161120.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>

<a href="https://github.com/kdw1000/Test/blob/master/_250321.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>
