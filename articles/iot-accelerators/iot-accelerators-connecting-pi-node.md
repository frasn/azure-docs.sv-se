---
title: Etablera Raspberry Pi till Fjärrövervaknings i Node.js - Azure | Microsoft Docs
description: Beskriver hur du ansluter en hallon Pi-enhet till Fjärrövervaknings solution accelerator med hjälp av ett program som skrivits i Node.js.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 01/24/2018
ms.author: dobett
ms.openlocfilehash: 78647612fc747ec328279536d82fb31bb4858688
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34626878"
---
# <a name="connect-your-raspberry-pi-device-to-the-remote-monitoring-solution-accelerator-nodejs"></a>Ansluta enheten hallon Pi till Fjärrövervaknings solution accelerator (Node.js)

[!INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

Den här kursen visar hur du ansluter en fysisk enhet till Fjärrövervaknings solution accelerator. I den här kursen använder du Node.js, vilket är ett bra alternativ för miljöer med minimal resurs begränsningar.

### <a name="required-hardware"></a>Nödvändig maskinvara

En stationär dator så att du kan fjärransluta till kommandoraden på hallon Pi.

[Microsoft IoT startpaket för hallon Pi 3](https://azure.microsoft.com/develop/iot/starter-kits/) eller motsvarande komponenter. Den här kursen använder följande objekt från kit:

- Raspberry Pi 3
- MicroSD-kort (med NOOBS)
- En Mini USB-kabel
- En Ethernet-kabel

### <a name="required-desktop-software"></a>Nödvändig programvara för skrivbordet

Du måste SSH-klienten på den stationära datorn så att du kan fjärransluta till kommandoraden på hallon Pi.

- Windows innehåller inte en SSH-klient. Vi rekommenderar att du använder [PuTTY](http://www.putty.org/).
- De flesta distributioner för Linux och Mac OS inkluderar SSH-kommandoradsverktyget. Mer information finns i [SSH med Linux- eller Mac OS x](https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md).

### <a name="required-raspberry-pi-software"></a>Programvara som krävs hallon Pi

Om du inte redan gjort det, installera Node.js version 4.0.0 eller senare på din hallon Pi. Följande steg visar hur du installerar Node.js v6 på din hallon Pi:

1. Anslut till din hallon Pi med `ssh`. Mer information finns i [SSH (Secure Shell)](https://www.raspberrypi.org/documentation/remote-access/ssh/README.md) på den [hallon Pi webbplats](https://www.raspberrypi.org/).

1. Använd följande kommando för att uppdatera din hallon Pi:

    ```sh
    sudo apt-get update
    ```

1. Använd följande kommandon för att ta bort en befintlig installation av Node.js från din hallon Pi:

    ```sh
    sudo apt-get remove nodered -y
    sudo apt-get remove nodejs nodejs-legacy -y
    sudo apt-get remove npm  -y
    ```

1. Använd följande kommando för att hämta och installera Node.js v6 på din hallon Pi:

    ```sh
    curl -sL https://deb.nodesource.com/setup_6.x | sudo bash -
    sudo apt-get install nodejs -y
    ```

1. Använd följande kommando för att verifiera att du har installerat Node.js v6.11.4 har:

    ```sh
    node --version
    ```

## <a name="create-a-nodejs-solution"></a>Skapa en Node.js-lösning

Utför följande steg med hjälp av den `ssh` anslutning till din hallon Pi:

1. Skapa en mapp med namnet `remotemonitoring` i arbetsmappen på hallon Pi. Navigera till den här mappen i kommandoraden:

    ```sh
    cd ~
    mkdir remotemonitoring
    cd remotemonitoring
    ```

1. Om du vill hämta och installera de paket som du behöver utföra sample-appen genom att köra följande kommandon:

    ```sh
    npm init
    npm install async azure-iot-device azure-iot-device-mqtt --save
    ```

1. I den `remotemonitoring` mapp, skapa en fil med namnet **remote_monitoring.js**. Öppna den här filen i en textredigerare. På hallon Pi, kan du använda den `nano` eller `vi` textredigerare.

1. I den **remote_monitoring.js** fil, lägger du till följande `require` instruktioner:

    ```nodejs
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    var Client = require('azure-iot-device').Client;
    var ConnectionString = require('azure-iot-device').ConnectionString;
    var Message = require('azure-iot-device').Message;
    var async = require('async');
    ```

1. Lägg till följande variabeldeklarationer efter `require`-instruktionerna. Ersätt platshållaren värdet `{device connection string}` med värdet som du antecknade för enheten etableras i Fjärrövervaknings-lösningen:

    ```nodejs
    var connectionString = '{device connection string}';
    var deviceId = ConnectionString.parse(connectionString).DeviceId;
    ```

1. Om du vill definiera vissa grundläggande telemetridata, lägger du till följande variabler:

    ```nodejs
    var temperature = 50;
    var temperatureUnit = 'F';
    var humidity = 50;
    var humidityUnit = '%';
    var pressure = 55;
    var pressureUnit = 'psig';
    ```

1. Om du vill definiera vissa egenskapsvärden, lägger du till följande variabler:

    ```nodejs
    var temperatureSchema = 'chiller-temperature;v1';
    var humiditySchema = 'chiller-humidity;v1';
    var pressureSchema = 'chiller-pressure;v1';
    var interval = "00:00:05";
    var deviceType = "Chiller";
    var deviceFirmware = "1.0.0";
    var deviceFirmwareUpdateStatus = "";
    var deviceLocation = "Building 44";
    var deviceLatitude = 47.638928;
    var deviceLongitude = -122.13476;
    var deviceOnline = true;
    ```

1. Lägg till följande variabel om du vill definiera egenskaperna rapporterade att skicka till lösningen. Dessa egenskaper innehåller metadata som beskriver metoderna och telemetri enheten använder:

    ```nodejs
    var reportedProperties = {
      "Protocol": "MQTT",
      "SupportedMethods": "Reboot,FirmwareUpdate,EmergencyValveRelease,IncreasePressure",
      "Telemetry": {
        "TemperatureSchema": {
          "Interval": interval,
          "MessageTemplate": "{\"temperature\":${temperature},\"temperature_unit\":\"${temperature_unit}\"}",
          "MessageSchema": {
            "Name": temperatureSchema,
            "Format": "JSON",
            "Fields": {
              "temperature": "Double",
              "temperature_unit": "Text"
            }
          }
        },
        "HumiditySchema": {
          "Interval": interval,
          "MessageTemplate": "{\"humidity\":${humidity},\"humidity_unit\":\"${humidity_unit}\"}",
          "MessageSchema": {
            "Name": humiditySchema,
            "Format": "JSON",
            "Fields": {
              "humidity": "Double",
              "humidity_unit": "Text"
            }
          }
        },
        "PressureSchema": {
          "Interval": interval,
          "MessageTemplate": "{\"pressure\":${pressure},\"pressure_unit\":\"${pressure_unit}\"}",
          "MessageSchema": {
            "Name": pressureSchema,
            "Format": "JSON",
            "Fields": {
              "pressure": "Double",
              "pressure_unit": "Text"
            }
          }
        }
      },
      "Type": deviceType,
      "Firmware": deviceFirmware,
      "FirmwareUpdateStatus": deviceFirmwareUpdateStatus,
      "Location": deviceLocation,
      "Latitude": deviceLatitude,
      "Longitude": deviceLongitude,
      "Online": deviceOnline
    }
    ```

1. Om du vill skriva ut resultatet av åtgärden, lägger du till följande hjälpfunktion:

    ```nodejs
    function printErrorFor(op) {
        return function printError(err) {
            if (err) console.log(op + ' error: ' + err.toString());
        };
    }
    ```

1. Lägg till följande hjälpfunktion du använder för att Slumpa telemetri värden:

    ```nodejs
    function generateRandomIncrement() {
        return ((Math.random() * 2) - 1);
    }
    ```

1. Lägg till följande allmänna funktion för att hantera direkt metodanrop från lösningen. Funktionen visar information om den direkta metoden som anropades, men i det här exemplet ändrar inte enheten på något sätt. Lösningen använder direkta metoder för att fungera på enheter:

    ```nodejs
    function onDirectMethod(request, response) {
      // Implement logic asynchronously here.
      console.log('Simulated ' + request.methodName);

      // Complete the response
      response.send(200, request.methodName + ' was called on the device', function (err) {
        if (err) console.error('Error sending method response :\n' + err.toString());
        else console.log('200 Response to method \'' + request.methodName + '\' sent successfully.');
      });
    }
    ```

1. Lägg till följande funktion att hantera den **FirmwareUpdate** direkt metodanrop från lösningen. Funktionen verifierar parametrar i nyttolasten direkta metoden och kör sedan en firmware-uppdatering simulering asynkront:

    ```node.js
    function onFirmwareUpdate(request, response) {
      // Get the requested firmware version from the JSON request body
      var firmwareVersion = request.payload.Firmware;
      var firmwareUri = request.payload.FirmwareUri;
      
      // Ensure we got a firmware values
      if (!firmwareVersion || !firmwareUri) {
        response.send(400, 'Missing firmware value', function(err) {
          if (err) console.error('Error sending method response :\n' + err.toString());
          else console.log('400 Response to method \'' + request.methodName + '\' sent successfully.');
        });
      } else {
        // Respond the cloud app for the device method
        response.send(200, 'Firmware update started.', function(err) {
          if (err) console.error('Error sending method response :\n' + err.toString());
          else {
            console.log('200 Response to method \'' + request.methodName + '\' sent successfully.');

            // Run the simulated firmware update flow
            runFirmwareUpdateFlow(firmwareVersion, firmwareUri);
          }
        });
      }
    }
    ```

1. Lägg till följande funktion för att simulera en tidskrävande firmware update flöde som rapporterar förlopp tillbaka till lösningen:

    ```node.js
    // Simulated firmwareUpdate flow
    function runFirmwareUpdateFlow(firmwareVersion, firmwareUri) {
      console.log('Simulating firmware update flow...');
      console.log('> Firmware version passed: ' + firmwareVersion);
      console.log('> Firmware URI passed: ' + firmwareUri);
      async.waterfall([
        function (callback) {
          console.log("Image downloading from " + firmwareUri);
          var patch = {
            FirmwareUpdateStatus: 'Downloading image..'
          };
          reportUpdateThroughTwin(patch, callback);
          sleep(10000, callback);
        },
        function (callback) {
          console.log("Downloaded, applying firmware " + firmwareVersion);
          deviceOnline = false;
          var patch = {
            FirmwareUpdateStatus: 'Applying firmware..',
            Online: false
          };
          reportUpdateThroughTwin(patch, callback);
          sleep(8000, callback);
        },
        function (callback) {
          console.log("Rebooting");
          var patch = {
            FirmwareUpdateStatus: 'Rebooting..'
          };
          reportUpdateThroughTwin(patch, callback);
          sleep(10000, callback);
        },
        function (callback) {
          console.log("Firmware updated to " + firmwareVersion);
          deviceOnline = true;
          var patch = {
            FirmwareUpdateStatus: 'Firmware updated',
            Online: true,
            Firmware: firmwareVersion
          };
          reportUpdateThroughTwin(patch, callback);
          callback(null);
        }
      ], function(err) {
        if (err) {
          console.error('Error in simulated firmware update flow: ' + err.message);
        } else {
          console.log("Completed simulated firmware update flow");
        }
      });

      // Helper function to update the twin reported properties.
      function reportUpdateThroughTwin(patch, callback) {
        console.log("Sending...");
        console.log(JSON.stringify(patch, null, 2));
        client.getTwin(function(err, twin) {
          if (!err) {
            twin.properties.reported.update(patch, function(err) {
              if (err) callback(err);
            });      
          } else {
            if (err) callback(err);
          }
        });
      }

      function sleep(milliseconds, callback) {
        console.log("Simulate a delay (milleseconds): " + milliseconds);
        setTimeout(function () {
          callback(null);
        }, milliseconds);
      }
    }
    ```

1. Lägg till följande kod för att skicka telemetridata till lösningen. Klientappen lägger till egenskaper i meddelandet för att identifiera meddelandet schemat:

    ```node.js
    function sendTelemetry(data, schema) {
      if (deviceOnline) {
        var d = new Date();
        var payload = JSON.stringify(data);
        var message = new Message(payload);
        message.properties.add('$$CreationTimeUtc', d.toISOString());
        message.properties.add('$$MessageSchema', schema);
        message.properties.add('$$ContentType', 'JSON');

        console.log('Sending device message data:\n' + payload);
        client.sendEvent(message, printErrorFor('send event'));
      } else {
        console.log('Offline, not sending telemetry');
      }
    }
    ```

1. Lägg till följande kod för att skapa en klientinstans:

    ```nodejs
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```

1. Lägg till följande kod:

    * Öppna en anslutning.
    * Ställ in en hanterare för egenskaper.
    * Skicka rapporterat egenskaper.
    * Registrera hanterare för direkta metoder. En separat hanterare används för firmware-uppdateringsmetoden direkt.
    * Börja skicka telemetri.

    ```nodejs
    client.open(function (err) {
      if (err) {
        printErrorFor('open')(err);
      } else {
        // Create device Twin
        client.getTwin(function (err, twin) {
          if (err) {
            console.error('Could not get device twin');
          } else {
            console.log('Device twin created');

            twin.on('properties.desired', function (delta) {
              // Handle desired properties set by solution
              console.log('Received new desired properties:');
              console.log(JSON.stringify(delta));
            });

            // Send reported properties
            twin.properties.reported.update(reportedProperties, function (err) {
              if (err) throw err;
              console.log('Twin state reported');
            });

            // Register handlers for all the method names we are interested in.
            // Consider separate handlers for each method.
            client.onDeviceMethod('Reboot', onDirectMethod);
            client.onDeviceMethod('FirmwareUpdate', onFirmwareUpdate);
            client.onDeviceMethod('EmergencyValveRelease', onDirectMethod);
            client.onDeviceMethod('IncreasePressure', onDirectMethod);
          }
        });

        // Start sending telemetry
        var sendTemperatureInterval = setInterval(function () {
          temperature += generateRandomIncrement();
          var data = {
            'temperature': temperature,
            'temperature_unit': temperatureUnit
          };
          sendTelemetry(data, temperatureSchema)
        }, 5000);

        var sendHumidityInterval = setInterval(function () {
          humidity += generateRandomIncrement();
          var data = {
            'humidity': humidity,
            'humidity_unit': humidityUnit
          };
          sendTelemetry(data, humiditySchema)
        }, 5000);

        var sendPressureInterval = setInterval(function () {
          pressure += generateRandomIncrement();
          var data = {
            'pressure': pressure,
            'pressure_unit': pressureUnit
          };
          sendTelemetry(data, pressureSchema)
        }, 5000);

        client.on('error', function (err) {
          printErrorFor('client')(err);
          if (sendTemperatureInterval) clearInterval(sendTemperatureInterval);
          if (sendHumidityInterval) clearInterval(sendHumidityInterval);
          if (sendPressureInterval) clearInterval(sendPressureInterval);
          client.close(printErrorFor('client.close'));
        });
      }
    });
    ```

1. Spara ändringarna i den **remote_monitoring.js** fil.

1. Om du vill starta exempelprogrammet, kör du följande kommando vid en kommandotolk på hallon Pi:

    ```sh
    node remote_monitoring.js
    ```

[!INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]
