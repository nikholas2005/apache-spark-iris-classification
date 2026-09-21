# Taller Apache Spark - Clasificación con PySpark MLlib

Repositorio correspondiente al desarrollo del laboratorio de introducción a Apache Spark y aprendizaje automático distribuido con PySpark MLlib, desarrollado para la asignatura **Procesamiento de Datos a Gran Escala** en la **Pontificia Universidad Javeriana**.

---

## Información Académica

* **Institución:** Pontificia Universidad Javeriana (Bogotá, Colombia)
* **Facultad:** Ingeniería
* **Departamento:** Ingeniería de Sistemas
* **Asignatura:** Procesamiento de Datos a Gran Escala
* **Número de clase:** 1255
* **Estudiante:** Nikholas Barsky Angulo
* **Fecha:** Septiembre de 2026

---

## Descripción del Proyecto

El objetivo de este taller es implementar, ejecutar y documentar un flujo completo de clasificación supervisada utilizando **Apache Spark 4.2.0** y **PySpark MLlib**. El procesamiento se realizó sobre la infraestructura de clúster distribuido provista por la universidad, evaluando tres modelos de clasificación sobre el conjunto de datos clásico **Iris Flower Dataset**:

1. Árbol de Decisión (Decision Tree Classifier)
2. Bosque Aleatorio (Random Forest Classifier)
3. Clasificador Bayesiano Ingenuo (Naive Bayes Classifier)

---

## Infraestructura y Entorno de Ejecución

El desarrollo se llevó a cabo sobre máquinas virtuales desplegadas en la nube privada institucional con las siguientes características:

* **Nodo Driver / Máquina de trabajo:** NBDG44 (IP: `10.43.97.61`)
* **Sistema Operativo:** Rocky Linux 9.8 (Blue Onyx)
* **Motor de Procesamiento:** Apache Spark 4.2.0 con soporte Hadoop 3.5.0
* **Almacenamiento Compartido del Clúster:** Sistema de archivos de red NFS montado en `/opt/cluster`, permitiendo que todos los nodos ejecutores (*workers*) accedan de forma transparente a los conjuntos de datos en `/opt/cluster/data/`.
* **Acceso Remoto:** Conexión cifrada vía SSH y túnel de puertos para acceso a la interfaz web de Jupyter Notebook:
  ```bash
  # En la máquina virtual (terminal 1)
  python3 -m notebook --no-browser

  # En la máquina local (terminal 2 - Túnel SSH)
  ssh -L 8888:localhost:8888 estudiante@10.43.97.61
  ```

---

## Estructura del Repositorio

```text
.
|-- ApacheSparkTutorial.ipynb   # Cuaderno Jupyter con el desarrollo completo, ejecuciones y análisis
|-- README.md                   # Documentación general del proyecto y resultados
|-- javeriana_logo.png          # Escudo institucional incrustado en el informe
`-- archive/
    `-- iris.csv                # Conjunto de datos Iris (150 registros, 4 características, 3 clases)
```

---

## Flujo de Procesamiento en Spark

El flujo de trabajo implementado en el cuaderno comprende las siguientes etapas técnicas:

1. **Inicialización de SparkSession:** Creación de la sesión distribuida con asignación controlada de recursos (1 GB de memoria RAM por ejecutor y 4 núcleos de procesamiento).
2. **Carga y Almacenamiento en Caché:** Lectura distribuida de `iris.csv` desde el directorio NFS `/opt/cluster/data/iris.csv` con inferencia automática de tipos de datos (`inferSchema=true`) y persistencia en memoria principal (`cache()`).
3. **Exploración Inicial:** Verificación de tipos de datos, conteo de registros (150 filas) y análisis de balance de clases (50 observaciones por especie).
4. **Indexación de Etiquetas:** Aplicación de `StringIndexer` para codificar la variable categórica `species` en identificadores numéricos continuos (`species_indx`: 0.0, 1.0, 2.0).
5. **Ingeniería de Características con VectorAssembler:** Empaquetado de las cuatro medidas de las flores (`sepal_length`, `sepal_width`, `petal_length`, `petal_width`) en un único vector continuo (`features`). Esta operación se ejecuta de manera nativa dentro de la máquina virtual de Java (JVM) de Spark, evitando sobrecostos de serialización hacia procesos externos de Python.
6. **Estandarización de Datos:** Uso de `StandardScaler` para normalizar las características a varianza unitaria, garantizando que las variables con mayor dispersión no sesguen el proceso de optimización.
7. **Partición del Conjunto de Datos:** División aleatoria reproducible mediante semilla (`seed=12345`) en un 90% para entrenamiento y un 10% para evaluación independiente (conjunto de prueba de 11 muestras).
8. **Entrenamiento y Evaluación:** Ajuste de los modelos sobre el conjunto de entrenamiento y cálculo de la métrica de exactitud (*accuracy*) mediante `MulticlassClassificationEvaluator`.

---

## Resultados Obtenidos

La comparación de desempeño sobre el conjunto de prueba arrojó los siguientes resultados:

| Algoritmo de Clasificación | Exactitud (*Accuracy*) | Aciertos / Muestras |
| :--- | :---: | :---: |
| **Decision Tree Classifier** | 90.91% | 10 / 11 |
| **Random Forest Classifier** | 100.00% | 11 / 11 |
| **Naive Bayes Classifier** | 100.00% | 11 / 11 |

### Análisis Técnico de los Resultados

* **Decision Tree (90.91%):** El árbol individual presentó un único fallo de clasificación. En el conjunto de datos Iris, la clase *setosa* es linealmente separable de las demás, pero existe un solapamiento en las dimensiones de sépalos y pétalos entre las especies *versicolor* y *virginica*. Dado que los árboles univariados trazan hiperplanos perpendiculares a los ejes de las variables, los puntos cercanos a la frontera de separación son más susceptibles a una clasificación errónea.
* **Random Forest (100.00%):** Al generar un ensamble de 10 árboles de decisión (`numTrees=10`) entrenados con remuestreo aleatorio de observaciones y selección estocástica de variables (*bagging*), el modelo reduce drásticamente la varianza frente al árbol individual. El consenso mediante votación mayoritaria suaviza los límites de decisión y permite clasificar correctamente la totalidad de las muestras de prueba.
* **Naive Bayes (100.00%):** A pesar de fundamentarse en el supuesto teórico de independencia condicional entre variables predictoras (el cual rara vez se cumple de manera estricta en rasgos morfométricos correlacionados), la separación estadística entre las distribuciones de las tres clases fue suficiente para que la probabilidad a posteriori máxima señalara la clase correcta en todas las observaciones. Asimismo, el escalador preservó valores no negativos al no centrar la media (`withMean=False`), satisfaciendo la restricción matemática indispensable para el funcionamiento del modelo multinomial.

---

## Instrucciones de Reproducción Local

Para clonar y ejecutar este proyecto en un entorno local:

```bash
# 1. Clonar el repositorio
git clone <URL_DEL_REPOSITORIO>
cd <NOMBRE_DEL_REPOSITORIO>

# 2. Instalar las dependencias requeridas
pip install pyspark numpy pandas tabulate scikit-learn jupyter

# 3. Iniciar Jupyter Notebook
jupyter notebook
```

*Nota:* En caso de ejecutar el proyecto fuera del clúster de la universidad, debe actualizarse la variable `url` en la celda de carga de datos por la ruta local del archivo CSV (`archive/iris.csv`).
