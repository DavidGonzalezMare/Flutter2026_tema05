![Union europea](./images/union_europea.jpeg)  ![Generalitat](./images/generalitat.jpeg) ![Mare Nostrum](./images/mare_nostrum.png)


<br>

<a id="_apartado1"></a>

<br>

# Práctica 5. Future Builder

En esta tarea haremos evolucionar nuestra aplicación y obtendremos la información de los continentes y países **desde la nube**, utilizando la API que vimos en la **práctica 2**.

Como vimos en la unidad 2 y trabajamos en la práctica 2, los accesos a servicios web nos devuelven la información a través de Futures, de manera que ahora utilizaremos el widget `FutureBuilder` para construir los widgets que dependan de estos servicios.

Vamos a ver los aspectos más relevantes de la práctica.

<br>

## Estructura general del proyecto

La estructura general será similar a la siguiente:

```
lib
├── main.dart
├── data
│   ├── 
│   ├── repository
|   |    ├── paises_respository.dart
│   │    └── wheather_respository.dart
│   └── services 
│        ├── paises_api.dart
|        └── wheather_service.dart       
├── domain
│   └── entities
│       ├── continente.dart
│       └── pais.dart
├── themes
│      └── tema_paises.dart
└── ui
    ├── screens
    │   ├── continentes_screen.dart
    │   ├── infopais_detalle.dart
    │   ├── infopais_general.dart
    │   ├── launcher_screen.dart
    │   └── paises_screen.dart
    └──widgets
        ├── my_weather_info.dart
        └── my_circular_progress_indicator.dart

```
Tenemos:

- La carpeta `domain/entities` , con los modelos de `Continente` y `Pais`. Estas clases no sufrirán ningún cambio.
  
- La carpeta `ui/screens`, con las diferentes pantallas y widgets, que habrá que adaptar para trabajar con `Futures`. 
- 
- La carpeta `ui/widgets`. En ella implementaremos la clase `MyWeatherInfo`. Como veis, se ha incorporado una clase `MyCircularProgressIndicator`, que no es más que un indicador de progreso circular que hemos envuelto en un `Center` y un `SizedBox` para personalizar la estética del mismo.
  
- La carpeta `data/services`, que implementa la parte de acceso a los servicios, tanto de información de los países como de información meteorológica.
  
- La carpeta `data/repository`, que se encarga de obtener la información a través de los servicios. 

<br>

## Adaptación de Servicios y Repositorios

En la `práctica 2` creamos la clase `PaisesApi` , que nos ofrecía acceso a la información de la API mediante llamadas HTTP.

Esta clase nos ofrecía los métodos:

- `getContinentes`, que nos devuelve, en un `Future` de tipo JSON la lista de continentes,
  
- `getPaises`, que devolvía, en un `Future`, una lista de los países por continente.
  
- `infoPais`, que devolvía la información JSON de un país que ofrecía la API.
  
Vamos a incorporar ahora este servicio a nuestro proyecto. Para ello sólo será necesario, copiarlo en la carpeta `data/services`. 

Con el fin de integrar este fichero en el proyecto, sólo tendréis que modificar los imports del principio, con el fin de importar las clases `Continente` y `Pais` de este proyecto. Además, como este servicio hace uso de **peticiones http**, deberemos incorporar la librería al proyecto con `flutter pub add http`.

Este servicio no lo utilizaremos directamente en nuestra aplicación, sino que accederemos a él **a través de un repositorio**. Para ello, copiaremos o sustituiremos el fichero `data/repository/paises_repository.dart` con el correspondiente de la práctica 2.

Recordad que ahora necesitamos pasar la urlBase.


<br>

<hr>

¿Por qué hacemos uso de un repositorio que simplemente realiza llamadas al servicio en lugar de utilizar directamente el servicio?

Esta arquitectura propuesta es una práctica habitual, y presenta varias ventajas:

- En primer lugar, la clase que hace de Repository hace de capa de abstracción entre las fuentes de datos (ya sea una API o un acceso a bases de datos, por ejemplo) y el resto de la aplicación, de manera que la aplicación no necesita saber cómo se obtiene la información de la API.
  
- Esto nos permite un mayor desacoplamiento entre las diferentes partes de la aplicación. Si ahora cambiamos la forma de acceder a los datos (por ejemplo, si se cambia la API o hacemos uso de una base de datos local), sólo será necesario modificar la clase de repositorio. Además, resultará más fácil de probar, ya que podemos simular fuentes de datos y nos permitirá reutilizar código cuando disponemos de métodos similares a la API.

<hr>

<br>

## El servicio de información del tiempo

Además del servicio de información de Paises, implementaremos otro servicio (y su correspondiente repositorio) para la información del tiempo.

Para ello creamos la clase `WeatherService` en el fichero `lib/data/services/weather_service.dart`. 

Recuerdad que ya disponéis de esta implementación en el ejemplo del tiempo, visto en la unidad 5, que hace uso de la **API de OpenMeteo**. Lo que tendremos que hacer es copiar este fichero (o crearlo si no lo hemos hecho todavía), y generar el correspondiente repositorio.

Con el fin de crear el repositorio, creamos el fichero `lib/data/repository/wheather_respositry.dart` con la clase `WeatherRepository`. Esta clase ofrecerá el método estático:

- `static Future<dynamic> obtenerClima({required double longitud, required double latitud }) async`: Que nos devolverá el resultado de invocar el mismo método del servicio, proporcionándole también la longitud y la latitud.

<br>

## Pantallas y Widgets

Las pantallas y la navegación a utilizar serán las que ya tenemos implementadas de tareas anteriores, con la diferencia de que ahora harán uso de la información que obtenemos mediante las peticiones HTTP. 

Las modificaciones concretas que habrá que implementar son:

- Sobre la pantalla `Continentes`, habrá que obtener los continentes mediante la función `getContinentes` de la clase `PaisesRepository`.
  
- Como este método ahora nos devolverá un `Future`, habrá que hacer uso de un `FutureBuilder`, de manera que no se genere el contenido hasta que no se resuelva la petición. Mientras tanto se mostrará un indicador de progreso.
  
- Sobre la pantalla `Países`, habrá que obtener los países del continente en cuestión, haciendo uso de la función `getPaises`, del repositorio, y generar el contenido con un `FutureBuilder` cuando se resuelva la petición. Mientras no se resuelva se mostrará un indicador de progreso.
  
- Sobre la pantalla `infoPais`, habrá que obtener la información de diversas fuentes. Por un lado, la información sobre el pais seleccionado, mediante el método `getInfoPais`, y por otro, la información del tiempo actual, mediante la función `obtenerClima` del repositorio `RepositoryWeather`. Este método requerirá que le proporcionemos la latitud y la longitud que aparece en el pais, por lo que no lo podremos invocar hasta que las tengamos La forma de hacerlo será obteniendo esta información de la información detallada del país. Como en las pantallas anteriores, se deberá hacer uso de los respectivos `FutureBuilder` con el fin de generar los contenidos de manera asíncrona.

## Consideraciones finales sobre el widget MyWeatherInfo

En la práctica anterior hemos hecho uso del widget `MyWeatherInfo`, que mostraba de manera estática información fija sobre el tiempo. 

Para esta práctica, esta información se obtendrá a partir de la API de información meteorológica, y su contenido será prácticamente el que se ha implementado en la clase `WidgetClima` del ejemplo del tiempo en la unidad 5. Por ello, podéis hacer uso directamente del contenido de esta clase, o adaptar el `MyWeatherInfo` que tenéis para que sea dinámico. 

Para ello, habrá que tener en cuenta los siguientes aspectos:

- Ahora el widget `MyWeatherInfo` será un widget con estado (`StatefulWidget`), en lugar de sin estado (`StatelessWidget`).
  
- El widget, definirá un par de propiedades de tipo `double?`, para la longitud y la latitud.
  
- Estas propiedades longitud y latitud **se recibirán como argumentos** con nombre en el constructor del widget. 
  
- El widget tendrá un estado asociado, que tendrá como propiedad un objeto `JSON (dynamic)` con la información que se recibirá en un futuro sobre el tiempo. Esta propiedad se definirá también como `late`, y se le asignará valor en la inicialización del estado (método `initState()`). Este valor se obtendrá a partir del método `obtenerClima` del repositorio correspondiente, proporcionándole como argumentos la longitud y la latitud almacenada en el widget.
  
- El método `build` generará el contenido del widget haciendo uso del `FutureBuilder` que dependerá del `Future` obtenido anteriormente.

<br>
<hr> 
Para la entrega de la práctica recordad hacer flutter clean y entregar un pdf con las dificultades encontradas y lo que queráis comentar sobre la práctica
<hr>