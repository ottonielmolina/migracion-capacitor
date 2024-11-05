# Migración Cordova a Capacitor

Según documentación oficial hay varios pasos que se deben realizar para migrar una aplicación que utiliza Cordova a Capacitor, así mismo, recomiendan realizarlo en una rama de trabajo separada de la rama estable.

##  1. Agregar Capacitor
Se debe abrir el proyecto en una terminal e instalar las dependencias de capacitor para una aplicación ionic:

```bash
$ npm i @capacitor/app @capacitor/haptics @capacitor/keyboard @capacitor/status-bar
```

Inicializa la aplicación con Capacitor. Se solicitará información que puede encontrarla en el archivo **config.xml**, tal como el app name y el bundle ID.

```bash
$ npx cap init
```

##  2. Construir la aplicación web
Deberás construir el proyecto web antes de agregar cualquier plataforma nativa.

```bash
$ npm run build
```
Esto garantiza que la carpeta **www** se haya configurado automáticamente para utilizar como **webDir** en el archivo de configuración de Capacitor.

##  3. Agregar plataformas
Después de instalar las dependencias de capacitor agrega las plataformas móbiles a la aplicación:

```bash
$ ionic capacitor add android
$ ionic capacitor add ios
```

Ambas plataformas nativas se agregan a nivel raiz del proyecto. Estos son proyectos nativos separados que pueden ser considerados parte de la aplicación y pueden ser incluidos como parte del versionamiento de código. 

Adicionalmente cualquier **plugin de Cordova** que se encuentre bajo el atributo **_dependencies_** en el archivo **_package.json_** será instalado de forma automática por Capacitor dentro del nuevo proyecto nativo:

```json
"dependencies": {
    "@ionic-native/camera": "^5.3.0",
    "@ionic-native/core": "^5.3.0",
    "@ionic-native/file": "^5.3.0",
    "cordova-android": "12.0.1",
    "cordova-ios": "6.3.0",
    "cordova-plugin-camera": "4.0.3",
    "cordova-plugin-file": "6.0.1",
}
```

>[!NOTE]
> Algunos plugins de cordova son incompatibles por lo que se exceptuarán cuando se agregue la plataforma, puede consultar la lista de [plugins incompatibles].

##  4. Splash Screens e Iconos
Las imágenes de Splash Screen e íconos se encuentran en la carpeta resources en el nivel raíz del proyecto, con lo que podremos ayudarnos de la herramienta **@capacitor/assets** para regenerarlos en cada proyecto nativo.

- Primero instalar **@capacitor/assets**
```bash
$ npm install @capacitor/assets --save-dev
```

- Los recursos de iconos y splash screen deberán estar como la siguiente estructura:
```
assets/
├── icon-only.png
├── icon-foreground.png
├── icon-background.png
├── splash.png
└── splash-dark.png
```
  Iconos deberán tener un tamaño de al menos **1024px** x **1024px**.  
  Splash Screen deberá tener un tamaño de al menos **2732px** x **2732px**.  
  El formato de archivos puede ser **jpg** o **png**.

- Generar los recursos para cada plataforma:

```bash
$ npx capacitor-assets generate --android
$ npx capacitor-assets generate --ios
```

##  5. Migración de Plugins
Al comenzar a auditar los plugins de cordova presentes en la aplicación será posible encontrar algunos que ya no serán utilizados, por lo que deben ser removidos.  
Lo siguiente será revisar en la web de [plugins oficiales] así como en la [comunidad de plugins] para encontrar el plugin de **Capacitor** equivalente a cada plugin de **Cordova**.  

Este [archivo excel] contiene un detalle de los plugins de la aplicación.

Después de reemplazar el plugin de **Cordova** por su equivalente en **Capacitor** se deberá desinstalar el plugin de **Cordova** y ejecutar el comando `sync` para remover el plugin del proyecto nativo:

```bash
$ npm uninstall cordova-plugin-name
$ npx cap sync
```

En algunos casos no habrá un plugin equivalente en capacitor, por lo que será necesario utilizar el plugin de `@awesome-cordova-plugins`

##  6. Establecer permisos
De forma predeterminada, todos los permisos iniciales solicitados para Capacitor se configuran en los proyectos nativos predeterminados tanto para iOS como para Android. Sin embargo, es posible que deba aplicar permisos adicionales manualmente mediante la asignación entre `plugin.xml` y la configuración requerida en iOS y Android.

### Permisos Android
Las aplicaciones android maneja los permisos, características del dispositivo y otras configuraciones en el archivo `AndroidManifest.xml` el cual se localiza en la ruta `android/app/src/main/AndroidManifest.xml`.  

En Android los permisos de la aplicación están definidos en el archivo `AndroidManifest.xml`  en la etiqueta `<manifest>` generalmente al final del archivo.  
Un ejemplo de agregar el permiso de red sería el siguiente:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.getcapacitor.myapp">
    <activity>
      <!-- other stuff -->
    </activity>

    <!-- More stuff -->

    <!-- Your permissions -->

    <!-- Network API -->
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
</manifest>
```

>[!NOTE]
> Los permisos en Android deberán ser configurados con la plataforma nativa abierta en Android Studio.

### Permisos iOS
Los permisos en iOS no necesitan ser especificados explícitamente como en el caso de Android, sin embargo, iOS si requiere que se defina la llave `Usage Descriptions` en el archivo `Info.plist`. Estas configuraciones contiene descripciones entendibles por humanos y serán presentadas al usuario cuando el permiso sea requerido por el dispositivo.

En la web de [Cocoa Keys] puede encontrar las claves que contienen UsageDescription para ver las diversas configuraciones de descripción de uso que pueden ser necesarias para la aplicación.  
Un ejemplo para configuración del permiso de cámara puede ser:
```xml
<plist version="1.0">
    <dict>
        <!-- Other keys -->
        <key>NSCameraUsageDescription</key>
        <string>Amigo Banrural, necesitamos acceso a la camara</string>
    </dict>
</plist>
```

>[!NOTE]
> Los permisos en iOS deberán ser configurados con la plataforma nativa abierta en Xcode en la configuración del proyecto en la sección `Info`.

##  7. Preferencias de Cordova Plugin
Cuando se ejecuta `npx cap init`, **Capacitor** lee todas las preferencias del archivo `config.xml` y las transfiere al archivo de configuración de **Capacitor**. Puede agregar manualmente más preferencias al objeto `cordova.preferences`:

```json
{
  "cordova": {
    "preferences": {
      "DisableDeploy": "true",
      "CameraUsesGeolocation": "true"
    }
  }
}
```

##  8. Campos adicionales del archivo `config.xml`
>[!TIP]
> Es imposible cubrir cada unos de los elementos disponibles en `config.xml`. No obstante, muchas de las preguntas relacionadas a "*Como configuro 'X' en Capacitor?*" debe ser pensado como "*Como configuro 'X' en [plataforma] (iOS/Android)?*" cuando se realizan búsquedas en línea.

##  9. Establecer scheme
Cuando usa Ionic con **Cordova**, la aplicación utiliza `cordova-plugin-ionic-webview` de forma predeterminada, que en iOS usa el esquema `ionic://` para entregar el contenido. Las aplicaciones de **Capacitor** usan `capacitor://` como esquema predeterminado en iOS. Esto significa que el uso de una API web vinculada al origen, como LocalStorage, provocará una pérdida de datos ya que el origen es diferente. Esto se puede solucionar cambiando el esquema que se utiliza para publicar el contenido en el archivo `capacitor.config.ts`:

```json
{
  "server": {
    "iosScheme": "ionic",
    "androidScheme": "ionic"
  }
}
```

## 10. Remover Cordova
Una vez se ha probado que todos los cambios en la migración han sido aplicados y funcionan correctamente, puede remover **Cordova** del proyecto. Elimina el archivo `config.xml` y los directorios `platforms` y `plugins`.  
Se debe tomar en cuenta que técnicamente no es necesario eliminar **Cordova**, ya que **Capacitor** funciona junto con él. De hecho, si se planea continuar utilizando los complementos de Cordova o se cree que se podrá hacerlo en el futuro, se puede dejar los assets de **Cordova** donde están.

..

[plugins oficiales]: https://capacitorjs.com/docs/apis
[comunidad de plugins]: https://capacitorjs.com/docs/plugins/community
[archivo excel]: https://docs.google.com/spreadsheets/d/1-EfHPetkALJgrliTac72OWMXdaqMWi4Bi8_oCVVC0hM/edit?usp=sharing
[Cocoa Keys]: https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html
[plugins incompatibles]: https://capacitorjs.com/docs/plugins/cordova#known-incompatible-plugins