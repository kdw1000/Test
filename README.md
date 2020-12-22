### Wie funktioniert SDU (Secure Device Update)?

Eine SDU-Anwendung soll eine Komponente (Device) hinter dem RMG/941 über eine IoT-Anbindung mit Software-, Konfigurations- und Machine-Learning-Modell-Updates versorgen. Dabei liegt der Schwerpunkt, neben der Remote-Update-Funktion, im Bereich der IT-Sicherheit. Um eine dem Stand der Technik entsprechende Sicherheit zu gewährleisten, nutzt SDU eine Public-Key-Infrastruktur (PKI) für digitale Signaturen mit privaten und öffentlichen Schlüsseln, Zertifikaten und einer Sperrliste (Blacklist).

![Übersicht: Zusammenhänge SDU ](https://ssv-comm.de/forum/bilder/sdu_141220.png) 

Der gesamte Update-Prozess wurde für den vollautomatischen Betrieb entwickelt und lässt sich in Bezug auf die Device-Anzahl nahezu beliebig skalieren. Die Abbildung zeigt die wichtigsten SDU-Funktionsbausteine.

**1:** SDU Server. Dieser Serverprozess läuft im Internet und dient für die Weitergabe der Updates als Bindeglied zwischen dem PC des Softwareentwickler (bzw. zuständigen Maintainer) und der Device hinter dem RMG/941. Ein SDU Server wird als Docker Container Image ausgeliefert und ist daher nahezu in jeder geeigneten Docker-Laufzeitumgebung ausführbar.

**2:** RMG/941 mit SDU Agent und SDU Gateway Update Client (SDU GUC). Der SDU GUC ist für die Kommunikation mit dem SDU Server zuständig. Er sorgt dafür, dass eine gültige Update-Datei bei Bedarf vom Server zum RMG/941 heruntergeladen und dem SDU Agent zur Verfügung gestellt wird. Ein anwendungsbezogener SDU Agent übernimmt die Kommunikation mit der jeweiligen Device hinter dem RMG/941, z. B. um die Update-Datei an die Device zu übertragen. Der SDU Agent muss daher projektspezifisch an das Protokoll der jeweiligen Device angepasst werden. Beide Softwarekomponenten werden in der Regel von SSV werksseitig vorinstalliert und für eine bestimmte Anwendung konfiguriert.

**3:** Entwickler-PC mit SDU Signing Tool und SDU Maintainer Update Client (SDU MUC). Mit dem Signing Tool werden die für den Update erforderlichen einzelnen Dateien auf dem PC zu einer SDU-Update-Datei zusammengefügt und mit einer digitalen Unterschrift (digital Signature) versehen. Mit Hilfe des SDU MUC wird diese signierte Datei dann zum SDU Server hochgeladen. Sowohl das SDU Signing Tool als auch der SDU Maintainer Update Client wurden als Electron-Cross-Plattform-Desktop-Anwendung entwickelt. Sie lassen sich auf Linux-, macOS- und Windows-Rechnern ausführen. 

**Device:** Das eigentliche Zielsystem für den Update. Es ist per CAN, Modbus (RTU, TCP), Nahbereichs-Funkschnittstelle usw. mit dem RMG/941 verbunden. Über den RMG/941-SDU-Agent wurde das Gateway an das jeweilige Update-Protokoll der Device angepasst ...

<a href="https://github.com/kdw1000/Test/blob/master/_161120.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>
