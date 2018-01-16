---
title: Maus als Sensor
layout: post
permalink: maus-als-sensor
published: true
---
Jeder hat sie rumliegen. Die typische optische Maus, die keiner mehr brauchen kann, weil man davon droelfzig Stueck hat.

Was waere, wenn man sie fuer eigene kleine Bastelprojekte gebrauchen kann, immerhin hat sie 3 Taster, ein Mausrad und einen Bewegungssensor.
Wie man die einzelnen Sensoren an einem PC, oder Raspberry Pie, mit Python auslesen kann, will ich genaueren erlaeutern.

# Die typische Maus

![Mouse](/content/images/2016/04/IMGP1287.JPG)
Ich verwende hier ein 1.5 Euro Maus aus China, da diese leicht und billig zu bekommen ist. Dies geht natuerlich auch mit jeder anderen, die von euch ausgesondert werden kann.

Nach einem aufschrauben und zerlegen wird euch vielleicht eine etwas andere Platine ins Auge springen, aber trotzdem wird sie aehnlich aufgebaut sein.
![Platine](/content/images/2016/04/IMGP1291.JPG)
Es gibt insgesamt 3 Taster (SW1-3), ein Mausrad, 2 LEDs, und einen grossen Chip mit einer optischen Erkennung.
Das schoene an der Maus: es wertet alle Komponenten aus und bringt sie in eine Form, die wir einfach ueber USB auslesen koennen.

Beim Anschliessen dieser Maus passieren bei einem typischen Linux PC mit X ein paar Dinge: es wird ein Device registriert und dieses wird als Input angenommen. Dann kann man seinen Mauszeiger bewegen und rumklicken. Das wollen wir aber an dem Punkt garnicht. Wir wollen selber die Taster auswerten und direkt auf das USB Geraet zugreifen.

Ich habe da mal ein wenig vorgearbeitet und ein kleines Python Skript geschrieben, welches ueber pyUSB auf die Maus zugreift.

```python
import sys
import time
import usb.core
import usb.util

device = usb.core.find(idVendor=0x0000,  idProduct=0x0538)
if device is None:
    print("Error connecting")
    sys.exit(1)
else:
    if device.is_kernel_driver_active(0):
        device.detach_kernel_driver(0)
        usb.util.claim_interface(device,0)
    device.reset()
    device.set_configuration()


while(True):
    try:
        n = [int(i) for i in device.read(0x81,6)]
        buttons = [
            True if n[1]&1 != 0 else False,
            True if n[1]&2 != 0 else False,
            True if n[1]&4 != 0 else False,
        ]
        print(buttons)
        time.sleep(.5)
    except KeyboardInterrupt:
        sys.exit()
    except:
        pass
```
Das Skript waehlt ein USB Geraet aus durch eine Vendor- und ProductID. Daraufhin detached es alle Kerneltreiber, damit es das Geraet komplett fuer sich alleine hat.
Durch Auslesen des USB-Endpoints 0x81 mit genau 6 Bytes sehen wir die komplette Kommunikation zwischen Maus und Skript. Das 2. Byte in dem resultierenden Byte Array enthaelt die Taster. Wobei die Taster hier auf einzelne Bits kodiert sind.
```
[1, 1, 0, 0, 0, 0] <- Button 1
[1, 2, 0, 0, 0, 0] <- Button 2
[1, 4, 0, 0, 0, 0] <- Button 3
[1, 3, 0, 0, 0, 0] <- Button 1,2
[1, 6, 0, 0, 0, 0] <- Button 2,3
[1, 7, 0, 0, 0, 0] <- Button 1,2,3
```
Das letzte Byte beinhaltet das Mausrad. 1 ist die eine und 255 die andere Richtung. Dadurch koennen wir auf eine einfache Art und Weise Drehbewegungen auslesen.
```
[1, 0, 0, 0, 0, 1]
[1, 0, 0, 0, 0, 255] 
```

Daten werden immer dann uebertragen, wenn eine Aenderung passiert. Bloederweise passieren die Recht haeufig, da wir einen optischen Sensor haben, deshalb hab ich ein sleep eingebaut um die CPU nicht zu sehr zu belasten. Man kann sich ueberlegen den Sensor zu ueberkleben. Vielleicht kann man ihn auch direkt Deaktivieren. Das wird aber nicht bei allen Maeusen identisch sein.

so long

