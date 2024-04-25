---
lab:
  title: Clasificación de imágenes con Custom Vision de Azure AI (heredado)
---

# Clasificar imágenes con Custom Vision de Azure AI

El servicio **Custom Vision de Azure AI** le permite crear modelos de visión del equipo que están entrenados con sus propias imágenes. Puede usarlo para entrenar modelos de *clasificación de imágenes* y *detección de objetos*, que más tarde puede publicar y consumir desde las aplicaciones.

En este ejercicio, usará el servicio Custom Vision para entrenar un modelo de clasificación de imágenes que pueda identificar tres tipos de fruta (manzana, plátano y naranja).

## Clonación del repositorio para este curso

Si aún no ha clonado el repositorio del código**mslearn-ai-vision** en el entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada en Visual Studio Code.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-vision` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**. Si aparece el mensaje *Se ha detectado un proyecto de función de Azure en la carpeta*, puede cerrarlo de forma segura.

## Creación de recursos de Custom Vision

Antes de poder entrenar un modelo, necesitará recursos de Azure para el *entrenamiento* y la *predicción*. Puede crear recursos de **Custom Vision** para cada una de estas tareas, o bien puede crear un único recurso de **Servicios de Azure AI** y usarlo para ello (o ambos).

En este ejercicio, creará recursos de **Custom Vision** para entrenamiento y predicción para que pueda administrar el acceso y los costes de estas cargas de trabajo por separado.

1. En una nueva pestaña del explorador, abra Azure Portal en `https://portal.azure.com` e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Seleccione el botón **&#65291;Crear un recurso**, busque *Custom Vision* y cree un recurso de **Custom Vision** con la siguiente configuración:
    - **Opciones de creación**: ambas
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa de entrenamiento**: F0
    - **Plan de tarifa de predicción**: F0

    > **Nota**: Si ya tiene un servicio Custom Vision F0 en su suscripción, seleccione **S0** en este caso.

3. Espere a que se creen los recursos, consulte los detalles de la implementación y observe que se aprovisionan dos recursos de Custom Vision, uno para el entrenamiento y otro para la predicción (evidente por el sufijo **-Prediction**). Para ver estos recursos, vaya al grupo de recursos en el que los creó.

> **Importante**: Cada recurso tiene su propio *punto de conexión* y sus *claves*, que se usan para administrar el acceso desde el código. Para entrenar un modelo de clasificación de imágenes, el código debe usar el recurso de *entrenamiento* (con su punto de conexión y su clave); y para usar el modelo entrenado para predecir clases de imágenes, el código debe usar el recurso de *predicción* (con su punto de conexión y su clave).

## Creación de un proyecto de Custom Vision

Para entrenar un modelo de clasificación de imágenes, debe crear un proyecto de Custom Vision basado en su recurso de entrenamiento. Para hacerlo, debe usar el portal de Custom Vision.

1. En Visual Studio Code, consulte las imágenes de entrenamiento en la carpeta **07-custom-vision-image-classification/training-images** donde clonó el repositorio. Esta carpeta contiene subcarpetas de imágenes de manzana, plátano y naranja.
2. En una nueva pestaña del explorador, abra el portal de Custom Vision en `https://customvision.ai`. Si se le solicita, inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure y acepte las Condiciones del servicio
3. En el portal de Custom Vision, cree un nuevo proyecto con la siguiente configuración:
    - **Nombre**: clasificar fruta
    - **Descripción** clasificación de imágenes de frutas
    - **Recurso**: *el recurso de Custom Vision que ha creado anteriormente*
    - **Tipos de proyecto**: Clasificación
    - **Tipos de clasificación**: multiclase (etiqueta única por imagen)
    - **Dominios**: comida
4. En el nuevo proyecto, haga clic en **\[+\] Agregar imágenes** y seleccione todos los archivos de la carpeta **training-images/apple** que vio anteriormente. Después, cargue los archivos de imágenes e incluya la etiqueta *apple*, como en este ejemplo:

![Cargar la manzana con la etiqueta apple](../media/upload_apples.jpg)
   
5. Repita el paso anterior para cargar las imágenes de la carpeta **banana** con la etiqueta *banana* y las imágenes de la carpeta **orange** con la etiqueta *orange*.
6. Explore las imágenes cargadas en el proyecto de Custom Vision, debería tener 15 imágenes de cada clase, como en este ejemplo:

![Imágenes de frutas etiquetadas: 15 manzanas, 15 bananas y 15 naranjas](../media/fruit.jpg)
    
7. En el proyecto de Custom Vision, sobre las imágenes, haga clic en **Train** para entrenar un modelo de clasificación con las imágenes etiquetadas. Seleccione la opción **Quick Training** y, después, espere a que se complete el entrenamiento (puede tardar alrededor de un minuto).
8. Una vez entrenado el modelo, compruebe las métricas de rendimiento *Precision*, *Recall* y *AP* (miden la precisión de la predicción del modelo de clasificación y sus valores deberían ser altos).

> **Nota**: Las métricas de rendimiento se basan en un umbral de probabilidad del 50 % para cada predicción (es decir, si el modelo calcula una probabilidad del 50 % o superior de que una imagen sea de una clase determinada, se predice esa clase). Puede ajustar esta opción en la parte superior izquierda de la página.

## Prueba del modelo

Ahora que ha entrenado el modelo, puede probarlo.

1. Sobre las métricas de rendimiento, haga clic en **Quick Test**.
2. En el cuadro **URL de la imagen**, escriba `https://aka.ms/apple-image` y haga clic en ➔
3. Consulte las predicciones obtenidas del modelo, la puntuación de probabilidad de *apple* debería ser la más alta, como en este ejemplo:

![Una imagen con una predicción de clase de una manzana](../media/test-apple.jpg)

4. Cierre la ventana **Quick Test**.

## Vea la configuración del proyecto

Al proyecto que ha creado se le ha asignado un identificador único, que deberá especificar en cualquier código que interactúe con él.

1. Haga clic en el icono *Configuración* (⚙) en la esquina superior derecha de la página **Rendimiento** para ver la configuración del proyecto.
2. En **General** (a la izquierda), anote el **identificador de proyecto** que identifica de forma única este proyecto.
3. A la derecha, en **Recursos**, observe que se muestran la clave y el punto de conexión. Estos son los detalles del recurso de *entrenamiento*. También puede obtener esta información viendo el recurso en Azure Portal.

## Uso de la API de *entrenamiento*

El portal de Custom Vision proporciona una cómoda interfaz de usuario que puede usar para cargar y etiquetar imágenes, y entrenar modelos. Sin embargo, en algunos escenarios puede que desee automatizar el entrenamiento de modelos mediante la API de entrenamiento de Custom Vision.

> **Nota**: En este ejercicio, puede elegir usar la API del SDK de **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **07-custom-vision-image-classification** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **train-classifier** y abra un terminal integrado. A continuación, instale el paquete Custom Vision Training mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.0
```

3. Consulte el contenido de la carpeta **train-classifier** y fíjese en que contiene un archivo para las opciones de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el punto de conexión y la clave del recurso de *entrenamiento* de Custom Vision, así como el id. del proyecto de clasificación que creó anteriormente. Guarde los cambios.
4. Tenga en cuenta que la carpeta **train-classifier** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: train-classifier.py

    Abra el archivo de código y revise el código que contiene, fijándose en los siguientes detalles:
    - Se importan los espacios de nombres del paquete instalado.
    - La función **Main** recupera los valores de configuración y usa la clave y el punto de conexión para crear un **CustomVisionTrainingClient** autenticado, que luego se usa con el identificador del proyecto para crear una referencia **Project** al proyecto.
    - La función **Upload_Images** recupera las etiquetas que se definen en el proyecto de Custom Vision y, a continuación, carga los archivos de imagen de las carpetas indicadas correspondientes al proyecto, asignando el identificador de etiqueta adecuado.
    - La función **Train_Model** crea una nueva iteración de entrenamiento para el proyecto y espera a que se complete el entrenamiento.
5. Vuelva al terminal integrado de la carpeta **train-classifier** y escriba el siguiente comando para ejecutar el programa:

**C#**

```
dotnet run
```

**Python**

```
python train-classifier.py
```
    
6. Espere a que el programa finalice. A continuación, vuelva al explorador y consulte la página **Imágenes de entrenamiento** del proyecto en el portal de Custom Vision (actualice el explorador si es necesario).
7. Compruebe que se han agregado al proyecto algunas imágenes etiquetadas nuevas. A continuación, vea la página **Rendimiento** y compruebe que se ha creado una nueva iteración.

## Publicación del modelo de clasificación de imágenes

Ya puede publicar su modelo entrenado para poder usarlo desde una aplicación cliente.

1. En el portal de Custom Vision, en la página **Rendimiento**, haga clic en **&#128504; Publicar** para publicar el modelo entrenado con la siguiente configuración:
    - **Nombre del modelo**: fruit-classifier
    - **Recurso de predicción**: *el recurso de **predicción** creado anteriormente que termina en -Prediction (<u>no</u> el recurso de entrenamiento)*.
2. En la parte superior izquierda de la página **Configuración del proyecto**, haga clic en el icono *Projects Gallery* (Galería de proyectos) (&#128065;) para volver a la página principal del portal de Custom Vision, donde debería aparecer su proyecto.
3. En la página principal del portal de Custom Vision, en la esquina superior derecha, haga clic en el icono de *configuración* (&#9881;) para ver la configuración de su servicio Custom Vision. A continuación, en **Recursos**, busque el recurso de *predicción* que termina en -Prediction (<u>no</u> el recurso de entrenamiento) para determinar sus valores de **Clave** y **Punto de conexión**. También puede consultar el recurso en Azure Portal para obtener esta información.

## Uso del clasificador de imágenes desde una aplicación cliente

Ahora que ha publicado el modelo de clasificación de imágenes, puede usarlo desde una aplicación cliente. Una vez más, puede optar por usar **C#** o **Python**.

1. En Visual Studio Code, en la carpeta **07-custom-vision-image-classification**, en la subcarpeta de su lenguaje preferido (**C-Sharp** o **Python**), haga clic con el botón derecho en la carpeta **test-classifier** y abra un terminal integrado. A continuación, escriba el siguiente comando específico del SDK para instalar el paquete de predicción de Custom Vision:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.0
```

> **Nota**: El paquete del SDK de Python incluye paquetes de entrenamiento y predicción, y es posible que ya esté instalado.

2. Expanda la carpeta **test-classifier** para ver los archivos que contiene, que se usan para implementar una aplicación cliente de prueba para el modelo de clasificación de imágenes.
3. Abra el archivo de configuración de la aplicación cliente (*appsettings.json* para C# o *.env* para Python) y actualice los valores de configuración que contiene para reflejar el punto de conexión y la clave del recurso de *predicción* de Custom Vision, el id. del proyecto de clasificación y el nombre del modelo publicado (que debe ser *fruit-classifier*). Guarde los cambios.
4. Abra el archivo de código de la aplicación cliente (*Program.cs* para C#, *test-classification.py* para Python) y revise el código que contiene, fijándose en los detalles siguientes:
    - Se importan los espacios de nombres del paquete instalado
    - La función **Main** recupera los valores de configuración y usa la clave y el punto de conexión para crear un **CustomVisionPredictionClient** autenticado.
    - El objeto de cliente de predicción se usa para predecir una clase para cada imagen de la carpeta **test-images**, especificando el identificador del proyecto y el nombre del modelo para cada solicitud. Cada predicción incluye una probabilidad para cada clase posible y solo se muestran las etiquetas predichas con una probabilidad superior al 50 %.
5. Vuelva al terminal integrado de la carpeta **test-classifier** y escriba el siguiente comando específico del SDK para ejecutar el programa:

**C#**

```
dotnet run
```

**Python**

```
python test-classifier.py
```

6. Vea la etiqueta y las puntuaciones de probabilidad para cada predicción. Puede ver las imágenes en la carpeta **test-images** para comprobar que el modelo las ha clasificado correctamente.

## Más información

Para obtener más información sobre la clasificación de imágenes con el servicio Custom Vision, consulte la [documentación de Custom Vision](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
