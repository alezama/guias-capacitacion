---
title: Apache Flink
---
## Apache Flink

Apache Flink es un framework para el procesamiento de flujos de datos en forma de stream y batch. 
Es un proyecto de código abierto.

Caracteristicas:

* Puede procesar flujos de datos masivos de manera distribuida.
* Se encuentra escrita en Java pero posee APIs en Java, Scala y Python.
* Puede utilizar flujos de datos de muchas fuentes y ejecutarse en varias plataformas de cluster.
* Posee una gran cantidad de librerías para aprendizaje de máquina, procesamiento de grafos y ejecución
de queries del tipo SQL.

Ejemplo de código en Flink en el lenguaje Scala para el conteo de palabras en un archivo de texto es el siguiente.

```:java

val file = env.readTextFile("file/path")

val count = file
    .flatMap(line => line.split(" "))
    .map(word => (word, 1))
    .groupBy(0)
    .sum(1)
```

Los pasos que se ejecutan el programa son los siguientes. Primero, lee un archivo de texto en la ruta "file/path"
Posteriormente, separa las lineas de texto en palabras, cada elemento del conjunto de datos es una palabra del texto 
original, después mapea cada palabra a una tupla en donde la llave es la palabra y el valor es el entero 1. Agrupa los 
valores por el primer elemento de la tupla, por lo que cada par con la misma palabra formara un mismo grupo. 
Finalmente, cuanta el número de palabras fueron usadas en el texto realizando la suma sobre los valores de la tupla. 

###Características Únicas de Apache Flink

Para Flink vuelve el procesamiento de streams de datos su principal foco en vez del procesamiento batch. Por lo 
tanto, en Flink el procesamiento batch es un caso especial del procesamiento de streams a diferencia de otros frameworks 
como Spark. Flink es flexible en la construcción de ventanas de datos, permitiendo definirlas por tiempo, por conteo, 
se pueden definir ventanas que se sobrepongan, que no se sobrepongan, ventanas customizadas, etc. Otra característica de 
Flink es su soporte a iteraciones, lo cual permite ejecutar las misma secuencia de operaciones sobre los datos multiples 
veces. Flink implementa su propio manejo de memoria, internamente Flink utiliza pare el manejo de memoria arreglos de 
bytes en lugar de relegar el manejo al colector de basura de la JVM. 

Las características anteriores le permiten a Flink tener un mejor desempeño con menores latencias. También es capaz de 
garantizar que la información se procesa exactamente una vez, en lugar de otros frameworks que sólo garantizan el 
procesamiento de la información al menos una vez. Finalmente, Flink realiza optimización de queries haciendo que su 
ejecución sea más rápida. 

Lista de características:
1. Alto desempeño y baja latencia sin configuración manual.
2. Ejecución exactamente una vez gracias a su sistema de checkpoint. Sin embargo para aplicaciones en donde el tiempo 
checpoint afecta el desempeño se puede lograr una ejecución de al menos una vez.
3. Ventanas de streaming flexibles. Se pueden diseñar ventanas por tiempo, conteo o sesiones. 
4. Tolerante a fallos. 
5. Manejo de memoria optimizado.
6. Procesamiento de streams y batchs en una sola plataforma.
7. Librerías de aprendizaje de máquina, procesamiento de grafos, datos relacionales, etc.
8. Semántica de eventos de tiempo, la cual ayuda a procesar streams cuando los eventos llegan a destiempo. 
 


###Arquitectura de Apache Flink

Flink tiene una arquitectura en forma de capas donde cada componente es parte de una capa específica. 

La base de la arquitectura de Flink son las fuentes de datos, es decir de donde es capaz de leer Flink los datos. Estas
incluyen archivos locales, o distribuidos como  HDFS, Amazon S3, etc. Flink puede a su vez tomar información de bases 
de datos como MongoDB, HBase y SQL. o de streams de datos como Kafka, RabbitMQ o Flume.

El segundo nivel de la arquitectura de Flink es su ejecución la cual puede hacerse de manera local, en un cluster con un 
manejador de recursos como YARN o Mesos,  o en modo standalone.

La siguiente capa de la arquitectura de Flink es el motor de ejecución distribuido de flujo de datos en stream, el cual 
unifica el procesamiento batch y el de stream de datos. 

Para interactuar sobre la capa de arquitectura se utiliza el Batch API o el Stream API de acuerdo al tipo de trabajo  que se 
desea ejecutar. Estas APIs envían los programas a la capa de arquitectura de Flink en la forma de **JobGraph** que son 
generadas cuando se compila un programa. 

Sobre las APIs, Flink posee un conjunto de librerías como son FlinkML para el procesamiento de aprendizaje de máquina, 
Table para implementar las aplicaciones como consultas SQL, Gelly para el procesamiento de grafos.      
  
###Ejecución de un programa en Apache Flink

Un programa en Apache Flink se envía a un **Job Client** el cual a su vez envía el programa a el **Job Manager** el
cual orquesta los recursos para su ejecución. Una vez que los recursos se encuentran disponibles el **Job Manager** cada
parte es enviado a los distintos **Task Mangers** para su ejecución. Mientras el trabajo se encuentra en ejecución los 
distintos **Task Managers** envían el estado de la ejecución al **Job Manger**.

Un programa en Apache Flink se puede dividir en **Source Operations** en la que se recibe la información, **Transformation 
Operations** como son Map() o groupBy() y finalmente **Sink Operation** que envían la información transformada a un 
archivo, base de datos o socket. 

