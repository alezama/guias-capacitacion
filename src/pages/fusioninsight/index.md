---
title: FusionInsight
---

Huawei FusionInsight es una plataforma de big data para el almacenamiento, búsqueda y análisis de información. 

FusionInsight se encuentra dividido en cinco subproductos los cuales son FusionInsight HD, FusionInsight LibrA, 
FusionInsight Miner, FusionInsight Farmer y FusionInsight Manager.

**FusionInsight HD** es la plataforma de procesamiento distribuido de Big Data,  que integra diversas herramientas de 
Apache: Zookeeper, Hadoop, HBase, Loader, Hive, Hue, Oozie, Phoenix, Solr, Redis, Spark, Streaming, Kafka, Elk, Flink y
otros componentes.

**FusionInsight Manager** es la plataforma de análisis de información, utiliza FusionInsight HD como fuente de 
almacenamiento y mina información utilizando su tecnología de procesamiento en paralelo.

**FusionInsight Farmer** es un contenedor enterprise de bigdata que provee una plataforma unificada de manejo, 
desarrollo y ejecución de servicios enterprise.

**FusionInsight Manager** es una plataforma de bigdata para la operación y mantenimiento de clusters. Soporta la 
instalación y despliegue, monitoreo de alertas, administración de usuarios, administración de permisos, auditoría,
manejo de servicios, revisión de salud, detección de fallas, upgrades, y patching de clusters de gran escala.

**FusionInsight LibrA** es una base relacional de procesamiento masivo en paralelo. Esta diseñada y desarrollada para 
análisis de información estructurada y puede procesar información del orden de PB.  


 
### Arquitectura de FusionInsight Manager
El FusionInsight manager está compuesto de OMS y OMA. Los OMS son los nodos de administración en el sistema de O&M (Operation
and Maintenance). El FusionInsight manager tiene dos nodos OMS en modo activo y stand by. OMA es el nodo administrado en el 
sistema de O&M. Existen múltiples nodos OMA que componen el FusionInsight manager. 

El OMS contiene los siguientes componentes:
* **Web Service**  Montado sobre Tomcat. El Web Service proporciona una interface HTTPS para acceder al FusionInsight manager 
desde un explorador Web.
* **CAS** 