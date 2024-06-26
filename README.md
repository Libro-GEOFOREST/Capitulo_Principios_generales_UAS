<a href="https://zenodo.org/doi/10.5281/zenodo.10730831"><img src="https://zenodo.org/badge/765599600.svg" alt="DOI"></a>

Carlos MARTÍN CORTÉS, Borja GARCÍA PASCUAL, Mauricio ACUNA, Guillermo PALACIOS RODRÍGUEZ, Miguel Ángel LARA GÓMEZ, Rafael Mª NAVARRO CERRILLO

# Capitulo 17: Adquisición y procesado de imágenes sobre plataformas UAS

## Cálculo de Índices de vegetación a partir de imágenes multiespectrales y creación de mapas.

Este ejemplo se centra en el análisis de imágenes multiespectrales obtenidas por UAS para calcular índices de vegetación y crear mapas temáticos. A demás se aprenderá a como automatizar este proceso, todo se realizará desde el entorno de programación R.

#### Objetivos:
- Familiarizarse con las imágenes multiespectrales y su aplicación en la monitorización de la vegetación.
- Familiarización con los índices de vegetación más comunes.
- Aprender a calcular índices de vegetación y creación de funciones para poder calcularlos.
- Desarrollar habilidades en el manejo de Sistemas de Información Geográfica desde un entorno de programación para la creación de mapas temáticos basados en los índices calculados.
- Interpretación de los mapas generados.

#### Materiales Necesarios:
- Orto mosaicos multi-espectrales
- Software RStudio


### Instalación y carga de librerías

Para trabajar con gráficos y datos geoespaciales en R, primero debemos instalar, y luego cargar varias librerías esenciales.

En este ejemplo, utilizaremos ggplot2, terra y sf. A continuación, se muestra cómo instalar estas librerías (si aún no las tienes instaladas) y cómo cargarlas para su uso.

##### Instalación de las librerías

Si no tienes estas librerías instaladas en tu entorno de R, puedes instalarlas utilizando el comando *install.packages()*.

```r
#Instalamos librerias
install.packages("ggplot2")
install.packages("terra")
install.packages("sf")
install.packages('ggspatial')
```

##### Carga de las librerías

Una vez instaladas las librerías, podemos cargarlas en nuestro entorno de trabajo usando la función *library()*.

```r
# Cargamos librerias
library(ggplot2)
library(terra)
library(sf)
library(ggspatial)
```

##### ggplot2

Esta librería es parte del conjunto de herramientas tidyverse y se utiliza para crear gráficos de alta calidad en R. Ofrece una forma consistente y poderosa de crear una amplia variedad de gráficos de manera eficiente. Url de información: ([ttps://ggplot2.tidyverse.org/]

A continuación se puede observar el **CHEATSHEET**:

![](./Auxiliares/Cheatsheet_ggplot.png)

##### terra

Esta es una librería diseñada para el análisis y manejo de datos raster. Los datos raster son representaciones matriciales de datos espaciales. Url de información: [https://rspatial.org/pkg/]

##### sf

Esta librería facilita el manejo de datos espaciales simples, especialmente datos vectoriales. sf soporta el uso de geometrías (puntos, líneas , polígonos). Url de información: [https://r-spatial.github.io/sf/]

A continuación se puede observar el **CHEATSHEET**:

![](./Auxiliares/Cheatsheet_sf.png)

##### ggspatial

Esta librería proporciona funciones adicionales para trabajar con datos geoespaciales en ggplot2, permitiendo agregar elementos como coordenadas, escalas y mapas base a los gráficos creados con ggplot2. Url de información: [https://paleolimbot.github.io/ggspatial/]

### ¿Qué es una imagen multiespectral?

Para poder comenzar, primero realizaremos una breve explicación de que es una imagen multiespectral:

Una imagen multiespectral es una representación de una escena capturada en múltiples bandas espectrales. A diferencia de una imagen convencional, que contiene tres bandas (rojo, verde y azul), una imagen multiespectral puede incluir varias bandas adicionales, cada una capturando información en diferentes rangos del espectro electromagnético.

En el caso de este ejemplo trabajaremos con una imagen multiespectral que contiene cuatro bandas: 
- Banda 1: Rojo
- Banda 2: Verde
- Banda 3: Azul
- Banda 4: Infrarrojo

#### Apertura y visualización de una imagen multiespectral

En esta parte del ejemplo, comenzaremos por abrir una imagen multiespectral y visualizarla.

##### Carga de una imagen multiespectral

Primero, cargaremos la imagen multiespectral desde el archivo que se adjunta. Para poder cargar la imagen debemos de conocer la ruta donde tenemos esta imagen.

Esta misma imagen la podemos conseguir en el Dataset de Pix4Dcloud, se encuentra en el siguiente enlace: [https://cloud.pix4d.com/site/26624/dataset/91715/map?shareToken=41648cb2-a688-47a3-b4a2-21f21431c94b]

![](./Auxiliares/PIX4D.png)

```r
# Especificamos la ruta al archivo de la imagen multiespectral
ruta_imagen <- "D:/Ecublens_20160803_transparent_mosaic_group1.tif" #Adaptar a donde se haya realizado la descarga en el paso anterior
```

Una vez que tenemos la ruta, se procederá a cargar la imagen con la función rast() de la librería terra.

```r
# Abrir la imagen multiespectral utilizando la función rast()
imagen_multiespectral <- rast(ruta_imagen)
```

##### Carga de una imagen multiespectral

Una vez cargada la imagen, podemos visualizarla utilizando la función plot(). Esta función nos permite crear una representación gráfica de la imagen raster.

```r
# Visualizamos la imagen multiespectral
plot(imagen_multiespectral)
```

![](./Auxiliares/Multiespectral.png)

Como podemos observar directamente, nos da tres gráficos, pero no sabemos a que banda se refiere cada uno, por lo que procederemos a visualizarlos de forma individual:

```r
# Primero dividimos la pantalla en 2 filas y 2 columnas
par(mfrow = c(2, 2))

# Visualizamos las cuatro bandas de la imagen multiespectral

# Visualización de la banda 1
plot(imagen_multiespectral[[1]], main = "Banda 1 - Rojo")

# Visualización de la banda 2
plot(imagen_multiespectral[[2]], main = "Banda 2 - Verde")

# Visualización de la banda 3
plot(imagen_multiespectral[[3]], main = "Banda 3 - Azul")

# Visualización de la banda 4
plot(imagen_multiespectral[[4]], main = "Banda 4 - Infrarrojo")
```

![](./Auxiliares/Multiespectral2.png)

##### Visualizamos en color real

También podemos visualizar la imagen en color real con la función plotRGB(), para obtener la visualización en color real, utilizamos las tres bandas correspondientes a los colores rojo (R), verde (G) y azul (B).

```r
# Visualización en color real
plotRGB(imagen_multiespectral, r = 1, g = 2, b = 3)
```

![](./Auxiliares/Multiespectral3.png)

##### Visualizamos en falso color

En el caso de la visualización en falso color, comúnmente se utiliza la banda del infrarrojo cercano (NIR) junto con las bandas del rojo y verde. Esto resalta la vegetación, ya que las plantas reflejan fuertemente el infrarrojo cercano.

```r
# Visualización en falso color
plotRGB(imagen_multiespectral, r = 4, g = 1, b = 2)
```

![](./Auxiliares/Multiespectral4.png)

### Recorte de la imagen multiespectral para una zona de estudio

Siempre que trabajemos con imágenes multiespectrales, estas cubrirán un área mayor a la zona de estudio, por lo que lo primero que tendremos que realizar es el recorte de la zona de estudio de la imagen multiespectral.

El recorte nos permite enfocar nuestro análisis únicamente en el área de interés, reduciendo así la cantidad de datos y facilitando el procesamiento.

Para realizar el recorte, necesitamos definir una extensión (extensión espacial) que especifique los límites de la zona de estudio. Para esto utilizaremos un polígono en formato Shapefile (.shp) que representa la zona de estudio.

La creación de este polígono la podemos realizar desde un programa de Sistemas de Información Geográfica (SIG), como podría ser QGis.

##### Carga del archivo vectorial

Cargaremos el polígono en el entorno de R, con la función *read_sf()*.

```r
# Establecemos la ruta del polígono
ruta_poligono <-"D:/Zona_estudio.shp" #Adaptar al polígono generado en QGIS previamente

# Cargamos el polígono con la función read_sf()
poligono <- read_sf(ruta_poligono)
```

##### Creacción de función para recortar la Imagen

Una vez que cargamos el polígono de la zona de estudio, procedemos a realizar una función que recorte una imagen multiespectral utilizando este polígono.

La función que vamos a crear se llamará recortar_imagen().

Esta va a realizar los siguientes pasos: 
1. Comprobación del Sistema de Coordenadas: Verifica si la imagen raster y el polígono tienen el mismo sistema de coordenadas. En el caso de que los sistemas de coordenadas no coinciden, transforma el sistema de coordenadas del polígono para que coincida con el del raster.
2. Recorte del Raster: Se usa la función crop() para recortar la imagen raster a la extensión del polígono, eliminando las partes que están fuera del área de interés definida por el polígono.
3. Aplicación de la Máscara del Polígono: Se usa la función mask() para aplicar la máscara del polígono al raster recortado, asegurando que sólo se mantengan los datos dentro del polígono.

Por último, la función devuelve el raster recortado y enmascarado.

```r
# Creamos una función para recortar la imagen multiespectral
recortar_imagen <- function(raster){
   
  # Comprobación del Sistema de Coordenadas 
  if (st_crs(poligono) != crs(raster)) {
    # Transformación del sistema de coordenadas
    poligono  <- st_transform(poligono , crs(raster))
  }
  
  # Recorte del Raster
  raster_recortado <- crop(raster, poligono)
  
  # Aplicación de la Máscara del Polígono
  raster_recortado <- mask(raster_recortado, poligono)
  
  # Devolución del Raster recortado y enmascarado
  return(raster_recortado)
}
```

##### Recorte de la imagen

Procedemos a recortar la imagen con *recortar_imagen()*.

```r
imagen_recortada <- recortar_imagen(imagen_multiespectral)
```

##### Comprobamos de que se ha recortado correctamente

Realizamos una comprobación de que se ha recortado la imagen visualizandola.

```r
# Comprobamos de que se recorta correctamente
plotRGB(imagen_recortada, r = 1, g = 2, b = 3)
```

![](./Auxiliares/Recorte.png)

##### Guardado de la imagen recortada

Guardamos la imagen recortada con la función *write_raster()*.

```r
# Guardamos Imagen en formato .tiff
writeRaster(imagen_recortada, "D:/Imagen_recortada.tiff", overwrite=TRUE)
```

### Calculo de Índices de Vegetación

Los índices de vegetación son métricas calculadas a partir de datos multiespectrales que permiten evaluar la salud y la cobertura de la vegetación.

En esta sección, calcularemos tres de los índices de vegetación más comunes: el NDVI (Índice de Vegetación de Diferencia Normalizada), el EVI (Índice de Vegetación Mejorado) y el SAVI (Índice de Vegetación Ajustado por el Suelo).

##### Índice de Vegetación de Diferencia Normalizada (NDVI)

El NDVI es uno de los índices de vegetación más utilizados en teledetección para monitorear la salud de la vegetación. Utiliza las bandas del infrarrojo cercano (NIR) y el rojo (RED) para medir la cantidad y vigor de la vegetación verde.

Se calcula con la siguiente fórmula:

```math
NDVI = \frac{NIR - RED} {NIR + RED}
```

Los valores del NDVI oscilan entre -1 y 1. Valores positivos cercanos a 1 indican vegetación densa y saludable, mientras que valores cercanos a 0 indican vegetación escasa o estrés. Valores negativos generalmente corresponden a superficies de agua, nubes, nieve o áreas sin vegetación.

##### Índice de Vegetación Ajustado por el Suelo (SAVI)

El SAVI ajusta el NDVI para tener en cuenta la influencia del suelo en áreas con poca vegetación. Este ajuste es crucial en regiones donde el suelo expuesto afecta significativamente los valores del NDVI. La fórmula del SAVI es:

```math
SAVI = \frac{(NIR - RED)·(1 + L)} {(NIR + RED + L)}
```

donde: L es un factor de corrección que varía según la densidad de la vegetación. Comúnmente se utiliza un valor de 0.5.

Interpretación de los valores de SAVI:

- Valores Altos (Cercanos a 1): Indican vegetación densa y saludable. En estas áreas, la influencia del suelo es mínima.
- Valores Moderados (Entre 0.2 y 0.8): Representan áreas con vegetación moderada. Estas pueden ser praderas, cultivos en crecimiento o vegetación menos densa.
- Valores Bajos (Cercanos a 0): Indican áreas con poca o ninguna vegetación. En estas áreas, la influencia del suelo es significativa y el SAVI ajusta los valores para reflejar con mayor precisión la cantidad de vegetación presente.
- Valores Negativos: Pueden indicar superficies de agua, nubes, nieve o áreas sin vegetación.

##### Creación de funciones para calcular

Una vez que conocemos que es cada índice, podemos crear funciones en R, que posteriormente calcularán los índices.

Función *calculadora_NDVI()*: Esta función calcula el NDVI utilizando la fórmula estándar del índice.

```r
# Creamos la calculadora del NDVI
calculadora_NDVI <- function(raster){
  # Extracción de la banda roja
  banda_red <- raster[[1]] 
  
  # Extracción de la banda infraroja
  banda_nir <- raster[[4]] 
  
  # Calculo del NDVI
  ndvi <- (banda_nir - banda_red) / (banda_nir + banda_red)
  
  # Retorno de resultado
  return(ndvi)
}
```

Función *calculadora_SAVI()*: Esta función calcula el SAVI.

```r
# Creamos la calculadora de SAVI
# Se puede modificar el valor de L
calculadora_SAVI <- function(raster, L = 0.5) {
  # Extracción de la banda roja
  banda_red <- raster[[1]] 
  
  # Extracción de la banda infraroja
  banda_nir <- raster[[4]] 
  
  # Calculo del SAVI
  savi <- (banda_nir - banda_red) * (1 + L) / (banda_nir + banda_red + L)
  
  # Retorno de resultado
  return(savi)
}
```

##### Aplicación de la funciones de Índices de Vegetación

Se procede a calcular dichos índices de vegetación a partir de la imagen recortada.

```r
# Calculamos el NDVI de la imagen recortada
ndvi <- calculadora_NDVI(imagen_recortada)

# Calculamos el SAVI de la imagen recortada
savi <- calculadora_SAVI(imagen_recortada)
```

##### Visualizamos los resultados

En esta sección, representaremos gráficamente los diferentes Índices que hemos calculado anteriormente. La función plot() se utiliza para visualizar los indices en forma de un gráfico.

Visualización del NDVI:

```r
# Visualización del NDVI
plot(ndvi, main = "NDVI (Índice de Vegetación de Diferencia Normalizada)")
```

![](./Auxiliares/NDVI.png)

Visualización del SAVI:

```r
# Visualización del SAVI
plot(savi, main = "SAVI (Índice de Vegetación Ajustado por el Suelo)")
```

![](./Auxiliares/SAVI.png)

### Creación de mapas

En esta sección, crearemos mapas a partir de nuestros datos de índices de vegetación. Para ello, primero transformaremos los datos raster en un data frame.

Esta transformación facilita la manipulación y visualización de los datos con herramientas de gráficos como *ggplot2*.

##### Transformamos raster a data frame

El siguiente código convierte los datos del NDVI, que están en formato raster, a un data frame. Esto se hace utilizando la función *as.data.frame()*, y se incluyen las coordenadas x e y para cada valor del NDVI. Luego, renombramos las columnas del data frame para mayor claridad.

```r
# Convertimos a data frame
ndvi_df <- as.data.frame(ndvi, xy = TRUE)

# Modificamos los nombres de las columnas
colnames(ndvi_df) <- c("x", "y", "ndvi")
```

Hacemos lo mismo con el Índice SAVI:

```r
# Convertimos a data frame
savi_df <- as.data.frame(savi, xy = TRUE)

# Modificamos los nombres de las columnas
colnames(savi_df) <- c("x", "y", "savi")
```

##### Creación del mapa con ggplot2

En esta sección, utilizaremos ggplot2 para crear un mapa que visualice los valores de los índices.

Esto lo realizaremos de la siguiente forma:

1. Creamos el gráfico base: Se Utiliza *ggplot()* para iniciar el gráfico.
2. Se añaden los datos: Se usa *geom_raster()* para añadir los datos desde el data frame, mapeando las coordenadas x e y, y los valores de los indices a los colores del gráfico.
3. Configuración de la escala de colores: Se aplica *scale_fill_gradientn()* para definir una escala de colores que va de marrón (bajos valores) a verde (altos valores), con na.value para manejar valores faltantes y name para la leyenda.
4. Ajuste de coordenadas: Se usa *coord_fixed()* para asegurar que las unidades en el eje x y el eje y tengan la misma escala.
5. Tema minimalista: Se usa *theme_minimal()* para un diseño limpio y sencillo.
6. Añadimos títulos y etiquetas: Se usa *labs()* para agregar un título al mapa y etiquetas a los ejes.
7. Añadimos elementos de anotación: Se usa *annotation_scale()* para añadir una escala al mapa y *annotation_north_arrow()* para agregar una flecha de norte.
8. Personalización del tema: Configuración del borde del panel y las líneas de la cuadrícula.

Primero lo hacemos con el índice NDVI:

```r
# Creamos el gráfico base
ggplot() +
  # Se añaden los datos
  geom_raster(data = ndvi_df, aes(x = x, y = y, fill = get("ndvi"))) +
  
  # Configuración de la escala de colores
  scale_fill_gradientn(colors = c("brown", "yellow", "green"), 
                       na.value = "transparent",
                       name = "NDVI") +
  # Ajuste de coordenadas
  coord_fixed() +
  # Tema minimalista
  theme_minimal() +
  # Añadimos títulos y etiquetas
  labs(title = "Mapa de NDVI", x = "Longitud", y = "Latitud")+
  # Añadimos ecala
  annotation_scale() +
  # Añadimos flecha del norte
  annotation_north_arrow(location='tr')+
  # Personalización del tema
  theme(
    panel.border = element_rect(colour = "black", fill = NA, size = 1),
    panel.grid.major = element_line(colour = "black", size = 0.2, linetype = "dashed"),
    panel.grid.minor = element_blank())
```

![](./Auxiliares/NDVI2.png)

Y por último con el índice SAVI:

```r
ggplot() +
  geom_raster(data = savi_df, aes(x = x, y = y, fill = get("savi"))) +
  scale_fill_gradientn(colors = c("brown", "yellow", "green"), 
                       na.value = "transparent",
                       name = "SAVI") +
  coord_fixed() +
  theme_minimal() +
  labs(title = "Mapa de SAVI", x = "Longitud", y = "Latitud")+
  annotation_scale() +
  annotation_north_arrow(location='tr')+
  theme(
    panel.border = element_rect(colour = "black", fill = NA, size = 1),
    panel.grid.major = element_line(colour = "black", size = 0.2, linetype = "dashed"),
    panel.grid.minor = element_blank())
```

![](./Auxiliares/SAVI2.png)

En el caso de querer guardar los mapas en formato .pdf, lo primero que tendremos que realizar es la creación de un objeto que contenga el mapa.

En este caso, lo realizaremos directamente con los dos mapas de los índices.

```r
# Mapa NDVI
mapa_ndvi <- ggplot() +
  geom_raster(data = ndvi_df, aes(x = x, y = y, fill = get("ndvi"))) +
  scale_fill_gradientn(colors = c("brown", "yellow", "green"), 
                       na.value = "transparent",
                       name = "NDVI") +
  coord_fixed() +
  theme_minimal() +
  labs(title = "Mapa de NDVI", x = "Longitud", y = "Latitud")+
  annotation_scale() +
  annotation_north_arrow(location='tr')+
  theme(
    panel.border = element_rect(colour = "black", fill = NA, size = 1),
    panel.grid.major = element_line(colour = "black", size = 0.2, linetype = "dashed"),
    panel.grid.minor = element_blank())

# Mapa SAVI
mapa_savi <- ggplot() +
  geom_raster(data = savi_df, aes(x = x, y = y, fill = get("savi"))) +
  scale_fill_gradientn(colors = c("brown", "yellow", "green"), 
                       na.value = "transparent",
                       name = "SAVI") +
  coord_fixed() +
  theme_minimal() +
  labs(title = "Mapa de SAVI", x = "Longitud", y = "Latitud")+
  annotation_scale() +
  annotation_north_arrow(location='tr')+
  theme(
    panel.border = element_rect(colour = "black", fill = NA, size = 1),
    panel.grid.major = element_line(colour = "black", size = 0.2, linetype = "dashed"),
    panel.grid.minor = element_blank())
```

En el caso de que deseemos guardar los mapas lo podemos realizar con la función ggsave(). Con esto podremos guardar el mapa generado en un archivo PDF.

```r
# Guardamos el Mapa del Índice NDVI
ggsave("D:/Mapa_NDVI.pdf", plot = mapa_ndvi, width = 10, height = 8)

# Guardamos el Mapa del Índice SAVI
ggsave("D:/Mapa_SAVI.pdf", plot = mapa_savi, width = 10, height = 8)
```

### Automatización de la creación de mapas

Una vez que entendemos correctamente como podemos trabajar con una única imagen multiespectral en R, podemos pasar a la automatización del procedimiento.

La automatización es especialmente útil cuando se trabaja con múltiples imágenes multiespectrales, ya que permite procesar todas las imágenes de manera eficiente y consistente sin necesidad de intervención manual para cada una.

En este ejemplo lo intentaremos realizar de una forma eficiente, ordenada e ir guardando cada producto generado, por si en algún momento queremos acceder a un producto, ir directamente a él.

##### Creacción de funciones anteriores

Creamos las funciones que se han explicado anteriormente, cuando hemos trabajado únicamente con una imagen multiespectral.

```r
# Creamos una función para recortar la imagen multiespectral
recortar_imagen <- function(raster){
   
  # Comprobación del Sistema de Coordenadas 
  if (st_crs(poligono) != crs(raster)) {
    # Transformación del sistema de coordenadas
    poligono  <- st_transform(poligono , crs(raster))
  }
  
  # Recorte del Raster
  raster_recortado <- crop(raster, poligono)
  
  # Aplicación de la Máscara del Polígono
  raster_recortado <- mask(raster_recortado, poligono)
  
  # Devolución del Raster recortado y enmascarado
  return(raster_recortado)
}

# Creamos la calculadora del NDVI
calculadora_NDVI <- function(raster){
  # Extracción de la banda roja
  banda_red <- raster[[1]] 
  
  # Extracción de la banda infrarroja
  banda_nir <- raster[[4]] 
  
  # Calculo del NDVI
  ndvi <- (banda_nir - banda_red) / (banda_nir + banda_red)
  
  # Retorno de resultado
  return(ndvi)
}

# Creamos la calculadora de SAVI
# Se puede modificar el valor de L
calculadora_SAVI <- function(raster, L = 0.5) {
  # Extracción de la banda roja
  banda_red <- raster[[1]] 
  
  # Extracción de la banda infraroja
  banda_nir <- raster[[4]] 
  
  # Calculo del SAVI
  savi <- (banda_nir - banda_red) * (1 + L) / (banda_nir + banda_red + L)
  
  # Retorno de resultado
  return(savi)
}
```

##### Creación de carpeta de trabajo

Lo primero que realizaremos es la creación de una carpeta de trabajo específica para nuestro proyecto. Esto facilita la organización y gestión de todos los archivos relacionados con el análisis de índices de vegetación, como imágenes multiespectrales,imágenes recortadas, polígonos de interés y mapas generados.

Esto lo podemos realizar directamente desde el explorador de archivos o realizarlo directamente en R, con la función *dir.create()*.

###### Creamos una función para crear directorios

Antes de empezar a trabajar con nuestros datos, es importante asegurarnos de que las carpetas necesarias para almacenar los archivos y resultados no estén creadas.

Para ello, crearemos una función en R que verifique si un directorio no existe. En el caso de que no exista, procede a crearlo, y si no, nos imprimirá un mensaje diciendo que ese directorio existe.

```r
# Creamos la función
crear_directorio_si_no_existe <- function(directorio) {
  
  # Si la ruta del directorio no existe
  if (!dir.exists(directorio)) {
    # Creame la carpeta
    dir.create(directorio)
  
  # En el caso que no se cumpla la condición de que no existe, es decir existe
  } else {
    
    # Imprime ya existe
    print("Ya existe")
  }
}
```

###### Creación de la carpeta

Para comenzar, estableceremos la ruta para la carpeta principal de nuestro proyecto y utilizaremos la función *crear_directorio_si_no_existe()* para asegurarnos de que esta carpeta no existe y comenzar el proyecto.

```r
# Establecemos la ruta
directorio_proyecto <- "D:/Proyecto_multiespectral"

# Creamos la carpeta si no existe
crear_directorio_si_no_existe(directorio_proyecto)
```

###### Creación de subcarpetas

Para mantener todo bien organizado en nuestro proyecto, crearemos varias subcarpetas dentro de la carpeta principal del proyecto. Estas subcarpetas nos permitirán clasificar y gestionar mejor nuestros archivos, asegurando que cada tipo de dato se almacena en su lugar correspondiente.

A continuación, definimos las subcarpetas necesarias para nuestro proyecto:

1. Imagenes_multiespectrales: Para almacenar las imágenes multiespectrales originales.
2. Imagenes_multiespectrales_recortadas: Para almacenar las imágenes multiespectrales recortadas según el área de interés.
3. Polígonos: Para guardar los archivos de polígonos en formato .shp que definen las áreas de interés.
4. Indices: Para guardar los raster de los índices de vegetación calculados.
5. Mapas_indices: Para almacenar los mapas generados de los índices.

###### Creamos un conjunto con los nombres de las carpetas

Creamos un conjunto llamado carpetas que contiene los nombres de cinco carpetas diferentes que crearemos posteriormente.

```r
# Creamos un conjunto con los nombres de las carpetas
carpetas <- c("Imagenes_multiespectrales","Imagenes_multiespectrales_recortadas"
              , "Poligonos", "Mapas_indices", "Indices")
```

###### Creamos las carpetas con un bucle for

Utilizaremos un bucle para crear todas estas carpetas.

El funcionamiento de este bucle es el siguiente: - La variable carpeta tomará sucesivamente los valores de cada elemento del conjunto *carpetas*.

- Se construirá la dirección de la carpeta concatenando el *directorio_proyecto* con el nombre de la carpeta actual utilizando *paste0*(). Un ejemplo de cómo funciona sería que para la primera carpeta la concatenación quedaría: D:/Proyecto_multiespectral/Imagenes_multiespectrales
- Luego se llama a la función *crear_directorio_si_no_existe()* pasando la dirección como argumento, lo que creará la carpeta si no existe.

```r
# Por cada Carpeta en Carpetas
for (carpeta in carpetas) {
  
  # Construye la dirección
  direccion <- paste0(directorio_proyecto,"/",carpeta)
  
  # Créame la dirección si no existe
  crear_directorio_si_no_existe(direccion)
}
```

###### Incorporación de las imágenes multiespectral en la carpeta Imagenes_multiespectrales

El siguiente paso a crear las carpetas del proyecto, será la incorporación de las imágenes multiespectrales en la carpeta, para continuar con la automatización correctamente es necesario que estas imágenes sean capturadas con el mismo sensor o en su defecto que tengan el mismo orden de las bandas, es decir, que la banda 1 sea la roja, la bada 2 sea la verde, la banda 3 sea la azul y la banda 4 la infrarroja.

Una vez incorporadas las imágenes en las carpetas procedemos a crear una lista con todos los directorios completos de las imágenes. Para ello lo haremos de la siguiente manera:

1. Creamos una variable llamada *directorio_imagenes*, que contiene la ruta del directorio de la primera carpeta que hemos creado dentro del directorio del proyecto. Se construye concatenando el *directorio_proyecto* con el primer elemento del conjunto carpetas.
2. Utilizamos la función *list.files()* para listar todos los archivos *.tif* o *.tiff* en el directorio de imágenes multiespectrales (*directorio_imagenes*). El resultado se guarda en la variable *imagenes_multiespectrales*. En el caso de que tengamos las imágenes en otro tipo de formato se podria modificar para que listara las imágenes en otros formatos.

```r
# Creamos un directorio de la carpeta de las imágenes multiespectrales
directorio_imagenes <- paste0(directorio_proyecto, "/", carpetas[1])


# Listar todos los archivos .tif o .tiff en el directorio de las imágenes
imagenes_multiespectrales <- list.files(directorio_imagenes, 
                                        #Aqui es donde se debería de modificar para otros formatos
                                        pattern = "\\.tif$|\\.tiff", 
                                        full.names = TRUE)
}
```

Comprobamos que ha conseguido los directorios de las imágenes multiespectrales.

```r
# Imprimimos la lista para comprobar
print(imagenes_multiespectrales)
```

```r annotate
## [1] "D:/Proyecto_multiespectral/Imagenes_multiespectrales/Ecublens_20160803_transparent_mosaic_group1.tif"
## [2] "D:/Proyecto_multiespectral/Imagenes_multiespectrales/Imagen_Multi_2.tif"                             
## [3] "D:/Proyecto_multiespectral/Imagenes_multiespectrales/Imagen_Multi_4.tif"                             
## [4] "D:/Proyecto_multiespectral/Imagenes_multiespectrales/Imagen_Multi_5.tif"
```

###### Por cada imagen crearemos una subcarpeta donde iremos guardando el trabajo

En esta sección, para cada carpeta excepto en las imágenes multiespectrales, crearemos una subcarpeta donde iremos guardando los resultados del trabajo realizado.

Un ejemplo de cómo deben de aparecer una de las carpetas dentro de la carpeta *poligonos* es el siguiente: D:/Proyecto_multiespectral/Poligonos/Imagen_Multi_1

###### Crearemos una función para extraer el nombre

Para conseguir estas carpetas lo primero que debemos de hacer es extraer los nombres de las imágenes, sin el formato.

Para ello, crearemos la función *extraer_nombre()*, y lo haremos de la siguiente manera:

1. Definimos la función *extraer_nombre()*.
2. Utilizamos la función *basename()* para extraer el nombre del archivo de la ruta completa del archivo.
3. Utilizamos la función *file_path_sans_ext()* para quitar la extensión del nombre del archivo.
4. Devolvemos el nombre del archivo sin la extensión.

```r
extraer_nombre <- function(archivo) {
  
  # Extraemos el nombre del archivo con su extensión
  name <- basename(archivo)
 
  # Quitamos la extensión del nombre del archivo
  name <- tools::file_path_sans_ext(name)

  # Devolvemos el nombre del archivo sin la extensión. 
  return(name)
}
```

###### Creación de subcarpetas

Procedemos a crear las subcarpetas de una forma similar a la utilizada anteriormente. Pero en este caso tenemos que realizar un bucle dentro de otro.

Para ello tenemos que utilizar el siguiente código:

1. Se crea un nuevo conjunto *carpetas_sin_imagenes* que excluye la carpeta *“Imagenes_multiespectrales”* del conjunto original carpetas.
2. Tenemos que realizar un bucle que trabaje con cada elemento del conjunto de imágenes multiespectrales.
3. Extraemos el nombre de cada imagen con la función *extraer_nobre()*.
4. Hacemos un bucle interno que itera sobre cada carpeta en el conjunto *carpetas_sin_imagenes*.
5. Creamos la ruta para cada subcarpeta, utilizando el nombre de la imagen y la carpeta actual.
6. Creamos esta subcarpeta si no existe.

```r
# Creamos una lista de las carpetas, pero sin la carpeta de las Imagenes_multiespectrales
carpetas_sin_imagenes <- carpetas[carpetas!= "Imagenes_multiespectrales"]

# Para cada imagen en la lista de imagenes multiespectrales
for (imagen in imagenes_multiespectrales) {
  
  # Sacamos el nombre de la imagen
  nombre <- extraer_nombre(imagen)
  
  # Cara cada carpeta en la lista de las carpetas en el conjunto de carpetas sin Imagenes_multiespectrales
  for (carpeta in carpetas_sin_imagenes) {
    
    # Creamos la ruta de cada subcarpeta
    ruta_subcarpeta <- paste0(directorio_proyecto,"/",carpeta,"/", nombre)
    
    # Creamos la subcarpeta si no existe
    crear_directorio_si_no_existe(ruta_subcarpeta)
  }
}
```

###### Recorte de imágenes multiespectrales

Insertamos zonas de estudio en formato .shp en directorio_proyecto/poligono/nombre

Creamos las capas vectoriales en formatos .shp en un Sistema de Información Geográfica, como podría ser QGis.

Es necesario crear un archivo vectorial por cada imagen.

Hay tres cosas importantes que debemos de hacer cuando creamos los polígonos:

1. Deben de tener el mismo sistema de coordenadas para evitar problemas, aunque si no es el mismo se transformara al ejecutar *recortar_imagen()*.
2. Siempre en el *“id”* del polígono debe contener un número entero positivo, un ejemplo serían los siguientes numeros, 1, 2, 3, ……. , 100. Esto se podrá modificar en la tabla de atributos del archivo vectorial.
3. En el caso de imágenes con mas de una zona de estudio, no se deben de crear más archivos vectoriales, este script solo funcionara con un único archivo vectorial por imagen, pero si se pueden crear varios polígonos dentro del mismo archivo vectorial con diferentes Id, y así podremos estudiar diferentes zonas de estudio en el mismo archivo vectorial.

En la siguiente imagen se puede observar un ejemplo:

![](./Auxiliares/Recorte2.png)

Una vez que hemos creado la capa vectorial por cada imagen, la incorporamos en cada carpeta de cada carpeta de imagen dentro de la carpeta de Poligonos: directorio_proyecto/poligono/nombre.

Ahora automatizaremos el proceso de recorte de imágenes multiespectrales utilizando polígonos.

Lo realizaremos de la siguiente manera:

1. Por cada imagen multiespectral en *imagenes_multiespectrales*, el código extrae el nombre de la imagen y carga el archivo raster correspondiente.
2. Definimos el directorio donde se encuentran el archivo de los polígonos asociados a la imagen, y cargamos los mismos.
3. Extraemos los “id” de los polígonos de la tabla de atributos.
4. Por cada ID de polígono, selecciona el polígono correspondiente y recortamos la imagen multiespectral.
5. Definimos la ruta (Forma automatizada) donde se guardará la imagen recortada y guarda la imagen usando *writeRaster()*, asegurándose de que se puede sobrescribir un archivo existente.

```r
# Por cada imagen en imágenes
for (imagen_multiespectral in imagenes_multiespectrales) {
    
  # Extraemos el nombre de la imagen multiespectral
  nombre <- extraer_nombre(imagen_multiespectral)
  
  # Cargamos la imagen
  imagen <- rast(imagen_multiespectral)
  
  # Establecemos el directorio de la carpeta Poligonos/Imagen
  directorio_poligonos <- paste0(directorio_proyecto,"/Poligonos/", nombre)
  
  # Conseguimos la ruta del polígono
  ruta_poligono <- list.files(directorio_poligonos, pattern = "\\.shp",
                         full.names = TRUE)
  
  # Cargamos el polígono en el entorno, importante que sea únicamiente un
  # archivo .shp, si no dara Error
  poligonos <- read_sf(ruta_poligono)
  
  # Extraemos de la tabla de atributos la columna de id
  id_poligonos <- poligonos$id
  
  # Por cada id en id _poligono
  for (id in id_poligonos) {
    
    # Nos quedamos con el poligono "id" de poligonos
    poligono <- poligonos[id, ]
    
    # Recortamos la imagen multiespectral
    imagen_recortada <- recortar_imagen(imagen)
    
    # Creamos la ruta donde la guardaremos
    ruta_guardado <-paste0(directorio_proyecto,
                       "/Imagenes_multiespectrales_recortadas/", 
                       nombre, "/", nombre,"_poligono_",id,".tiff")
    
    # Guardamos La imagen
    writeRaster(imagen_recortada, 
                ruta_guardado, 
                overwrite=TRUE)
  }
}
```

###### Calculo de Índices de vegetación

El proceso de aplicación de índices de vegetación (como NDVI y SAVI) a las imágenes multiespectrales recortadas, incluye la creación de subcarpetas organizadas para almacenar los resultados y la evaluación de los índices.

Se realiza de la siguiente manera:

1. Se define la ruta donde se almacenan las imágenes multiespectrales recortadas.
2. Creación de un conjunto con los nombres de los índices que se calcularán (en este caso, NDVI y SAVI).
3. Por cada imagen multiespectral en la lista imagenes_multiespectrales, se extrae el nombre de la imagen y se define el directorio correspondiente a las imágenes recortadas.
4. Por cada índice, crea una subcarpeta específica dentro del directorio principal para almacenar los resultados.
5. Aplicación de Índices a las imágenes recortadas, para cada índice, se construye dinámicamente la llamada a la función de cálculo del índice correspondiente y se evalúa la expresión. Así conseguimos los índices calculados, una vez que tenemos el resultado del cálculo del índice, lo guardamos como un archivo raster en la subcarpeta correspondiente.

```r
# Establecemos el direcorio de las imagenes recortadas
directorio_recortadas <- paste0(directorio_proyecto,
                                "/Imagenes_multiespectrales_recortadas")

# Creamos un conjunto con los nombres de los índices
indices <- c("NDVI", "SAVI")

# Por cada imagen multiespectral en imagenes_multiespectrales
for (imagen_multiespectral in imagenes_multiespectrales) {
  
  # Extraemos el nombre
  nombre <- extraer_nombre(imagen_multiespectral)
  
  # Establecemos el directorio de las imagenes recortadas
  directorio_imagenes_recortadas <- paste0(directorio_recortadas, "/", nombre)
  
  # Sacamos una lista de las imágenes recortadas
  imagenes_multiespectrales_recortadas <- list.files(directorio_imagenes_recortadas, 
                                        pattern = "\\.tif$|\\.tiff", 
                                        full.names = TRUE)
  

  #########################Creación de subcarpetas##############################
  
  # Por cada índice en índices
  for (indice in indices) {
    
    # Creamos un directorio para las carpetas
    directorio_indice <- paste0(directorio_proyecto,"/Indices/", nombre, "/",indice)
    
    # Creamos las carpetas si no existen
    crear_directorio_si_no_existe(directorio_indice)
  
  }
  #########################Aplicación de índices################################
  
  # Por cada imagen recortada en imagenes_multiespectrales_recortadas
  for (imagen_recortada in imagenes_multiespectrales_recortadas) {
    
    # Cargamos la imagen
    imagen <- rast(imagen_recortada)
    
    # Extraemos el nombre de la imagen recortada, este incluye informacion del polígono
    nombre_imagen <- extraer_nombre(imagen_recortada)
    
    for (indice in indices) {
      
      # Creamos el directorio con el índice correspondiente
      directorio_indice <- paste0(directorio_proyecto,"/Indices/", nombre, "/",indice)
      
      # Construir la llamada a la función como una cadena
      expr <- paste0("calculadora_", indice, "(imagen)")
      
      # Evaluar la expresión y almacenar el resultado
      resultado <- eval(parse(text = expr))
      
      # Guardamos el archivo raster
      writeRaster(resultado, 
                  paste0(directorio_indice,"/",nombre_imagen,"_",indice,".tiff"), 
                  overwrite=TRUE)
    }
  }
}
```

###### Creación de mapas

El último paso de este ejemplo es la creación de mapas de forma automatizada, este proceso se organiza de la siguiente manera:

1. Definimos la ruta para las imágenes recortadas y creamos un conjunto con los nombres de los índices.
2. Creamos subcarpetas para cada índice dentro del directorio de mapas.
3. Listamos las imágenes recortadas y extraemos el nombre de cada imagen recortada.
4. Cargamos cada índice en formato raster, convertimos el raster a un data frame y creamos el mapa.
5. Definimos la ruta de guardado y guardamos el mapa en formato PDF.

```r
# Establecemos el directorio de las imagenes recortadas
directorio_recortadas <- paste0(directorio_proyecto,
                                "/Imagenes_multiespectrales_recortadas")

# Creamos un conjunto con los nombres de los índices
indices <- c("NDVI", "SAVI")

# Por cada imagen multiespectral en imagenes_multiespectrales
for (imagen_multiespectral in imagenes_multiespectrales) {
  
  # Extraemos el nommbre
  nombre <- extraer_nombre(imagen_multiespectral)
  
  # Establecemos el directorio de las imagenes recortadas
  directorio_imagenes_recortadas <- paste0(directorio_recortadas, "/", nombre)
  
  # Sacamos una lista de las imagenes recortadas
  imagenes_multiespectrales_recortadas <- list.files(directorio_imagenes_recortadas, 
                                        pattern = "\\.tif$|\\.tiff", 
                                        full.names = TRUE)
  
  # Por cada índice en índices
  for (indice in indices) {
    
    # Creamos un di5rectorio para cada indice dentro de la carpeta de mapas 
    directorio_indice <- paste0(directorio_proyecto,"/Mapas_indices/", nombre, "/",indice)
    
    # Creamos el directorio si no existe
    crear_directorio_si_no_existe(directorio_indice)
  }
  
  # Por cada imagen recortada en imagenes recortadas
  for (imagen_recortada in imagenes_multiespectrales_recortadas) {
    
    # Extraemos el nombre de la imagen
    nombre_imagen <- extraer_nombre(imagen_recortada)
    
    # Por cada índice en índices
    for (indice in indices) {
      
      # Creamos directorio indice 
      directorio_indice <- paste0(directorio_proyecto,"/Indices/", nombre, "/",indice)
      
      # Creamos directorio mapas índice
      directorio_mapas_indice <- paste0(directorio_proyecto,"/Mapas_indices/", nombre, "/",indice)
      
      # Abrimos el índice en formato raster
      raster <- rast(paste0(directorio_indice,"/",nombre_imagen,"_",indice,".tiff"))
      
      # Convertimos a data frame
      raster_df <- as.data.frame(raster, xy = TRUE)
      
      # Establecemos nombres de columnas
      colnames(raster_df) <- c("x", "y", paste0(indice))
      
      # Creamos el mapa, igual que en los apartados anteriores pero automatizado
      # Automatización con paste0(indice)
      mapa <- ggplot() +
        geom_raster(data = raster_df, aes(x = x, y = y, fill = get(paste0(indice)))) +
        scale_fill_gradientn(colors = c("brown", "yellow", "green"), 
                             na.value = "transparent",
                             name = paste0(indice)) +
        coord_fixed() +
        theme_minimal() +
        labs(title = paste0("Mapa de ",indice), x = "Longitud", y = "Latitud")+
        annotation_scale() +
        annotation_north_arrow(location='tr')+
        theme(
          panel.border = element_rect(colour = "black", fill = NA, size = 1),
          panel.grid.major = element_line(colour = "black", size = 0.2, linetype = "dashed"),
          panel.grid.minor = element_blank())
      
      # Ruta guardado en formato .pdf
      ruta_guardado <- paste0(directorio_mapas_indice,"/",
                              nombre_imagen,"_",indice,".pdf")
      
      # Guardamos mapa en pdf
      ggsave(ruta_guardado, plot = mapa, width = 10, height = 8)
    
    }
  }
}
```
