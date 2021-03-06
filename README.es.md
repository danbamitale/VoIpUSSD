# Librería MANEJO USSD 

[![Platform](https://img.shields.io/badge/platform-android-brightgreen.svg)](https://developer.android.com/index.html)
[![API](https://img.shields.io/badge/API-17%2B-brightgreen.svg?style=flat)](https://android-arsenal.com/api?level=17) 
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](https://github.com/romellfudi/VoIpUSSDSample/blob/master/LICENSE)
[![Bintray](https://img.shields.io/bintray/v/romllz489/maven/ussd-library.svg)](https://bintray.com/romllz489/maven/ussd-library)
[![Android Arsenal]( https://img.shields.io/badge/Android%20Arsenal-Void%20USSD%20Library-green.svg?style=flat )]( https://android-arsenal.com/details/1/7151 )
[![Jitpack](https://jitpack.io/v/romellfudi/VoIpUSSDSample.svg)](https://jitpack.io/#romellfudi/VoIpUSSDSample)
[![CircleCi](https://img.shields.io/circleci/project/github/romellfudi/VoIpUSSDSample.svg)](https://circleci.com/gh/romellfudi/VoIpUSSDSample/tree/master)

### by Romell Dominguez
[![](snapshot/icono.png)](https://www.romellfudi.com/)

## Objetivo [High Quality](https://raw.githubusercontent.com/romellfudi/VoIpUSSD/Rev04/snapshot/device_recored.gif):

![](snapshot/device_recored.gif#gif)

Para manejar la comunicación ussd, hay que tener presente que la interfaz depende del SO y del fabricante.

## USSD LIBRARY

`latestVersion` is 1.1.c

Agregar en tu archivo `build.gradle` del proyecto Android:

```groovy
repositories {
    jcenter()
}
dependencies {
    compile 'com.romellfudi.ussdlibrary:ussd-library:{latestVersion}'
}
```

* Escribir el archivo xml [acá](https://github.com/romellfudi/VoIpUSSD/blob/master/ussd-library/src/main/res/xml/ussd_service.xml) to res/xml folder (if necessary), this config file allow link between App and SO:

```xml
<?xml version="1.0" encoding="utf-8"?>
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    .../>
```

### Application

Agregar las dependencias: CALL_PHONE, READ_PHONE_STATE and SYSTEM_ALERT_WINDOW:

```xml
    <uses-permission android:name="android.permission.CALL_PHONE" />
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
```

Agregar el servicio:

```xml
    <service
        android:name="com.romellfudi.ussdlibrary.USSDService"
        android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
        <intent-filter>
            <action android:name="android.accessibilityservice.AccessibilityService" />
        </intent-filter>
        <meta-data
            android:name="android.accessibilityservice"
            android:resource="@xml/ussd_service" />
    </service>
```

# Uso del API:

Primero necesitamos mapear los mensajes de respuesta USSD, para idetificar los de logeo y de error

| KEY MESSAGE | String Messages |
| ------ | ------ |
| KEY_LOGIN | "espere","waiting","loading","esperando",... |
| KEY_ERROR | "problema","problem","error","null",... |

```java
map = new HashMap<>();
map.put("KEY_LOGIN",..."waiting"...);
map.put("KEY_ERROR",..."problem"...);
```

Instancia un objeto ussController con su activity

```java
ussdController = USSDController.getInstance(activity);
ussdController.callUSSDInvoke(phoneNumber, map, new USSDController.CallbackInvoke() {
    @Override
    public void responseInvoke(String message) {
        // message has the response string data
        dataToSend // send "data" into USSD's input text
        ussdController.send(dataToSend,new USSDController.CallbackMessage(){
            @Override
            public void responseMessage(String message) {
                // message has the response string data from USSD
            }
        });
    }

    @Override
    public void over(String message) {
        // message has the response string data from USSD
        // response no have input text, NOT SEND ANY DATA
    }
});

```

Si requiere un flujo de trabajo, tienes que usar la siguiente estructura:

```java
ussdController.callUSSDInvoke(phoneNumber, map, new USSDController.CallbackInvoke() {
    @Override
    public void responseInvoke(String message) {
        // first option list - select option 1
        ussdController.send("1",new USSDController.CallbackMessage(){
            @Override
            public void responseMessage(String message) {
                // second option list - select option 1
                ussdController.send("1",new USSDController.CallbackMessage(){
                    @Override
                    public void responseMessage(String message) {
                        ...
                    }
                });
            }
        });
    }

    @Override
    public void over(String message) {
        // message has the response string data from USSD
        // response no have input text, NOT SEND ANY DATA
    }
    ...
});
```

## OverlayShowingService Widget (no indispensable)

Un severo problema al manejar este tipo de widget, este no puede ocultarse, redimencionarse, no puede ser puesto en el fondo con un rogressDialog
Pero recientemente a partir del Android O, Google permite la construcción build a nw kind permission dde widget sobrepuestos, mi solución implementada fue este widget llamdo `OverlayShowingService`:
For use need add permissions at AndroidManifest:

```xml
<uses-permission android:name="android.permission.ACTION_MANAGE_OVERLAY_PERMISSION" />
```

Agregar Broadcast Service:

```xml
<service android:name="com.romellfudi.ussdlibrary.OverlayShowingService"
         android:exported="false" />
```

Invocar como cualquier servicio, necesita un titulo para ser mostrado mientras se ejecuta la llama `callUSSDInvoke` mediante una variable extra `EXTRA`:

```java
Intent svc = new Intent(activity, OverlayShowingService.class);
svc.putExtra(OverlayShowingService.EXTRA,"PROCESANDO");
getActivity().startService(svc);
// stop
getActivity().stopService(svc);
```

### EXTRA: Uso de la línea voip

En esta sección dejo las líneas claves para realizar la conexión VOIP-USSD

```java
ussdPhoneNumber = ussdPhoneNumber.replace("#", uri);
Uri uriPhone = Uri.parse("tel:" + ussdPhoneNumber);
context.startActivity(new Intent(Intent.ACTION_CALL, uriPhone));
```

Una vez inicializado la llamada el servidor telcom comenzará a enviar las *famosas pantallas **ussd***

![image](snapshot/telcom.png#center)

### License
```
Copyright 2018 Romell D.Z.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

<style>
img[src*='#center'] { 
    width:390px;
    display: block;
    margin: auto;
}
img[src*='#gif'] { 
    width:200px;
    display: block;
    margin: auto;
}
</style>