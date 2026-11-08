# Proyecto Final: Procesamiento Digital de Imágenes
## Detección de Estrés Hídrico y Simulación de Sensores Ópticos

Este repositorio contiene el desarrollo del proyecto final para el curso de **Procesamiento Digital de Imágenes**. 

Inspirados en la problemática planteada por [ATRIA Innovation](https://atriainnovation.com/blog/aplicaciones-camaras-hiperespectrales/) sobre la selección y dimensionamiento de sensores industriales, el proyecto responde a la siguiente pregunta:

> **"¿Se necesita realmente una cámara hiperespectral de 50.000 USD (con 448 bandas continuas) o basta con un sensor multiespectral económico (de 3 o 4 bandas estratégicas) para diagnosticar estrés hídrico en plantas?"**

El proyecto está desarrollado **100% mediante procesamiento de imágenes clásico** (sin modelos de inteligencia artificial ni redes neuronales), fundamentado en física óptica, álgebra de tensores, segmentación por histogramas y formulación de índices biofísicos.

---

## 🌿 Datos Utilizados

El estudio utiliza el dataset de fenotipado vegetal de **Zenodo** (*Agricultural Institute of Slovenia*):
* **Cultivo:** Plantas de papa (*Solanum tuberosum*).
* **Condiciones:** Plantas **Control** (`K`, bien regadas) frente a plantas con **Sequía** (`S`, déficit hídrico sostenido).
* **Formato:** Cubos hiperespectrales ENVI (`.hdr` y `.img`) de $64 \times 64$ píxeles y **448 canales continuos** ($409.8\text{ nm}$ a $2509.6\text{ nm}$, cubriendo el visible, NIR y SWIR).
* *Nota:* La carpeta con las imágenes pesadas (`imaging-5/`, ~8 GB) se encuentra excluida mediante `.gitignore` para mantener el repositorio ligero.

---

## 📓 Contenido del Cuaderno: `deteccion_estres_sensores.ipynb`

El archivo principal [`deteccion_estres_sensores.ipynb`](deteccion_estres_sensores.ipynb) implementa de manera secuencial y didáctica todo el flujo experimental dividido en 10 secciones:

### 1. Organización del Dataset y Separación por Planta
Identifica el tratamiento (`K` o `S`) y agrupa los recortes por ID de planta individual para asegurar comparaciones justas e independientes entre muestras de control y sequía.

### 2. Carga del Cubo Hiperespectral (Tema 1)
Función que abre el archivo ENVI, limpia valores nulos (`NaN` de saturación o fondo del sensor) asignándolos a $0.0$, y extrae el vector físico de longitudes de onda en nanómetros ($\lambda$).

### 3. Segmentación del Tejido Foliar con Otsu (Tema 3)
Aisla la hoja del fondo y sombras aplicando el algoritmo clásico de **Umbralización de Otsu** sobre la banda del Infrarrojo Cercano ($\sim 770\text{ nm}$), donde la reflectancia celular del mesófilo vegetal es máxima. Todas las mediciones posteriores se realizan exclusivamente sobre la máscara de tejido vivo.

### 4. Simulación de Tres Tipos de Sensores (Temas 1 y 2)
A partir de las 448 bandas disponibles, simula qué capturaría cada cámara:
* **Sensor 1 (RGB Visible):** Canales sintéticos Azul ($450\text{ nm}$), Verde ($555\text{ nm}$) y Rojo ($650\text{ nm}$).
* **Sensor 2 (Multiespectral de 4 Bandas):** Verde ($555\text{ nm}$), Rojo clorofila ($670\text{ nm}$), NIR ($850\text{ nm}$) y SWIR absorción de agua ($1450\text{ nm}$).
* **Sensor 3 (Hiperespectral Completo):** Las 448 bandas continuas de laboratorio.

### 5. Muestreo y Resolución Espacial (Tema 2)
Evalúa si una resolución espacial alta es indispensable para el diagnóstico, simulando el submuestreo espacial de $64 \times 64$ a $32 \times 32$ y $16 \times 16$ píxeles mediante promediado de vecindarios.

### 6. Cálculo de Índices Espectrales Biofísicos (Tema 4)
Calcula y visualiza en mapas 2D los índices de vegetación e hidratación:
* **ExG (Exceso de Verde):** $2G - R - B$ (para la cámara RGB).
* **NDVI (Vigor de Clorofila):** $\frac{R_{850} - R_{670}}{R_{850} + R_{670}}$ (para el sensor multiespectral).
* **NDWI (Contenido de Agua Foliar):** $\frac{R_{850} - R_{1450}}{R_{850} + R_{1450}}$ (aprovechando la fuerte absorción del agua en $1450\text{ nm}$).

### 7. Evaluación Estadística Grupal
Extrae y compila las distribuciones de los índices sobre más de **$169\,000$ píxeles de hoja** de decenas de plantas de control y sequía.

### 8. Cuantificación de Separabilidad: Criterio de Fisher (Tema 3a)
Compara matemáticamente los histogramas de plantas sanas vs estresadas mediante la **Razón de Separabilidad de Fisher ($J$)**:
$$J = \frac{(\mu_{\text{control}} - \mu_{\text{sequia}})^2}{\sigma^2_{\text{control}} + \sigma^2_{\text{sequia}}}$$
A mayor valor de $J$, menor es el solapamiento entre grupos y mayor es la certeza del diagnóstico.

### 9. Búsqueda Espectral Óptima en las 448 Bandas
Construye una matriz de calor $448 \times 448$ de diferencia de índice normalizado para encontrar el par de bandas $(\lambda_1, \lambda_2)$ más discriminante de todo el espectro (encontrando óptimos en $\sim 700.8\text{ nm}$ y $2428.0\text{ nm}$).

### 10. Conclusión y Respuesta al Problema de ATRIA
Genera la gráfica de barras comparativa final entre los sensores y resume formalmente las conclusiones del estudio.

---

## 📊 Resultados Principales

| Sensor Simulado | Banda / Índice Clave | Razón de Fisher ($J$) | Desempeño Diagnóstico |
| :--- | :--- | :--- | :--- |
| **Cámara RGB Comercial** | Exceso de Verde ($ExG$) | **$J = 0.0009$** | **Deficiente:** Campanas totalmente solapadas; no detecta estrés temprano. |
| **Multiespectral (Clorofila)** | NDVI ($670 + 850\text{ nm}$) | **$J = 0.0037$** | **Moderado:** Detecta caída de vigor fotosintético. |
| **Multiespectral (Agua en SWIR)** | NDWI ($850 + 1450\text{ nm}$) | **$J = 0.0100$** | **Óptimo:** Separación nítida, **11 veces superior al RGB**. |
| **Hiperespectral de Laboratorio** | Par Óptimo ($448$ bandas) | **$J \approx 0.0106$** | **Marginal:** Solo aporta un $\sim 6\%$ de mejora frente al multiespectral de 4 bandas. |

> **Conclusión del Proyecto:** **NO se necesita una cámara de 50.000 USD.**  
> Un sensor multiespectral agrícola de bajo costo con 4 bandas (Verde, Rojo, NIR y SWIR en $1450\text{ nm}$) resuelve con total nitidez el diagnóstico de estrés hídrico, reduciendo el costo en decenas de miles de dólares y evitando manejar volúmenes masivos de datos.

---

## 🛠️ Requisitos e Instalación

Para ejecutar los notebooks en tu entorno local:

```bash
# Crear y activar entorno virtual
python3 -m venv .venv
source .venv/bin/activate

# Instalar dependencias
pip install numpy matplotlib spectral ipykernel ipywidgets
```
