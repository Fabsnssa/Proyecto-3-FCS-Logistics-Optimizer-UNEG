**Logistics Optimizer (Smart Routing) - Proyecto de Ingeniería de Grafos**

Este proyecto implementa un sistema de gestión de logística diseñado para optimizar rutas de entrega considerando variables dinámicas como tráfico, costo y capacidad de carga, utilizando **Neo4j** y la librería **Graph Data Science (GDS)**.

**🛠️ Configuración de Entornos e Infraestructura**

1. Despliegue con Docker 

Para garantizar que el sistema funcione con todas las dependencias necesarias, se utiliza un archivo `docker-compose.yml` que prepara el terreno de forma masiva y eficiente.

**Configuración detallada del contenedor:** 

* **Imagen:** `neo4j:5.12` - Utiliza una versión específica compatible con las herramientas actuales de ingeniería de grafos.
*
  **Puertos:** `7474` para el cceso al **Neo4j Browser** para la visualización de grafos y `7687` para la comunicación vía Bolt para que el **Notebook de Python** interactúe con la base de datos.

* **Volúmenes:** Se configura la carpeta `./import:/import` como puerta de entrada para archivos de carga masiva.
* **Plugins Obligatorios:** Se activan `graph-data-science` y `apoc`, esenciales para el cálculo de rutas y algoritmos complejos.
* **Seguridad:** Se otorgan permisos totales a los procedimientos para evitar bloqueos durante la ejecución de algoritmos en Python.
* **Levantamiento:** Ejecutar el comando `docker-compose up -d` en la terminal.


**2. Entorno de Desarrollo Python (Notebook)** 

El desarrollo se realizó en **VS Code** utilizando la extensión de **Jupyter**.

* **Kernel:** Python versión 3.10.11.
* **Dependencias:** Instalación del driver oficial mediante `%pip install neo4j`.
* **Conexión:** Se establece mediante el protocolo `bolt://localhost:7687` con el usuario `neo4j` y contraseña `grafo123`.

---

**🚀 Proceso de Desarrollo y Carga de Datos** 

1. Optimización Inicial en Neo4j 

Antes de la carga, se configuraron restricciones de unicidad para garantizar que no existan IDs duplicados en los nodos de `Almacen`, `PuntoEntrega` e `Interseccion`, optimizando así el rendimiento del grafo.

2. Implementación del Script de Carga (Código 2) 

Este script realiza una limpieza total del grafo (`MATCH (n) DETACH DELETE n`) y crea el escenario logístico base con nodos interconectados por la relación `CONECTA_A`.

* **Variables dinámicas:** Se incluyen propiedades de distancia, `tiempo_estimado`, `estado_trafico` y `capacidad_max_ton` en cada ruta para permitir los desafíos técnicos de optimización.


3. Proyecciones de Grafo en Memoria (Códigos 3 y 4) 

Para ejecutar algoritmos de GDS en tiempo real, se crearon proyecciones del grafo en la memoria RAM.

* **Mapeo detallado:** Debido a los requisitos de GDS versión 2.6.9, se implementó un mapeo explícito de propiedades (`x`, `y` para nodos; distancia, tiempo y tráfico para relaciones).

---

**🧠 Algoritmos y Consultas Complejas** 

1. Comparativa de Algoritmos (Código 5) 

Se cumple el desafío técnico de comparar dos enfoques de enrutamiento:

* **Dijkstra:** Encuentra la ruta basándose puramente en la **distancia física** (km).
* **A* (A-Star):** Utiliza una heurística basada en las coordenadas geográficas y el **tiempo/tráfico** para una ruta inteligente.

2. Diccionario de Consultas (Código 6) 

Documentación de 5 consultas avanzadas que demuestran el poder de Cypher:

* **Análisis de Tráfico:** Uso de `WITH` para identificar tramos con nivel de alerta crítico (> 0.7).
* **Desglose Dinámico:** Uso de `UNWIND` para extraer propiedades masivas de los nodos.
* **Restricciones de Carga:** Filtrado de rutas en tiempo real que soportan camiones de 15 toneladas o más.
* **Costo de Ruta:** Implementación de la fórmula escrita en el PDF.
* **Auditoría GDS:** Procedimientos para verificar el estado y salud del grafo en memoria RAM.

---

**🔄 Persistencia y Reinicio del Sistema**

Es vital comprender qué información sobrevive al apagar el contenedor de Docker o reiniciar Neo4j:

* **Persistente (Disco):** Los nodos, relaciones, sus propiedades y las restricciones de unicidad se mantienen guardados.
* **Volátil (RAM):** La proyección `grafo_logistica` en memoria GDS se borra automáticamente al apagar el servicio.


Flujo de trabajo para reiniciar el proyecto: 

1. **Levantar Docker:** Ejecutar `docker-compose up -d`.
2. **⚠️ Tiempo de Espera Crítico:** Tras levantar el contenedor, **esperar al menos 60 segundos** antes de ejecutar cualquier celda en el Notebook. Neo4j necesita este tiempo para inicializar los plugins de GDS y APOC.
3. **Refrescar la Conexión (Driver):** Si el contenedor fue reiniciado, el objeto driver en Python puede perder el rastro del socket. Se recomienda volver a ejecutar la celda de configuración de conexión: `driver = GraphDatabase.driver(uri, auth=(user, password))`.
4. **Re-proyectar:** Es obligatorio ejecutar la función `forzar_proyeccion()`. Esto vuelve a cargar los datos del disco a la memoria RAM.
5. **Ejecutar:** Una vez re-proyectado, puedes correr las comparativas de algoritmos y el diccionario de consultas.
6. **Visualizar:** Ejecutar `MATCH (n) RETURN n` en el **Neo4j Browser** para validar visualmente el estado del grafo.

---

**Docente de la asignatura Base de datos 2:** Ing. Clinia Cordero - UNEG 

**Alumnos creadores del proyecto:** 
* Fabiana Martínez (sección 2).
* Cesar Abache (sección 2).
* Sebastián Ortiz (sección 2).
