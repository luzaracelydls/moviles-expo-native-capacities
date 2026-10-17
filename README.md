# Capacidades nativas de Expo

## Objetivo

Construir paso a paso una aplicación móvil llamada **Explora**, que permita:

- Tomar una fotografía o seleccionar una imagen de la galería.
- Obtener la ubicación actual del dispositivo.
- Registrar una fotografía junto con sus coordenadas.
- Leer los valores del acelerómetro.
- Programar una notificación local.

Durante el laboratorio trabajaremos con APIs de Expo para comprender cómo una aplicación móvil accede a las capacidades del dispositivo, solicita permisos y responde a los resultados.

**Notas**
No todas las funciones nativas se comportan igual en Expo Web y Expo Go. Necesitas Expo GO.

---

# 1. Crear el proyecto

## 1.1. Requisitos

Necesitamos:

- Node.js instalado.
- Un proyecto nuevo o existente de Expo.
- Expo Router configurado.
- Expo Go instalado en el teléfono o un development build compatible.
- VS Code o el editor de código de tu preferencia
- Conexión entre la computadora y el teléfono para las pruebas en vivo.

## 1.2. Revisar la estructura

Crearemos los siguientes archivos:

```text
explora/
├── app/
│   ├── _layout.js
│   └── index.js
├── components/
│   ├── CameraPicker.js
│   ├── LocationDemo.js
│   ├── MotionDemo.js
│   └── ReminderDemo.js
├── assets/
├── app.json
└── package.json
```

**Nota para quienes utilizan algun template** Si tu proyecto tiene una estructura diferente, conserva las rutas y la configuración existentes y adapta los archivos de esta guía.

## 1.3. Instalar las dependencias

Desde la raíz del proyecto, ejecuta:

```bash
npx expo install expo-image-picker
npx expo install expo-location
npx expo install expo-sensors
npx expo install expo-notifications
```

Utilizamos `npx expo install` para instalar versiones compatibles con el SDK de Expo del proyecto.

Después, inicia la aplicación:

```bash
npx expo start
```

Si tienes problemas de conexión con Expo Go, puedes probar:

```bash
npx expo start --tunnel
```

No continúes con los ejemplos nativos hasta comprobar que puedes abrir el proyecto en el entorno elegido.

---

# 2. Primera capacidad: cámara y galería


Permitir que el usuario tome una fotografía o seleccione una imagen y mostrarla en la interfaz.

### Topicos

- API nativa.
- Permisos.
- Funciones asíncronas.
- `async` y `await`.
- Manejo de cancelaciones.
- URI de una imagen.

## 2.1. Crear el componente

Crea `components/CameraPicker.js`:

```jsx
import {
  View,
  Text,
  Pressable,
  Image,
  StyleSheet,
  Alert,
} from 'react-native';

import * as ImagePicker from 'expo-image-picker';

export default function CameraPicker({
  photoUri,
  onPhotoSelected,
}) {
  async function takePhoto() {
    try {
      const permission =
        await ImagePicker.requestCameraPermissionsAsync();

      if (!permission.granted) {
        Alert.alert(
          'Permiso necesario',
          'Necesitamos acceso a la cámara para tomar una fotografía.'
        );
        return;
      }

      const result = await ImagePicker.launchCameraAsync({
        mediaTypes: ['images'],
        quality: 0.8,
      });

      if (!result.canceled) {
        onPhotoSelected(result.assets[0].uri);
      }
    } catch (error) {
      console.error(error);

      Alert.alert(
        'Error',
        'No fue posible abrir la cámara.'
      );
    }
  }

  async function choosePhoto() {
    try {
      const result =
        await ImagePicker.launchImageLibraryAsync({
          mediaTypes: ['images'],
          quality: 0.8,
        });

      if (!result.canceled) {
        onPhotoSelected(result.assets[0].uri);
      }
    } catch (error) {
      console.error(error);

      Alert.alert(
        'Error',
        'No fue posible abrir la galería.'
      );
    }
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>
        1. Fotografía
      </Text>

      <Pressable
        style={styles.button}
        onPress={takePhoto}
      >
        <Text style={styles.buttonText}>
          Tomar fotografía
        </Text>
      </Pressable>

      <Pressable
        style={styles.secondaryButton}
        onPress={choosePhoto}
      >
        <Text style={styles.buttonText}>
          Elegir de la galería
        </Text>
      </Pressable>

      {photoUri ? (
        <Image
          source={{ uri: photoUri }}
          style={styles.photo}
          resizeMode="cover"
        />
      ) : (
        <Text>
          Todavía no hay una fotografía seleccionada.
        </Text>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    gap: 12,
    backgroundColor: '#FFFFFF',
    borderRadius: 16,
  },
  title: {
    fontSize: 22,
    fontWeight: 'bold',
  },
  button: {
    backgroundColor: '#176B57',
    padding: 14,
    borderRadius: 10,
    alignItems: 'center',
  },
  secondaryButton: {
    backgroundColor: '#344054',
    padding: 14,
    borderRadius: 10,
    alignItems: 'center',
  },
  buttonText: {
    color: '#FFFFFF',
    fontWeight: 'bold',
  },
  photo: {
    width: '100%',
    height: 220,
    borderRadius: 12,
  },
});
```

## ¿Qué sucede aquí?

1. `requestCameraPermissionsAsync()` solicita acceso a la cámara.
2. `launchCameraAsync()` abre la cámara.
3. `launchImageLibraryAsync()` permite seleccionar una imagen.
4. `result.canceled` indica si el usuario canceló la selección.
5. `result.assets[0].uri` contiene la referencia a la imagen seleccionada.
6. `onPhotoSelected()` devuelve la URI al componente padre.

La galería puede utilizar un selector del sistema que no requiera el mismo flujo de permisos que la cámara.

## Para comentar en clase

- ¿Por qué utilizamos `await`?
- ¿Qué ocurre si el usuario cancela la cámara?
- ¿Por qué guardamos la URI en el estado?
- ¿La fotografía se ha subido a internet? No: solo hemos obtenido una referencia a ella.

---

# Segunda parte: Ubicación y GPS

## Objetivo

Obtener las coordenadas actuales del dispositivo y manejar los permisos correspondientes.

### Tópicos

- Latitud y longitud.
- Permisos en primer plano.
- Estado de carga.
- Errores de ubicación.
- Privacidad y consentimiento.

## Componente Location

Crea `components/LocationDemo.js`:

```jsx
import { useState } from 'react';

import {
  View,
  Text,
  Pressable,
  StyleSheet,
} from 'react-native';

import * as Location from 'expo-location';

export default function LocationDemo({
  onLocationFound,
}) {
  const [location, setLocation] = useState(null);
  const [status, setStatus] = useState(
    'Presiona el botón para comenzar.'
  );
  
  const [loading, setLoading] = useState(false);

  async function getLocation() {
    setLoading(true);
    setStatus('Solicitando permiso...');

    try {
      const permission =
        await Location.requestForegroundPermissionsAsync();

      if (!permission.granted) {
        setStatus(
          'No se concedió el permiso de ubicación.'
        );
        return;
      }

      setStatus('Buscando ubicación...');

      const result =
        await Location.getCurrentPositionAsync({
          accuracy: Location.Accuracy.Balanced,
        });

      const point = {
        latitude: result.coords.latitude,
        longitude: result.coords.longitude,
      };

      setLocation(point);
      setStatus('Ubicación obtenida correctamente.');

      onLocationFound?.(point);
    } catch (error) {
      console.error(error);

      setStatus(
        'No pudimos obtener la ubicación. Intenta de nuevo.'
      );
    } finally {
      setLoading(false);
    }
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>
        2. Ubicación
      </Text>

      <Pressable
        style={styles.button}
        onPress={getLocation}
        disabled={loading}
      >
        <Text style={styles.buttonText}>
          {loading
            ? 'Buscando...'
            : 'Obtener ubicación'}
        </Text>
      </Pressable>

      <Text>{status}</Text>

      {location && (
        <View style={styles.result}>
          <Text>
            Latitud: {location.latitude.toFixed(5)}
          </Text>

          <Text>
            Longitud: {location.longitude.toFixed(5)}
          </Text>
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    gap: 12,
    backgroundColor: '#FFFFFF',
    borderRadius: 16,
  },
  title: {
    fontSize: 22,
    fontWeight: 'bold',
  },
  button: {
    padding: 14,
    backgroundColor: '#176B57',
    borderRadius: 10,
    alignItems: 'center',
  },
  buttonText: {
    color: '#FFFFFF',
    fontWeight: 'bold',
  },
  result: {
    padding: 12,
    backgroundColor: '#E7F4EF',
    borderRadius: 8,
    gap: 6,
  },
});
```

## ¿qué significan estas coordenadas?

La latitud representa la posición norte-sur y la longitud representa la posición este-oeste.

El método `getCurrentPositionAsync()` puede tardar varios segundos. La precisión depende de la configuración, los permisos y las condiciones del dispositivo.

No es necesario solicitar acceso a la ubicación en segundo plano para esta actividad.


---

# Integrar fotografía y ubicación

Crear un registro que combine una fotografía con las coordenadas obtenidas.

## Estructura de un registro

```jsx
const entry = {
  id: String(Date.now()),
  photoUri: photoUri,
  latitude: location.latitude,
  longitude: location.longitude,
  createdAt: new Date().toISOString(),
};
```

El objeto contiene:

- `id`: identificador temporal.
- `photoUri`: referencia a la fotografía.
- `latitude` y `longitude`: coordenadas.
- `createdAt`: fecha y hora del registro.

## Componente principal (index.js)

Reemplaza el contenido de `app/index.js` por:

```jsx
import { useState } from 'react';

import {
  ScrollView,
  View,
  Text,
  Pressable,
  Alert,
  StyleSheet,
} from 'react-native';

import CameraPicker from '../components/CameraPicker';
import LocationDemo from '../components/LocationDemo';

export default function HomeScreen() {
  const [photoUri, setPhotoUri] = useState(null);
  const [location, setLocation] = useState(null);
  const [entry, setEntry] = useState(null);

  function saveEntry() {
    if (!photoUri) {
      Alert.alert(
        'Falta la fotografía',
        'Primero toma o selecciona una imagen.'
      );
      return;
    }

    if (!location) {
      Alert.alert(
        'Falta la ubicación',
        'Primero obtiene las coordenadas.'
      );
      return;
    }

    const newEntry = {
      id: String(Date.now()),
      photoUri,
      latitude: location.latitude,
      longitude: location.longitude,
      createdAt: new Date().toISOString(),
    };

    setEntry(newEntry);

    console.log('Registro creado:', newEntry);

    Alert.alert(
      'Registro creado',
      'La fotografía y la ubicación están listas.'
    );
  }

  return (
    <ScrollView contentContainerStyle={styles.container}>
      <Text style={styles.heading}>
        Explora
      </Text>

      <Text style={styles.subtitle}>
        Documenta los lugares que descubres.
      </Text>

      <CameraPicker
        photoUri={photoUri}
        onPhotoSelected={setPhotoUri}
      />

      <LocationDemo
        onLocationFound={setLocation}
      />

      <Pressable
        style={styles.saveButton}
        onPress={saveEntry}
      >
        <Text style={styles.saveButtonText}>
          Crear registro
        </Text>
      </Pressable>

      {entry && (
        <View style={styles.result}>
          <Text style={styles.resultTitle}>
            Registro actual
          </Text>

          <Text>ID: {entry.id}</Text>

          <Text>
            Latitud: {entry.latitude.toFixed(5)}
          </Text>

          <Text>
            Longitud: {entry.longitude.toFixed(5)}
          </Text>

          <Text>
            Fecha: {new Date(
              entry.createdAt
            ).toLocaleString()}
          </Text>
        </View>
      )}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 20,
    paddingTop: 60,
    gap: 20,
    backgroundColor: '#F2F4F7',
  },
  heading: {
    fontSize: 32,
    fontWeight: 'bold',
  },
  subtitle: {
    color: '#475467',
  },
  saveButton: {
    backgroundColor: '#176B57',
    padding: 16,
    borderRadius: 12,
    alignItems: 'center',
  },
  saveButtonText: {
    color: '#FFFFFF',
    fontWeight: 'bold',
  },
  result: {
    backgroundColor: '#FFFFFF',
    padding: 16,
    borderRadius: 12,
    gap: 8,
  },
  resultTitle: {
    fontSize: 20,
    fontWeight: 'bold',
  },
});
```

## Pruebas

Prueba estos escenarios:

1. Crear un registro con fotografía y ubicación.
2. Intentar crear un registro sin fotografía.
3. Intentar crear un registro sin ubicación.
4. Cambiar la fotografía y crear otro registro.

**Resultado esperado:** el registro se crea únicamente cuando existen ambos datos.

La información permanece en el estado de React. Si reinicias la aplicación, se perderá. 

---

# 5. Sensores y acelerómetro

Leer los datos del acelerómetro y observar cómo cambian al mover o inclinar el dispositivo.

### Conceptos clave

- Sensores del dispositivo.
- Datos que se actualizan continuamente.
- `useEffect`.
- Suscripciones.
- Limpieza de recursos.

## Crea el componente MotionDemo

Crea `components/MotionDemo.js`:

```jsx
import { useEffect, useState } from 'react';

import {
  View,
  Text,
  StyleSheet,
} from 'react-native';

import { Accelerometer } from 'expo-sensors';

export default function MotionDemo() {
  const [available, setAvailable] = useState(null);

  const [data, setData] = useState({
    x: 0,
    y: 0,
    z: 0,
  });

  useEffect(() => {
    let subscription;

    async function startSensor() {
      try {
        const supported =
          await Accelerometer.isAvailableAsync();

        setAvailable(supported);

        if (!supported) return;

        Accelerometer.setUpdateInterval(250);

        subscription = Accelerometer.addListener(
          (measurement) => {
            setData(measurement);
          }
        );
      } catch (error) {
        console.error(error);
        setAvailable(false);
      }
    }

    startSensor();

    return () => {
      subscription?.remove();
    };
  }, []);

  if (available === false) {
    return (
      <Text>
        El acelerómetro no está disponible.
      </Text>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>
        3. Movimiento
      </Text>

      {available === null ? (
        <Text>Comprobando sensor...</Text>
      ) : (
        <>
          <Text>X: {data.x.toFixed(2)}</Text>
          <Text>Y: {data.y.toFixed(2)}</Text>
          <Text>Z: {data.z.toFixed(2)}</Text>

          <Text>
            Mueve o inclina el teléfono para observar
            los cambios.
          </Text>
        </>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    gap: 12,
    backgroundColor: '#E8F1FF',
    borderRadius: 16,
  },
  title: {
    fontSize: 22,
    fontWeight: 'bold',
  },
});
```

## Tópicos

- `isAvailableAsync()` comprueba si el sensor está disponible.
- `addListener()` recibe actualizaciones del acelerómetro.
- `setUpdateInterval(250)` solicita actualizaciones aproximadamente cada 250 ms.
- `subscription.remove()` elimina la suscripción cuando el componente se desmonta.
- `useEffect()` permite iniciar el sensor y limpiar los recursos utilizados.

Los valores pueden cambiar incluso cuando el teléfono está quieto, porque el acelerómetro también registra la influencia de la gravedad y la orientación.

## Incorporar el componente

En `app/index.js`, importa el componente:

```jsx
import MotionDemo from '../components/MotionDemo';
```

Después, colócalo dentro del `ScrollView`:

```jsx
<MotionDemo />
```

## Pruebas

1. Coloca el teléfono sobre una mesa.
2. Observa los valores.
3. Inclina el dispositivo.
4. Muévelo suavemente.
5. Sal de la pantalla y verifica que la suscripción se limpie.

Pregunta: ¿qué problema habría si cada vez que entramos a la pantalla creamos una suscripción y nunca la eliminamos?

---

# Notificaciones locales


Programar un recordatorio local que avise al usuario que debe completar las notas de una visita.

### Tópicos

- Permisos de notificación.
- Notificaciones locales y push.
- Programación temporal.
- Diferencias entre plataformas.
- Presentación de notificaciones.

## Crea el componente de Reminder

Crea `components/ReminderDemo.js`:

```jsx
import { useState } from 'react';

import {
  View,
  Text,
  Pressable,
  Alert,
  Platform,
  StyleSheet,
} from 'react-native';

import * as Notifications from 'expo-notifications';

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowBanner: true,
    shouldShowList: true,
    shouldPlaySound: false,
    shouldSetBadge: false,
  }),
});

export default function ReminderDemo() {
  const [status, setStatus] = useState(
    'Sin recordatorio'
  );

  async function scheduleReminder() {
    try {
      if (Platform.OS === 'android') {
        await Notifications.setNotificationChannelAsync(
          'explora-reminders',
          {
            name: 'Recordatorios de Explora',
            importance:
              Notifications.AndroidImportance.DEFAULT,
          }
        );
      }

      const permission =
        await Notifications.requestPermissionsAsync();

      if (!permission.granted) {
        setStatus(
          'Permiso de notificaciones denegado.'
        );
        return;
      }

      await Notifications.scheduleNotificationAsync({
        content: {
          title: 'Completa tu visita',
          body: 'Agrega notas a tu registro de Explora.',
          sound: false,
        },
        trigger: {
          type:
            Notifications.SchedulableTriggerInputTypes
              .TIME_INTERVAL,
          seconds: 60,
          ...(Platform.OS === 'android'
            ? { channelId: 'explora-reminders' }
            : {}),
        },
      });

      setStatus(
        'Recordatorio programado para dentro de un minuto.'
      );

      Alert.alert(
        'Listo',
        'El recordatorio quedó programado.'
      );
    } catch (error) {
      console.error(error);

      setStatus(
        'No se pudo programar el recordatorio.'
      );
    }
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>
        4. Recordatorios
      </Text>

      <Pressable
        style={styles.button}
        onPress={scheduleReminder}
      >
        <Text style={styles.buttonText}>
          Recordarme en un minuto
        </Text>
      </Pressable>

      <Text>{status}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    gap: 12,
    backgroundColor: '#FFFFFF',
    borderRadius: 16,
  },
  title: {
    fontSize: 22,
    fontWeight: 'bold',
  },
  button: {
    padding: 14,
    backgroundColor: '#4935A8',
    borderRadius: 10,
    alignItems: 'center',
  },
  buttonText: {
    color: '#FFFFFF',
    fontWeight: 'bold',
  },
});
```

## Incorporar el componente

En `app/index.js`, agrega:

```jsx
import ReminderDemo from '../components/ReminderDemo';
```

Después, dentro del `ScrollView`:

```jsx
<ReminderDemo />
```

## Pruebas

1. Pulsa el botón.
2. Concede el permiso de notificaciones.
3. Deja pasar un minuto.
4. Observa el resultado con la app en primer plano y en segundo plano.
5. Repite la prueba con los permisos denegados.

**Notas:**

- Una notificación local se programa en el dispositivo.
- Una notificación push se envía mediante un servicio remoto.

**La aparición de una notificación depende de la plataforma, la configuración, los permisos y el entorno de ejecución. Comprueba la compatibilidad del SDK antes de realizar la demostración.**

---

# Integra los sensores y notificaciones

Una vez creados los componentes, agrega sus importaciones a `app/index.js`:

```jsx
import MotionDemo from '../components/MotionDemo';
import ReminderDemo from '../components/ReminderDemo';
```

Colócalos dentro del `ScrollView`, después de la cámara y la ubicación:

```jsx
<MotionDemo />

<ReminderDemo />
```

La pantalla principal tendrá cuatro secciones:

1. Fotografía.
2. Ubicación.
3. Movimiento.
4. Recordatorios.

El objetivo de esta etapa es comprobar que los componentes funcionan juntos sin reemplazar la estructura de navegación existente.

---

#  Pruebas


| Prueba | Resultado esperado |
|---|---|
| Tomar una fotografía | Se muestra la imagen seleccionada. |
| Cancelar la cámara | La app permanece abierta. |
| Denegar el permiso de cámara | Se informa que no se concedió el permiso. |
| Obtener ubicación | Se muestran latitud y longitud. |
| Denegar ubicación | Se muestra un mensaje y no se crea el registro. |
| Crear un registro completo | Se combinan fotografía y coordenadas. |
| Mover el teléfono | Cambian los valores del acelerómetro. |
| Salir de la pantalla del sensor | Se elimina la suscripción. |
| Programar un recordatorio | Se registra la notificación local. |
| Denegar notificaciones | Se informa que no se concedió el permiso. |

## Challenge:

Por pares, propongan una respuesta para cada escenario:

- El usuario no quiere compartir su ubicación.
- El dispositivo no tiene disponible el sensor.
- La cámara se cancela.
- La app no puede mostrar una notificación.
- El usuario quiere guardar una fotografía sin ubicación.

La respuesta no tiene que ser siempre bloquear la funcionalidad. La experiencia debe adaptarse al propósito de la aplicación.

---

# Notas y buenas prácticas:

- Solicita permisos cuando el usuario intenta utilizar la funcionalidad.
- Explica por qué necesitas cada permiso.
- Maneja las cancelaciones y los errores.
- No mantengas sensores activos cuando no sean necesarios.
- Evita capturar o almacenar información personal sin una razón clara.
- No asumas que una API se comporta igual en web, iOS y Android.
- Mantén la navegación y los componentes existentes del proyecto.
- Prueba cada cambio antes de integrar el siguiente.

---

# Referencias

- [Expo ImagePicker](https://docs.expo.dev/versions/latest/sdk/imagepicker/)
- [Expo Location](https://docs.expo.dev/versions/latest/sdk/location/)
- [Expo Sensors](https://docs.expo.dev/versions/latest/sdk/sensors/)
- [Expo Notifications](https://docs.expo.dev/versions/latest/sdk/notifications/)
- [Expo Router](https://docs.expo.dev/router/introduction/)
