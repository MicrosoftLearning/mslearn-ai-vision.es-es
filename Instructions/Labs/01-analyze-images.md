---
lab:
  title: Análisis de imágenes
  description: 'Use Análisis de imágenes de Visión de Azure AI para analizar imágenes, sugerir leyendas y etiquetas, y detectar objetos y personas.'
---

# Análisis de imágenes

Visión de Azure AI es una capacidad de inteligencia artificial que permite a los sistemas de software interpretar la entrada visual mediante el análisis de imágenes. En Microsoft Azure, el servicio **Visión** de Azure AI proporciona modelos predefinidos para tareas comunes de Computer Vision, incluido el análisis de imágenes para sugerir subtítulos y etiquetas, la detección de objetos comunes y de personas. También puede usar el servicio Visión de Azure AI para quitar el fondo o crear un mate en primer plano de las imágenes.

> **Nota**: Este ejercicio se basa en software de SDK de la versión preliminar, que podrían cambiar. Cuando ha sido necesario, hemos usado versiones específicas de paquetes; que pueden no ser las versiones disponibles más recientes. Puede que se produzcan algunos errores, comportamientos o advertencias inesperados.

Aunque este ejercicio se basa en el SDK de Python de Visión de Azure, puede desarrollar aplicaciones de visión mediante varios SDK específicos del lenguaje, como:

* [Azure AI Vision Analysis para JavaScript](https://www.npmjs.com/package/@azure-rest/ai-vision-image-analysis)
* [Azure AI Vision Analysis para Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Vision.ImageAnalysis)
* [Azure AI Vision Analysis para Java](https://mvnrepository.com/artifact/com.azure/azure-ai-vision-imageanalysis)

Este ejercicio dura aproximadamente **30** minutos.

## Aprovisionar un recurso de Visión de Azure AI

Si aún no tiene uno en la suscripción, deberá aprovisionar un recurso de Visión de Azure AI.

> **Nota**: En este ejercicio, usará un recurso de **Computer Vision** independiente. También puede usar los servicios de Visión de Azure AI en un recurso de varios servicios de los *servicios de Azure AI*, ya sea directamente o en un proyecto de *Fundición de IA de Azure*.

1. Abra [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`, e inicie sesión con sus credenciales de Azure. Cierre los mensajes de bienvenida o sugerencias que se muestran.
1. Seleccione **Crear un recurso**.
1. En la barra de búsqueda, busque `Computer Vision`, seleccione **Computer Vision** y cree el recurso con la siguiente configuración:
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Región**: *Seleccione entre **Este de EE. UU.**, **Oeste de EE. UU.**, **Centro de Francia**, **Centro de Corea del Sur**, **Norte de Europa**, **Sudeste Asiático**, **Oeste de Europa** o **Este de Asia**\**
    - **Nombre**: *Un nombre válido para el recurso de Computer Vision*
    - **Plan de tarifa**: F0 gratis.

    \*Los conjuntos de características completas de Visión de Azure AI 4.0 solo están disponibles actualmente en estas regiones.

1. Active las casillas necesarias y cree el recurso.
1. Espere a que se complete la implementación y, a continuación, vea los detalles de la implementación.
1. Cuando se haya implementado el recurso, vaya a él y, en el nodo **Administración de recursos** del panel de navegación, vea su página **Claves y punto de conexión**. Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## Desarrollo de una aplicación de análisis de imágenes con el SDK de Visión de Azure AI

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa el SDK de Visión de Azure AI para analizar imágenes.

### Preparación de la configuración de aplicación

1. En Azure Portal, use el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, y seleccione un entorno de ***PowerShell*** sin almacenamiento en su suscripción.

    Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

    > **Nota**: Si el portal le pide que seleccione un almacenamiento para conservar los archivos, elija **No se requiere una cuenta de almacenamiento**, seleccione la suscripción que usa y presione **Aplicar**.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. Cambie el tamaño del panel de Cloud Shell para que pueda ver la página **Claves y punto de conexión** del recurso de Computer Vision.

    > **Sugerencia**: Puede cambiar el tamaño del panel arrastrando el borde superior. También puede usar los botones de minimizar y maximizar para cambiar entre Cloud Shell y la interfaz principal del portal.

1. En el panel de Cloud Shell, escribe los siguientes comandos para clonar el repositorio de GitHub que contiene los archivos de código de este ejercicio (escribe el comando o cópialo en el Portapapeles y, a continuación, haz clic con el botón derecho en la línea de comandos y pega como texto sin formato):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repositorio, use el siguiente comando para desplazarse e ir a la carpeta que contiene los archivos de código de la aplicación:   

    ```
   cd mslearn-ai-vision/Labfiles/analyze-images/python/image-analysis
   ls -a -l
    ```

    La carpeta contiene archivos de código y configuración de aplicaciones para la aplicación. También contiene una subcarpeta **/images**, que contiene algunos archivos de imagen para que la aplicación los analice.
    
1. Instale el paquete del SDK de Visión de Azure AI y otros paquetes necesarios ejecutando los siguientes comandos:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-imageanalysis==1.0.0
    ```

1. Escriba el siguiente comando para editar el archivo de configuración de la aplicación:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Computer Vision (copiado de su página **Claves y punto de conexión** de Azure Portal).
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Adición de código para sugerir una leyenda

1. En la línea de comandos de Cloud Shell, escriba el siguiente comando para abrir el archivo de código de la aplicación cliente:

    ```
   code image-analysis.py
    ```

    > **Sugerencia**: Es posible que quiera maximizar el panel de Cloud Shell y mover la barra de división entre la consola de la línea de comandos y el editor de código para ver mejor el código.

1. En el archivo de código, busque el comentario **Importar espacios de nombres** y agregue el código siguiente para importar los espacios de nombres que deberá usar el SDK de Visión de Azure AI:

    ```python
   # import namespaces
   from azure.ai.vision.imageanalysis import ImageAnalysisClient
   from azure.ai.vision.imageanalysis.models import VisualFeatures
   from azure.core.credentials import AzureKeyCredential
    ```

1. En la función **Main**, observe que se ha proporcionado el código para cargar los valores de configuración y determinar el archivo de imagen que se va a analizar. A continuación, busque el comentario **Autenticar el cliente de Visión de Azure AI** y agregue el código siguiente para crear y autenticar un objeto cliente de Visión de Azure AI (asegúrese de mantener los niveles de sangría correctos):

    ```python
   # Authenticate Azure AI Vision client
   cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key))
    ```

1. En la función **Main**, en el código que acaba de agregar, busque el comentario **Analizar imagen** y agregue el código siguiente:

    ```python
   # Analyze image
   with open(image_file, "rb") as f:
        image_data = f.read()
   print(f'\nAnalyzing {image_file}\n')

   result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.CAPTION,
            VisualFeatures.DENSE_CAPTIONS,
            VisualFeatures.TAGS,
            VisualFeatures.OBJECTS,
            VisualFeatures.PEOPLE],
   )
    ```

1. Busque el comentario **Obtener leyendas de imagen** y agregue el código siguiente para mostrar leyendas de imagen y leyendas densas:

    ```python
   # Get image captions
   if result.caption is not None:
        print("\nCaption:")
        print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))
    
   if result.dense_captions is not None:
        print("\nDense Captions:")
        for caption in result.dense_captions.list:
            print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))
    ```

1. Guarde los cambios (*CTRL+S*) y cambie el tamaño de los paneles para que pueda ver claramente la consola de la línea de comandos mientras mantiene abierto el editor de código. A continuación, escriba el siguiente comando para ejecutar el programa con el argumento **images/street.jpg**:

    ```
   python image-analysis.py images/street.jpg
    ```

1. Observe la salida, que debe incluir una leyenda sugerida para la imagen **street.jpg**, que tiene este aspecto:

    ![Una imagen de una calle concurrida.](../media/street.jpg)

1. Vuelva a ejecutar el programa, esta vez con el argumento **images/building.jpg** para ver la leyenda que se genera para la imagen **building.jpg**, que se parece a esta:

    ![Imagen de un edificio.](../media/building.jpg)

1. Repita el paso anterior para generar una leyenda para el archivo **images/person.jpg**, que tiene este aspecto:

    ![Imagen de una persona.](../media/person.jpg)

### Adición de código para generar etiquetas sugeridas

A veces puede ser útil identificar *etiquetas* relevantes que proporcionan pistas sobre el contenido de una imagen.

1. En el editor de código, en la función **AnalyzeImage**, busque el comentario **Obtener etiquetas de imagen** y agregue el código siguiente:

    ```python
   # Get image tags
   if result.tags is not None:
        print("\nTags:")
        for tag in result.tags.list:
            print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
    ```

1. Guarde los cambios (*CTRL+S*) y ejecute el programa con el argumento **images/street.jpg**. Observe que, además de la leyenda de la imagen, se muestra una lista de etiquetas sugeridas.
1. Vuelva a ejecutar el programa para los archivos **images/building.jpg** e **images/person.jpg**.

### Adición de código para detectar y localizar objetos

1. En el editor de código, en la función **AnalyzeImage**, busque el comentario **Obtener objetos de la imagen** y agregue el código siguiente para enumerar los objetos detectados en la imagen, y llame a la función proporcionada para anotar una imagen con los objetos detectados:

    ```python
   # Get objects in the image
   if result.objects is not None:
        print("\nObjects in image:")
        for detected_object in result.objects.list:
            # Print object tag and confidence
            print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        # Annotate objects in the image
        show_objects(image_file, result.objects.list)
    ```

1. Guarde los cambios (*CTRL+S*) y ejecute el programa con el argumento **images/street.jpg**. Observe que, además de la leyenda de la imagen y las etiquetas sugeridas, se genera un archivo denominado **objects.jpg**.
1. Use el comando **download** (específico de Azure Cloud Shell) para descargar el archivo **objects.jpg**:

    ```
   download objects.jpg
    ```

    El comando download crea un vínculo emergente en la parte inferior derecha del explorador, que puedes seleccionar para descargar y abrir el archivo. La imagen debe tener un aspecto similar al siguiente:

    ![Imagen con rectángulos se selección de objetos.](../media/objects.jpg)

1. Vuelva a ejecutar el programa para los archivos **images/building.jpg** e **images/person.jpg**, y descargue el archivo objects.jpg generado después de cada ejecución.

### Adición de código para detectar y localizar personas

1. En el editor de código, en la función **AnalyzeImage**, busque el comentario **Obtener personas de la imagen** y agregue el código siguiente para enumerar las personas detectadas con un nivel de confianza del 20 % o más, y llame a una función proporcionada para anotarlas en una imagen:

    ```Python
   # Get people in the image
   if result.people is not None:
        print("\nPeople in image:")

        for detected_person in result.people.list:
            if detected_person.confidence > 0.2:
                # Print location and confidence of each person detected
                print(" {} (confidence: {:.2f}%)".format(detected_person.bounding_box, detected_person.confidence * 100))
        # Annotate people in the image
        show_people(image_file, result.people.list)
    ```

1. Guarde los cambios (*CTRL+S*) y ejecute el programa con el argumento **images/street.jpg**. Observe que, además de la leyenda de la imagen, las etiquetas sugeridas y el archivo objects.jpg, se genera una lista de ubicaciones de persona y un archivo denominado **people.jpg**.

1. Use el comando **download** (específico de Azure Cloud Shell) para descargar el archivo **objects.jpg**:

    ```
   download people.jpg
    ```

    El comando download crea un vínculo emergente en la parte inferior derecha del explorador, que puedes seleccionar para descargar y abrir el archivo. La imagen debe tener un aspecto similar al siguiente:

    ![Imagen con rectángulos de selección para las personas detectadas.](../media/people.jpg)

1. Vuelva a ejecutar el programa para los archivos **images/building.jpg** e **images/person.jpg**, y descargue el archivo people.jpg generado después de cada ejecución.

   > **Sugerencia:** Si ve que el modelo ha devuelto rectángulos de selección que no tienen sentido, consulte la puntuación de confianza de JSON e intente aumentar el filtrado de puntuación de confianza en la aplicación.

## Limpieza de recursos

Si ha terminado de explorar Visión de Azure AI, debe eliminar los recursos que ha creado en este ejercicio para evitar incurrir en costes innecesarios de Azure:

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.

1. En la barra de búsqueda superior, busque *Computer Vision* y seleccione el recurso de Computer Vision que creó en este laboratorio.

1. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

