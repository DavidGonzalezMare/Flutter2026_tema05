![Union europea](./images/union_europea.jpeg)  ![Generalitat](./images/generalitat.jpeg) ![Mare Nostrum](./images/mare_nostrum.png)


<br>

<a id="_apartado1"></a>

<br>

# Práctica 5. Future Builder

En esta tarea haremos evolucionar nuestra aplicación y obtendremos la información de las comarcas desde Internet, utilizando la API que vimos en la **práctica 2**.

Como vimos en la unidad 2 y trabajamos en la práctica 2, los accesos a servicios web nos devuelven la información a través de Futures, de manera que ahora utilizaremos el widget `FutureBuilder` para construir los widgets que dependan de estos servicios.

Vamos a ver los aspectos más relevantes de la práctica.

<br>

## Estructura general del proyecto

La estructura general será similar a la siguiente:

```
lib
├── main.dart
├── models
│ ├── comarca.dart
│ └── provincia.dart
├── repository
│ ├── repository_comarcas.dart
│ └── repository_weather.dart
├── screens
│ ├── comarcas_screen.dart
│ ├── inicio_usuario.dart
│ ├── infocomarca_detall.dart
│ ├── infocomarca_general.dart
│ ├── infocomarca_screen.dart
│ ├── provincias_screen.dart
│ └── widgets
│ 	├── my_circular_progress_indicator.dart
│ 	└── my_weather_info.dart
├── services
│ ├── comarcas_service.dart
│ └── weather_service.dart
└── themes
└── tema_comarcas.dart
```

Tenemos:

- La carpeta `models` , con los modelos de `Comarca` y `Provincia`. Estas clases no sufrirán ningún cambio.
  
- La carpeta `screens`, con las diferentes pantallas y widgets, que habrá que adaptar para trabajar con `Futures`. Como veis, se ha incorporado una clase `MyCircularProgressIndicator`, que no es más que un indicador de progreso circular que hemos envuelto en un `Center` y un `SizedBox` para personalizar la estética del mismo.
  
- La carpeta `services`, que implementa la parte de acceso a los servicios, tanto de información de las comarcas como de información meteorológica.
  
- La carpeta `repository`, que se encarga de obtener la información a través de los servicios. Observe que ahora hemos eliminado las clases `RepositoryEjemplo` y `RepositoryData`, ya que la información la obtendremos de Internet.

<br>

## Adaptación de Servicios y Repositorios

En la `práctica 2` creamos la clase `ComarcasService` , que nos ofrecía acceso a la información de la API mediante llamadas HTTP.

Esta clase nos ofrecía los métodos:

- `obtenerProvincias`, que nos devuelve, en un `Future` la lista de provincias,
  
- `obtenerComarcas`, que devolvía, en un `Future`, una lista de las comarcas por Provincia
  
- `infoComarca`, que devolvía una comarca con la información sobre la misma que ofrecía la API.
  
Vamos a incorporar ahora este servicio a nuestro proyecto. Para ello sólo será necesario, en primer lugar, crear la carpeta `lib/services` si no lo hemos hecho, y copiar dentro directamente el fichero `comarcas_service.dart` que tenemos de la **práctica 2**. 

Con el fin de integrar este fichero en el proyecto, sólo tendréis que modificar los imports del principio, con el fin de importar las clases `Comarca` y `Provincia` de este proyecto. Además, como este servicio hace uso de **peticiones http**, deberemos incorporar la librería al proyecto con `dart pub add http`.

Este servicio no lo utilizaremos directamente en nuestra aplicación, sino que accederemos a él **a través de un repositorio**. Para ello, crearemos en la carpeta `lib/repository` un fichero `repository_comarcas.dart` , con la clase `RepositoryComarcas` , que implemente los siguientes métodos:

- `static Future<List<Provincia>> obtenerProvincias()` : Que devolverá la lista de provincias, haciendo uso de la funcionalidad proporcionada por el servicio `ComarcasService`.
  
- `static Future<List<dynamic>> obtenerComarcas(String provincia)` : Que devolverá una lista de objetos dinámicos con el nombre y la imagen de cada comarca de la provincia que se pide. Para ello, simplemente pedirá esta información al servicio y la devolverá.
  
- `static Future<Comarca?> obtenerInfoComarca(String comarca)` : Devolverá la información completa de la comarca solicitada, haciendo uso del método `infoComarca` de la clase del servicio de Comarcas.


<br>

<hr>

¿Por qué hacemos uso de un repositorio que simplemente realiza llamadas al servicio en lugar de utilizar directamente el servicio, como hemos hecho en la tarea 2?

Esta arquitectura propuesta es una práctica habitual, y presenta varias ventajas:

- En primer lugar, la clase que hace de Repository hace de capa de abstracción entre las fuentes de datos (ya sea una API o un acceso a bases de datos, por ejemplo) y el resto de la aplicación, de manera que la aplicación no necesita saber cómo se obtiene la información de la API.
  
- Esto nos permite un mayor desacoplamiento entre las diferentes partes de la aplicación. Si ahora cambiamos la forma de acceder a los datos (por ejemplo, si se cambia la API o hacemos uso de una base de datos local), sólo será necesario modificar la clase de repositorio. Además, resultará más fácil de probar, ya que podemos simular fuentes de datos y nos permitirá reutilizar código cuando disponemos de métodos similares a la API.

<hr>

<br>

## El servicio de información del tiempo

Además del servicio de información de Comarcas, implementaremos otro servicio (y su correspondiente repositorio) para la información del tiempo.

Para ello creamos la clase `WeatherService` en el fichero `lib/services/weather_service.dart`. 

Recuerdad que ya disponéis de esta implementación en el ejemplo del tiempo, visto en la unidad 5, que hace uso de la **API de OpenMeteo**. Lo que tendremos que hacer es copiar este fichero (o crearlo si no lo hemos hecho todavía), y generar el correspondiente repositorio.

Con el fin de crear el repositorio, creamos el fichero `lib/repository/repository_weather.dart` con la clase `RepositoryWeather`. Esta clase ofrecerá el método estático:

- `static Future<dynamic> obtenerClima({required double longitud, required double latitud }) async`: Que nos devolverá el resultado de invocar el mismo método del servicio, proporcionándole también la longitud y la latitud.

<br>

## Pantallas y Widgets

Las pantallas y la navegación a utilizar serán las que ya tenemos implementadas de tareas anteriores, con la diferencia de que ahora harán uso de la información que obtenemos mediante las peticiones HTTP. 

Las modificaciones concretas que habrá que implementar son:

- Sobre la pantalla `Provincias`, habrá que obtener las provincias mediante la función `obtenerProvincias` de la clase `RepositoryComarcas`.
  
- Como este método ahora nos devolverá un `Future`, habrá que hacer uso de un `FutureBuilder`, de manera que no se genere el contenido hasta que no se resuelva la petición. Mientras tanto se mostrará un indicador de progreso.
  
- Sobre la pantalla `Comarcas`, habrá que obtener las comarcas de la provincia en cuestión, haciendo uso de la función `obtenerComarcas`, del repositorio, y generar el contenido con un `FutureBuilder` cuando se resuelva la petición. Mientras no se resuelva se mostrará un indicador de progreso.
  
- Sobre la pantalla `infoComarca`, habrá que obtener la información de diversas fuentes. Por un lado, la información sobre la comarca seleccionada, mediante el método `obtenerInfoComarca`, y por otro, la información del tiempo actual, mediante la función `obtenerClima` del repositorio `RepositoryWeather`. Este método requerirá que le proporcionemos la latitud y la longitud que aparece en la comarca, por lo que no lo podremos invocar hasta que tengamos éstas. La forma de hacerlo será obteniendo esta información de la información detallada de la comarca. Como en las pantallas anteriores, se deberá hacer uso de los respectivos `FutureBuilder` con el fin de generar los contenidos de manera asíncrona.

## Consideraciones finales sobre el widget MyWeatherInfo

En la práctica anterior hemos hecho uso del widget `MyWeatherInfo`, que mostraba de manera estática información fija sobre el tiempo. 

Para esta práctica, esta información se obtendrá a partir de la API de información meteorológica, y su contenido será prácticamente el que se ha implementado en la clase `WidgetClima` del ejemplo del tiempo en la unidad 5. Por ello, podéis hacer uso directamente del contenido de esta clase, o adaptar el MyWeatherInfo que tenéis para que sea dinámico. 

Para ello, habrá que tener en cuenta los siguientes aspectos:

- Ahora el widget `MyWeatherInfo` será un widget con estado (`StatefulWidget`), en lugar de sin estado (`StatelessWidget`).
  
- El widget, definirá un par de propiedades de tipo `double?`, para la longitud y la latitud.
  
- Estas propiedades longitud y latitud se recibirán como argumentos con nombre en el constructor del widget. 
  
- El widget tendrá un estado asociado, que tendrá como propiedad un objeto `JSON (dynamic)` con la información que se recibirá en un futuro sobre el tiempo. Esta propiedad se definirá también como `late`, y se le asignará valor en la inicialización del estado (método `initState()`). Este valor se obtendrá a partir del método `obtenerClima` del repositorio correspondiente, proporcionándole como argumentos la longitud y la latitud almacenada en el widget.
  
- El método `build` generará el contenido del widget haciendo uso del `FutureBuilder` que dependerá del `Future` obtenido anteriormente.

<br>
<hr> 
Para la entrega de la práctica recordad hacer flutter clean y entregar un pdf con las dificultades encontradas y lo que queráis comentar sobre la práctica
<hr>