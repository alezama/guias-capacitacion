---
title: Procesamiento de Datastreams
---

###Estructura de un Programa

Un programa en Apache Flink sigue los siguientes pasos
1. Obtiene un ambiente de ejecución y configurarlo.
2. Carga datos desde un origen de datos (datasource) o varios.
3. Modificar los datos utilizando operaciones de transformación de streams.
4. Almacena la información transformada en un archivo o base de datos, entregarla a otro sistema como un topic de 
Apache Kafka, la imprime en consola, etc. por medio de un deposito de datos (datasink).  
5. Ejecutar el programa. 

```java
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.api.windowing.time.Time

object WindowWordCount {
  def main(args: Array[String]) {
    // Paso 1: Se crea el ambiente de ejecución
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    // Se cargan los datos desde un datasource
    val text = env.socketTextStream("localhost", 9999)

    // Paso2: Se realizan operaciones de transformación de streams
    val counts = text.flatMap { _.toLowerCase.split("\\W+") filter { _.nonEmpty } }
      .map { (_, 1) }
      .keyBy(0)
      .timeWindow(Time.seconds(5))
      .sum(1)

    // Paso 3: Se imprime la información en consola (datasink)
    counts.print()

    // Paso 4: Se ejecuta el programa
    env.execute("Window Stream WordCount")
  }
}



```

##Obtener un ambiente de ejecución

Apache Flink permite obtener un ambiente de ejecución de al menos tres maneras.
1. Obtener un ambiente de ejecución ya existente.
2. Crear un ambiente local.
3. Crear un ambiente remoto.

```:java
// crea un ambiente de ejecución local
val ambLocal = StreamExecutionEnvironment.createLocalEnvironment()

// crea un ambiente de ejecución remoto
val ambRemoto = StremExecutionEnvironment.createRemoteEnvironment(
"host",                  // hostname del JobManager
5267,                   // puerto del JobManager
"path-to-JarFile.jar")   // ruta del archivo JAR que se deberá enviar al JobManager 
```

Una vez obtenido el ambiente de ejecución se podrán realizar configuraciones al mismo como instruir al ambiente para
que utilice semántica de tiempo, el nivel de paralelismo, la tolerancia a fallos, etc.  


##Origines de datos (Datasources)
Flink contiene **data sources** ya integrados y también permite la creación de **data sources** customs. Los 
siguientes son **data sources** ya integrados con Apache Flink.
1. Basados en socket
2. Basados en archivos - archivos de texto, archivos binarios, etc.

Estos data sources se agregan al ambiente obtenido. Por ejemplo:

```:java
ambLocal.socketTextStream("localhost",9999) // agrega un datasource leyendo desde un socket. 
ambLocal.readTextFile(path) // lee archivos de texto line apor linea 
```

Otros datasources pueden ser revisados en la documentación de Flink. 

##Transformación de datos
Las operaciones de transformación de datos convierten la información de un tipo a otro, pudiendo ser la salida cero o 
uno o mas data streams. 

Las operaciones de transformación de los datos se pueden dividir en:
1. Transformaciones básicas, que son transformaciones en eventos o registros individuales.
2. Transformaciones de KeyedStreams que son transformaciones hechas a eventos en el contexto de una llave.
3. Transformaciones de múltiples streams, las cuales mezclan múltiples streams de datos en uno o dividen un stream de 
datos en múltiples.
4. Transformaciones de particiones para reorganizar los eventos de un stream.


Los siguientes son algunas de las transformaciones de datos que maneja Flink. 

1. Map - transforma un registro del stream en exactamente otro registro posiblemente de diferente tipo. 
2. FlatMap  - toma un registro del stream  de información y lo transforma en uno, cero o multiples registros. FlatMap es 
una operación que generaliza las operaciones de Map y Filter, y puede realizar ambas. 
3. Filter - toma un record y evaluando una condición emite el record o lo elimina. 
4. KeyBy - parte la información de manera lógica mediante una llave, internamente utiliza una función hash para 
partir la información. Produce un flujo de datos del tipo KeyedDataStream.
5. Reduce -  Desenrolla la información de un KeyedDataStream al reducir el último valor reducido con el valor en curso.
Esta operación no cambia el tipo de dato del stream.
6. Fold - Desenrolla la información de un KeyedDataStream combinando el último valor doblado con el valor en curso. 
7. Aggregations - Flink posee varias funciones de agregación como son max, min, sum. También existen estas funciones 
sobre KeyedDataStream como son maxBy, minBy, donde la diferencia es que max obtiene el máximo valor en un stream y maxBy
obtiene la llave con el máximo valor. 
8. Window - La función de window permite agrupar KeyedDataStreams por tiempo u otra condición. Para poder crear ventanas 
sobre un stream es necesario asignar una llave con la cual se realizará la distribución y una función que realice la 
transformación sobre la ventana. 
9. WindowAll - Permite agrupar data streams. Esta operación se hace de manera secuencial y se ejecuta sobre data-streams 
no particionados. 
10. Union - La función de union permite la union de uno o mas data streams. Realiza la combinación en paralelo de la 
información. Ambos streams deben de ser del mismo tipo de datos.
11. Window join - Permite la unión de dos ventanas por medio de una llave común. 
12. Split - Esta función permite dividir un datastream en dos o más streams basados en algún criterio.
13. Select - Esta función permite seleccionar un stream especifico de un stream que haya sido divido, 
14. Project - Permite seleccionar un subconjunto de atributos de un stream y sólo enviar estos elementos al siguiente
stream para su procesamiento.
15. Connect - Esta función permite asociar dos streams de entrada que no son del mismo tipo, por ejemplo, unir eventos 
de dos tipos. 

##Ventanas

Flink posee distintos tipos de ventanas. Global windows son ventas que nunca terminan al menos que se provea de 
trigger el cual haga que la ventana corte. Tumbling windows son ventanas basadas en intervalos de tiempo, son fijas en 
longitud y no se superponen. Las tumbing windows son útiles para procesar la información en tiempos específicos. Sliding 
windows son coo tumbling windows pero si permiten superposición con las ventanas previas utilizando parámetros definidos 
por el usuario.  Session windows son utilizadas cuando las fronteras de la ventana se deciden respecto a los datos de 
entrada.

##KeyedStreams
Comúnmente se requiere el procesamiento de un grupo de eventos que poseen una característica compartida. Un KeyedStream es un
Stream el cual ha sido lógicamente particionados en substreams que comparten la misma llave. 

Transformaciones con estado (Stateful transformations) que se aplican sobre un KeyedStream leen y escriben al estado en el
mismo contexto lo cual permite que puedan ser procesados de manera conjunta. Un DataStream puede ser transformado en un 
KeyedDataStream mediante la operacion `keyBy`. 

##Rolling Aggregations
Las operaciones de agregación se aplican sobre KeyedDataStream y producen un stream que agrega elementos, por ejemplo, 
sum, minimum and maximum. Una operación de  rolling aggregation continua agregando elementos para cada llave observada.
Para cada evento, el operador actualiza el valor que se está agregando y emite un evento con el valor actualizado.

Las siguientes son las operaciones de rolling aggregation que proporciona Flink:
* sum() - realiza una operación de suma incremental sobre un campo.
* min() - obtiene el mínimo valor hasta el momento del campo especificado. 
* max() - obtiene el máximo valor hasta el momento del campo especificado.
* minBy() - obtiene el evento con el mínimo valor del campo especificado hasta el momento.
* maxBy() - obtiene el evento con el máximo valor del campo especificado hasta el momento.
 
 ```java
    val env = StreamExecutionEnvironment.getExecutionEnvironment

    val inputStream:DataStream[(Int,Int,Int)] = env.fromElements(
      (1,2,2), (2,3,1), (2,2,4), (1,5,3), (1,1,1), (2,7,2)
    )

    val resultStreamMin = inputStream.keyBy(0).min(1)
    val resultStreamMinBy = inputStream.keyBy(0).minBy(1)
    val resultStreamSum = inputStream.keyBy(0).sum( 1)

    // Print> (1,2,2) (1,2,2) (1,1,2) (2,3,1) (2,2,1) (2,2,1)
    // El elemento de la posición 1 con el menor valor hasta el momento,
    // por llave (posición 0) la posición 2 no está definida.
    resultStreamMin.print()
    // Print> (2,3,1) (2,2,4) (2,2,4) (1,2,2) (1,2,2) (1,1,1)
    // El menor elemento hasta el moemento con el menor valor de la
    // posición 1 por llave (posición 0).
    resultStreamMinBy.print()
    // Print> (1,2,2) (1,7,2) (1,7,2) (2,3,1) (2,3,1) (2,12,1)
    // La suma del elemento en la posiciön 1 hasta el momento, por llave
    // (posición 0) la posición 2 no está definida.
    resultStreamSum.print()

    env.execute()

```
 
##Particionamiento
Flink permite el particionamiento de los datos. Los KeyedStreams con la misma llave se encuentran en la misma partición,
aunque una partición puede compartir varios KeyedStreams con diferentes llaves.  

##Depósitos de Datos (datasinks)
Las aplicaciones de streaming de Flink normalmente envían sus resultados a un sistema externo como Apache Kafka, un
archivo o una base de datos. Flink posee una colección de datasinks que puede ser utilizado para escribir a varios 
sistemas. 

##Ejecución
Cuando la aplicación ha sido completamente definida, puede ser ejecutada llamando al método `StreamExecutionEnvironment.execute()`.
Los programas en Flink tiene ejecución lazy, es decir todas las transformaciones a los streams no son realizadas en su
lugar el ambiente de ejecución crea un plan de ejecución desde el punto en que los datasources son creados, nuevos datasources
son construidos a partir de las transformaciones y son escritos en los datasinks. Es en el momento en que se llega al punto 
de ejecución en el que ese plan es realmente ejecutado de manera local o remota.  

