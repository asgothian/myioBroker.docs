| titel | last change | from |
| -------- | -------- | -------- |
| MQTT | 01.02.2022 | [@hydrotec](https://forum.iobroker.net/user/hydrotec) |

# MQTT

## Inhalt
* [Allgemein](#Allgemein)
* [Protokoll](#Protokoll)
* [Adapter](#Adapter)
* [Best Practice / Tutorial](#best-practice--tutorial)
* [Issues](#Issues)

## Allgemein
Bei MQTT (Message Queuing Telemetry Transport)
handelt es sich um ein Netzwerkprotokoll, welches offen kommuniziert,
und auch bei minimaler Netzwerkbandbreite eingesetzt werden kann.  
MQTT eignet sich sehr gut für das Internet der Dinge (IoT).
In Fachkreisen spricht man auch von  Machine-to-Machine-Kommunikation (M2M).  
Es werden Daten (payloads) von Client zu Client über einen Broker ausgetauscht.  
Ausführliche Informationen können unter den folgenden Links aufgerufen werden.  

[<https://mqtt.org/>](https://mqtt.org/)  
[<https://de.wikipedia.org/wiki/MQTT>](https://de.wikipedia.org/wiki/MQTT)  

###### [zurück](#Inhalt)

## Protokoll

?> **Hinweis**:  
     In diesem Abschnitt wird nur auf den gängigsten Weg,
	 MQTT zu nutzen, eingegangen.  
	 Es bestehen viel mehr Möglichkeiten, wie man MQTT nutzen kann.  
	 Hier alle Möglichkeiten aufzuzählen,
	 würde den Rahmen dieser Dokumentation überschreiten.  


![](https://raw.githubusercontent.com/hydrotec468/test-md.docs/main/docs/de/media/Doku_mqtt_03.png)  

Der MQTT-Broker ist die zentrale Anlaufstelle für die MQTT-Clients.
Ein Client muss an dem Broker angemeldet sein, damit dieser seine *Payloads*,
über die *Topics*, dem Broker mitteilen (*Publish*),
oder sie empfangen (*Subscribe*) kann.
Der Broker akzeptiert erst einmal alle gesendeten *Payloads*,
in den entsprechenden *Topics* der Clients,
und stellt sie weiteren Clients zur Verfügung.
Damit ein Client, den eines anderen Clients zur Verfügung gestellten *Payload*
empfangen kann, muss er zuerst den zugehörigen *Topic* abonnieren.  

### Begriffe

**Topic**

Was steckt hinter den Begriffen *Topic*, *Payload* und *abonnieren*?  
Unter *Topic* kann man sich eine Art Verzeichnisstruktur vorstellen.
Diese Struktur wird üblicherweise vom Client vorgegeben. Im Fall von Geräten werden die Vorgaben daher meistens von den Herstellern gemacht. In Situationen wo der Anwender die Kommunikation zwischen mehreren Clients selber definiert können Anwenderdefinierte *Topics* vorgegeben werden.
Es ist Üblich im Topic eine Strukturierung vorzugeben die auf der einen Seite eine Identifikation von vergleichbaren *Payloads* erlaubt, auf der anderen Seite aber auch eine klare Zuordnung zu den einzelnen Devices gewährleistet.  

Beispiel :
`<Rubrik>/<Gerät>/<Sparte>/<Attribut>`  

**Payload**

Der *Payload* bezeichnet den Inhalt der Nachricht. Es gibt für den *Payload* keine festen Vorgaben wie dieser struktuiert werden soo, es haben sich aber verschiedene Methoden etabliert die öfter zu finden sind.
Bei strukturiertem Payload liegt zumeist eine Formatierung in *JSON* vor, so das in einem *Payload* mehrere Informationen übergeben werden können.
Unstrukturierter Payload ist üblicherweise einfach eine Zeichenkette die vom *Subscriber* der Nachricht ausgewertet werden muss.

Beispiele für verschiedene Payloads sind

- Format *JSON*: `{"state": "online"}`  
- Format *atribute*:  `online`  

**Subscription**

Damit der Broker weiß welche Nachrichten ein Client bekommen möchte
muss der Client die Nachrichten anfordern (*abonnieren*).
Das geschieht durch die Übermittlung des gewünschten *Topic* in dem Kanal  *Subscribe*. Bei der Bezeichnung des *Topics* sind Wildcards erlaubt so das mit einer Nachricht vom Client eine Gruppe von *Topics* *subscribed* werden können. Jedes mal wenn der Broker eine Nachricht zu einem dieser *Topics* erhält leitet er dieses and den entsprechenden Client weiter.
Die zwei gängigsten Wildcards sind ***#*** und ***+***. Die Wildcard ***#*** steht dabei für beliebige Topics unterhalb des fest vorgegebenen Anteils, während ***+*** eine einzelne Ebene im Topic markiert. Dieses ist am Besten an einigen Beispielen zu erklären:  

Angenommen, der Client 'MultiSensor1' *publisht* mehrere *Topics* nach dem folgenden Muster:
```
myhome/MultiSensor1/sensors/temperature
myhome/MultiSensor1/sensors/humidity
myhome/MultiSensor1/sensors/pressure
myhome/MultiSensor1/status
```  

In diesem Fall kann man durch den geschickten Einsatz von Wildcards mehrere *Topics* auf ein mal abonnieren.  

Um die Werte aller Sensoren
welche unter "myhome/MultiSensor1/sensors" gesendet werden zu empfangen reicht ein *subscribe* mit `myhome/MultiSensor1/sensors/#`  

Um alle oben angegebenen *Topics* zu abonnieren würde `myhome/MultiSensor1/#` ausreichen.  

Für den Fall das unter 'myhome' noch weitere *Topics* existieren können diese alle über `myhome/#` abonniert werden.

Um nur die Temperaturwerte von mehreren Multisensoren die nach dem oben angegebenen Schema ihre Werte *publishen* zu erhalten ist ein *subscribe* mit `myhome/+/sensors/temperature` der einfachste Weg.

Auch eine Kombination von ***#*** und ***+*** ist möglich. So bewirkt das subscription Topic `myhome/+/sensors/#` das alle Werte (Temperatur, Luftfeuchtigkeit, Luftdruck) von allen vorhandenen Sensoren abonniert werden.       

Der Aufbau der *Topics* und die Verwendung von Wildcards
wie sie innerhalb des Adapters verwendet werden
ist in der jeweiligen Adapterreferenz beschrieben.  

Zum besseren Verständnis, sind unter [Best Practice / Tutorial](#best-practice--tutorial)
ein paar Beispiele aufgeführt.  

###### [zurück](#Inhalt)

## Adapter
Aktuell stehen in ioBroker drei Adapter unter dem Repository "*default*",
und vier Adapter unter dem Repository "*latest*" zur Verfügung.
Mit diesen Adaptern können Geräte, welche das *MQTT-Protokoll* sprechen,
zu ioBroker angebunden werden können.  
![](https://raw.githubusercontent.com/hydrotec468/test-md.docs/main/docs/de/media/Doku_mqtt_01.png)  

1.) [hass.mqtt](https://github.com/smarthomefans/ioBroker.hass-mqtt#readme "https://github.com/smarthomefans/ioBroker.hass-mqtt#readme")  

| **Repository** | [![](http://iobroker.live/badges/hass-mqtt-stable.svg) ![](http://img.shields.io/npm/v/iobroker.hass-mqtt.svg)](https://www.npmjs.com/package/iobroker.hass-mqtt "https://www.npmjs.com/package/iobroker.hass-mqtt")|
| -------- | -------- |
| **Code frequency** | [GitHub Insights](https://github.com/smarthomefans/ioBroker.hass-mqtt/graphs/code-frequency "https://github.com/smarthomefans/ioBroker.hass-mqtt/graphs/code-frequency") |

 -  wird in Verbindung zu *Home Assistant* genutzt  

2.) [Sonoff](https://github.com/ioBroker/ioBroker.sonoff#readme "https://github.com/ioBroker/ioBroker.sonoff#readme")

| **Repository** | [![](http://iobroker.live/badges/sonoff-stable.svg) ![](http://img.shields.io/npm/v/iobroker.sonoff.svg)](https://www.npmjs.com/package/iobroker.sonoff "https://www.npmjs.com/package/iobroker.sonoff")|
| -------- | -------- |
| **Code frequency** | [GitHub Insights](https://github.com/ioBroker/ioBroker.sonoff/graphs/code-frequency "https://github.com/ioBroker/ioBroker.sonoff/graphs/code-frequency") |

 -  wird in Verbindung zu *Tasmota* und *ESP* genutzt  

3.) [MQTT Broker/Client](https://github.com/ioBroker/ioBroker.mqtt#readme "https://github.com/ioBroker/ioBroker.mqtt#readme")

| **Repository** | [![](http://iobroker.live/badges/mqtt-stable.svg) ![](http://img.shields.io/npm/v/iobroker.mqtt.svg)](https://www.npmjs.com/package/iobroker.mqtt "https://www.npmjs.com/package/iobroker.mqtt")|
| -------- | -------- |
| **Code frequency** | [GitHub Insights](https://github.com/ioBroker/ioBroker.mqtt/graphs/code-frequency "https://github.com/ioBroker/ioBroker.mqtt/graphs/code-frequency") |

 - kann sowohl als *Client*, wie auch als *Broker* konfiguriert werden  
 - in der *Client* Version, wird ein externer *Broker* (z.B. [mosquitto](https://mosquitto.org "https://mosquitto.org") ) vorausgesetzt  
 - spricht grundsätzlich mit allen Geräten, welche über das MQTT-Protokoll kommunizieren  

4.) [MQTT-Client](https://github.com/Pmant/ioBroker.mqtt-client#readme "https://github.com/Pmant/ioBroker.mqtt-client#readme")

| **Repository** | [![](http://iobroker.live/badges/mqtt-client-stable.svg) ![](http://img.shields.io/npm/v/iobroker.mqtt-client.svg)](https://www.npmjs.com/package/iobroker.mqtt-client "https://www.npmjs.com/package/iobroker.mqtt-client")|
| -------- | -------- |
| **Code frequency** | [GitHub Insights](https://github.com/Pmant/ioBroker.mqtt-client/graphs/code-frequency "https://github.com/Pmant/ioBroker.mqtt-client/graphs/code-frequency") |

 - reiner MQTT Client  
 - es wird ein externer *Broker* (z.B. [mosquitto](https://mosquitto.org "https://mosquitto.org") ) vorausgesetzt  
 - spricht grundsätzlich mit allen Geräten, welche über das MQTT-Protokoll kommunizieren  

?> todo:  
 - Hilfestellung wann welcher Adapter verwendet werden sollte  
 - Entscheidungsmatrix wann mqtt als Server, wann als client  

###### [zurück](#Inhalt)

## Best Practice / Tutorial
?> todo:  
 - Beispiel Konfiguration Server/Client  
 - Eindeutige IDs verwenden
 - Konfiguration des Ports beachten  
 - Tools erwähnen  
 - eventuell Link zu einem Tutorial  

###### [zurück](#Inhalt)

## Issues
?> todo:  
 - auf derzeitige Probleme hinweisen  

###### [zurück](#Inhalt)
