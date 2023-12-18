# Satellites on Fire Challenge
La idea de este desafío es trabajar con una parte de las tecnologías utilizadas en el stack de Satellites on Fire para resolver algunos problemas relevantes para nuestros clientes. Estos son los pasos para poder comenzar el desafío:

1. Clonar este repositorio y guardarlo de forma **privada**.
2. Realizar los desafíos propuestos a continuación.
3. Compartí tu repositorio a **@ulilopezpacho** para poder acceder a las respuestas.
4. ¡Contactanos por correo electrónico para poder revisar el desafío!

Además, estamos abiertos a feedback sobre el desafío. ¡Esperemos que les guste!

# Introducción

Recibir información de incendios en tiempo real es muy importante para nuestros clientes, por lo que estamos constantemente trabajando para poder agregar nuevos satélites a nuestra plataforma. Por ello, se lanzó un nuevo satélite geoestacionario *FIRESAT23*, un nuevo satélite de última generación que nos permitirá obtener incendios de todo el mundo.

Para ello, cuenta con una API muy agradable, a la cual se le pueden solicitar los incendios del último día (y de días previos si se quiere también, pero en este caso no será necesario).

¡Ayúdanos a terminar de implementar este nuevo satélite para nuestra plataforma!

## Lenguaje de Programación

Para este desafío, se deberá usar Python 3. Es crucial que se use Python de forma **tipada** (https://docs.python.org/3/library/typing.html), a pesar de que no sea tenido en cuenta en tiempo de ejecución ya que ahorra errores y fomenta las buenas prácticas.

## El Entorno

Para poder resolver el desafío, utilizaremos AWS SAM (Amazon Web Services Serverless Application Model). Este es útil para poder crear, testear y deployar funciones Lambda en la nube de AWS. Esta tecnología es utilizada por nosotros para poder realizar un producto escalable y eficiente, ya que nuestra aplicación tiene una carga de procesamiento de datos elevada. Para más información, está disponible la documentación de AWS SAM en https://docs.aws.amazon.com/serverless-application-model/ y de AWS Lambda en https://docs.aws.amazon.com/lambda/. Se recomienda testear las funciones con el entorno de Docker recomendado por AWS, pero también se puede realizar de forma estándar invocando las funciones en Python.

# Consignas

## Parte A: Implementación de Nuevo Satélite

El equipo de desarrollo comenzó a implementar la carga de datos de este nuevo satélite, pero por razones estratégicas se te solicitó que termines de implementarlo. Para ello, deberás terminar de completar el código que obtiene los incendios de la API externa, realizar el procesamiento de los datos para normalizarlos al estilo de nuestra base de datos, y subirlos.

Para esta parte del desafío, es importante notar que la API del *FIRESAT23* devuelve los incendios de un día solicitado, pero pueden cargarse incendios más de una vez por día. Por eso, deberá de realizarse una lógica que evite subir los incendios ya cargados previamente. Para ello, la función recibirá en el evento de ejecución cuáles incendios habían sido cargados hasta ese momento para que solo se agreguen los nuevos.

Además, al momento de cargar los incendios, se debe de clasificarlos según el continente que ocurrieron, y se deben descartar los eventos que no ocurrieron en la superficie terrestre.

### Detalles Técnicos

La función a completar se encuentra en la carpeta ``Firesat23``. En ella, se mockea la base de datos DynamoDB para poder simular la subida de datos a nuestra base de datos. Se podrá utilizar la mayoría de las funciones de DynamoDB al igual que se haría con la API de AWS.

Por favor, no modificar los archivos ``api/firesat23.py`` ni ``database/init.py``. Para entender el formato de como subir los datos a la base de datos, en ``database/init.py`` se encuentra un item de ejemplo. Recomendamos usar la operación ``BatchWrite`` de DynamoDB ya que hace la subida de datos más rápida. Más información de como usar la base de datos en https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html y en https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/table/index.html (Este segundo link proporciona una interfaz agradable para operar con la base de datos. Recomendamos usarla envés de las APIs directas).

Una vez subida la información, se recomienda leer toda la base de datos y retornear los registros de la función para poder analizar el output (aunque quizás conviene remover esto para la parte D del desafío).

### ¿Cómo correr la función?

Recomendamos usar AWS SAM CLI, que permitirá simular el entorno de AWS Lambda en tu máquina local. También, se puede llamar directamente a la función, aunque no garantiza que vaya a funcionar correctamente en AWS Lambda. Para correr la función com SAM CLI, se debe usar el siguiente comando:

```bash
sam build Firesat23 && sam local invoke Firesat23 -e "events/firesatTime.json"
```

En el archivo ``"events/firesatTime.json"`` se puede introducir la hora a partir de la cual se agreguen los incendios a la base de datos.

## Parte B: Segmentación de Incendios

Actualmente, contamos con varios usuarios que utilizan nuestro sistema de alertas (disponible en https://app.satellitesonfire.com) para poder recibir notificaciones de los fuegos que ocurren en su territorio. Ahora, están interesados en saber la cantidad de incendios (compuestos por uno o más fuegos) que ocurrieron en un período de tiempo determinado en su territorio.

Cada usuario tiene un criterio distinto para considerar que dos fuegos son del mismo incendio. Por ende, luego de hablar con cada uno de ellos, el equipo técnico determinó que en cada solicitud se puedan indicar dos parámetros (elegidos por cada usuario al momento de pedir la información) que ayudarán a determinar esto:

- $d: int$, la distancia máxima (inclusive) la cual dos fuegos deben tener para ser considerados del mismo incendio. Por simplicidad, consideraremos que la distancia entre dos puntos (fuegos) en coordenadas geodésicas puede ser calculado utilizando la siguiente cuenta: $\sqrt{(x_1 - x_2)^2 + (y_1 - y_2)^2}$, siendo ambos puntos $(x_1,y_1)$ y $(x_2, y_2)$.
- $t: int$, el tiempo máximo (inclusive), en minutos, que deben tener dos incendios a distancia menor o igual a $d$ para ser consideradas del mismo incendio.

Notar que la relación entre fuegos es transitiva. Es decir, que si $A$ y $B$ pertenecen al mismo incendio, y también $B$ y $C$ pertenecen a un mismo incendio, entonces $A$ y $C$ pertenecen al mismo incendio, a pesar de que pueden $A$ y $C$ no estar a un tiempo o distancia menor o igual a $t$ o $d$ respectivamente.

Teniendo estos datos, se debe programar una función en Python ``segmentacionDeIncendios(fuegos: list[dict], d: float, t: float) -> tuple[int,list[list[str]]]`` que, dado un conjunto de fuegos y los valores $d$ y $t$, indique la cantidad de incendios que ocurrieron, y además, se indique a que incendio pertenece cada fuego.

### Input

La cantidad de fuegos va a recibir esta función va a ser siempre menor o igual a $2500$. Cada fuego va a ser un diccionario con los siguientes datos: ``{id: string, x: float, y: float, time: string}``, donde además ``time`` es de la forma ``"YYYY-MM-DDTHH:MM"``. Además, está garantizado que $d$ y $t$ son mayores a $0$, y los ``id`` son distintos.

### Output

Una tupla. Su primer valor debe ser la cantidad de incendios distintos, y su segundo valor una lista que contenga en listas separadas cada uno de los incendios, guardando los ``id`` de cada uno de los fuegos dentro de una misma lista cuando pertenecen a un mismo incendio.

### Ejemplo
```python
fuegos: list[dict] = [
  {"id": "0", "x": 0.0, "y": 0.0, "time":"2023-01-01T00:00"},
  {"id": "1", "x": 1.0, "y": 0.0, "time":"2023-01-01T00:00"},
  {"id": "2", "x": 0.0, "y": 1.0, "time":"2023-01-01T00:00"},
  {"id": "3", "x": 10.0, "y": 10.0, "time":"2023-01-01T00:00"},
  {"id": "4", "x": 10.0, "y": 11.0, "time":"2023-01-01T00:00"}
]

d: float = 10.0
t: float = 60.0

num, lista = segmentacionDeIncendios(fuegos, d, t)

# num -> 2
# lista -> [["0","1","2"],["3","4"]]
```


### Aclaraciones

**No utilizar librerías fuera de la librería estandar de Python. (Es decir, las instaladas con PIP)**

La solución debe ser lo suficientemente eficiente para que el usuario no se quede esperando más de unos segundos la respuesta del request. Notar que se usó el termino "fuego" para denotar un punto, y el término "incendio" para denotar un conjunto de puntos que cumplen las características del enunciado.

Completar este función en la carpeta ``FireSegmentation``. No hace falta implementar un entorno de AWS Lambda para esta parte del desafío, ¡ya que eso será la Parte C!

## Parte C: Implementación de API con la función

Ahora, necesitamos implementar una API para que los usuarios puedan acceder a la segmentación de incendios. Para eso, deberás implementar una función en AWS Lambda que reciba como query los valores $d$, $t$, y fechas entre las cuales se quiera calcular la cantidad de incendios. Luego, se deberá obtener estos incendios de la base de datos (simulada para los fines de este desafío) y obtener la segmentación de incendios, que deberá ser devuelta al usuario a través del request.

Completar este función en la carpeta ``FireSegmentationAPI``, copiando en la carpeta ``segmentation`` la resolución de la parte B del desafío. La información de los atributos de la base de datos pueden verse en la carpeta ``database/init.py``, que define la base de datos **distinto** a la parte A. Para esta parte, es necesario entender como hacer queries de forma eficiente en DynamoDB, por lo que recomendamos leer la documentación y entender cómo funciona tanto DynamoDB como el sistema de Pricing, ya que se busca que las queries sean eficientes tanto en tiempo como en costo. Además, para fines del desafío, se poblará la base de datos solo con incendios desde el ``2023-01-01`` hasta el ``2023-06-30``, por lo que las queries fuera de ese rango no devolverían nada. La ejecución de la función Lambda puede ser lenta ya que se poblará la base de datos al correr la función para simularla con Mock.

La API debe retornar un JSON que contenga los dos valores que devuelve la función de la parte B del desafío. Por ejemplo, para el mismo caso detallado en la Parte C, la función devería devolver a través de un request POST la siguiente información:

```json
{
  "num": 2,
  "lista": [["0","1","2"],["3","4"]]
}
```

En ``events/fireSegmentationEventRaw.json`` se muestra un ejemplo del contenido del llamado a la función junto a los parámetros inyectados por AWS Lambda (NO será necesario hacer un POST con todos esos parámetros). De todas formas, para probar la función, se pueden utilizar los siguientes comandos.

```bash
sam build FireSegmentationApi && sam local start-api
```

Luego, se puede llamar a la función en Postman con el contenido como se muestra en ``events/fireSegmentationEvent.json``.


## Parte D: Testing

Para finalizar el desafío, se deberán implementar casos de prueba tanto para las funciones de Python construidas como para las funciones Lambda. Esto es sumamente importante ya que nos permitirá ver tu habilidad armando casos de prueba y desarrollando buenas prácticas. Para esta sección, recomendamos usar ``pytest``. Completar los tests en la carpeta ``tests``, donde se encuentran ejemplos generados automáticamente al crear el proyecto. Para esta parte, a demás, para probar cosas con DynamoDB se puede mockear la base de datos directamente en los tests (para ello, sacar los comandos que mockean DynamoDB en cada una de las funciones Lambda). De esta forma, se podá corroborar lo que se guardó en la base de datos luego de la llamada a la función.

**¡Exitos con el desafío!**
