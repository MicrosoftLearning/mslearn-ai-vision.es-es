---
lab:
  title: Detección y análisis de caras
  description: Use el servicio Face de Visión de Azure AI para implementar soluciones de análisis y detección de caras.
---

# Detección y análisis de caras

La capacidad de detectar y analizar caras de personas es una funcionalidad básica de la inteligencia artificial. En este ejercicio, explorará el servicio **Face** para trabajar con caras.

> **Nota**: Este ejercicio se basa en software de SDK de la versión preliminar, que podrían cambiar. Cuando ha sido necesario, hemos usado versiones específicas de paquetes; que pueden no ser las versiones disponibles más recientes. Puede que se produzcan algunos errores, comportamientos o advertencias inesperados.

Aunque este ejercicio se basa en el SDK de Python de Face de Visión de Azure, puede desarrollar aplicaciones de visión mediante varios SDK específicos del lenguaje, como:

* [Face de Visión de Azure AI para JavaScript](https://www.npmjs.com/package/@azure-rest/ai-vision-face)
* [Face de Visión de Azure AI para Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Vision.Face)
* [Face de Visión de Azure AI para Java](https://central.sonatype.com/artifact/com.azure/azure-ai-vision-face)

Este ejercicio dura aproximadamente **30** minutos.

> **Nota**: Las funcionalidades de los servicios de Azure AI que devuelven información de identificación personal están restringidas a los clientes a los que se les ha concedido [acceso limitado](https://learn.microsoft.com/legal/cognitive-services/computer-vision/limited-access-identity). Este ejercicio no incluye tareas de reconocimiento facial y se puede completar sin solicitar acceso adicional a características restringidas.

## Aprovisionamiento de un recurso de API de Azure AI Face

Si aún no tiene uno en la suscripción, deberá aprovisionar un recurso de API de Azure AI Face.

> **Nota**: En este ejercicio, usará un recurso de **Face** independiente. También puede usar los servicios de Azure AI Face en un recurso de varios servicios de los *servicios de Azure AI*, ya sea directamente o en un proyecto de *Fundición de IA de Azure*.

1. Abra [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`, e inicie sesión con sus credenciales de Azure. Cierre los mensajes de bienvenida o sugerencias que se muestran.
1. Seleccione **Crear un recurso**.
1. En la barra de búsqueda, busque `Face`, seleccione **Face** y cree el recurso con la siguiente configuración:
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Región**: *elige cualquier región disponible*
    - **Nombre**: *Un nombre válido para el recurso de Face*
    - **Plan de tarifa**: F0 gratis.

1. Cree el recurso y espere a que se finalice la implementación; a continuación, vea los detalles de la implementación.
1. Cuando se haya implementado el recurso, vaya a él y, en el nodo **Administración de recursos** del panel de navegación, vea su página **Claves y punto de conexión**. Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## Desarrollo de una aplicación de análisis facial con el SDK de Face

En este ejercicio, creará una aplicación cliente parcialmente implementada que usa el SDK de Azure Face para detectar y analizar las caras humanas de las imágenes.

### Preparación de la configuración de aplicación

1. En Azure Portal, use el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, y seleccione un entorno de ***PowerShell*** sin almacenamiento en su suscripción.

    Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

    > **Nota**: Si el portal le pide que seleccione un almacenamiento para conservar los archivos, elija **No se requiere una cuenta de almacenamiento**, seleccione la suscripción que usa y presione **Aplicar**.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. Cambie el tamaño del panel de Cloud Shell para que pueda ver la página **Claves y punto de conexión** del recurso de Face.

    > **Sugerencia**: Puede cambiar el tamaño del panel arrastrando el borde superior. También puede usar los botones de minimizar y maximizar para cambiar entre Cloud Shell y la interfaz principal del portal.

1. En el panel de Cloud Shell, escribe los siguientes comandos para clonar el repositorio de GitHub que contiene los archivos de código de este ejercicio (escribe el comando o cópialo en el Portapapeles y, a continuación, haz clic con el botón derecho en la línea de comandos y pega como texto sin formato):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repositorio, use el siguiente comando para ir a los archivos de código de la aplicación:

    ```
   cd mslearn-ai-vision/Labfiles/face/python/face-api
   ls -a -l
    ```

    La carpeta contiene archivos de código y configuración de aplicaciones para la aplicación. También contiene una subcarpeta **/images**, con algunos archivos de imagen para que la aplicación los analice.

1. Instale el paquete del SDK de Visión de Azure AI y otros paquetes necesarios ejecutando los siguientes comandos:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-face==1.0.0b2
    ```

1. Escriba el siguiente comando para editar el archivo de configuración de la aplicación:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Face (copiado de su página **Claves y punto de conexión** de Azure Portal).
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Adición de código para crear un cliente de API de Face

1. En la línea de comandos de Cloud Shell, escriba el siguiente comando para abrir el archivo de código de la aplicación cliente:

    ```
   code analyze-faces.py
    ```

    > **Sugerencia**: Es posible que quiera maximizar el panel de Cloud Shell y mover la barra de división entre la consola de la línea de comandos y el editor de código para ver mejor el código.

1. En el archivo de código, busque el comentario **Importar espacios de nombres** y agregue el código siguiente para importar los espacios de nombres que deberá usar el SDK de Visión de Azure AI:

    ```python
   # Import namespaces
   from azure.ai.vision.face import FaceClient
   from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection01
   from azure.core.credentials import AzureKeyCredential
    ```

1. En la función **Main**, observe que se ha proporcionado el código para cargar los valores de configuración y determinar la imagen que se va a analizar. A continuación, busque el comentario **Autenticar cliente de Face** y agregue el código siguiente para crear y autenticar un objeto **FaceClient**:

    ```python
   # Authenticate Face client
   face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key))
    ```

### Adición de código para detectar y analizar caras

1. En el archivo de código de la aplicación, en la función **Main**, busque el comentario **Especificar las características faciales que se van a recuperar** y agregue el código siguiente:

    ```python
   # Specify facial features to be retrieved
   features = [FaceAttributeTypeDetection01.HEAD_POSE,
                FaceAttributeTypeDetection01.OCCLUSION,
                FaceAttributeTypeDetection01.ACCESSORIES]
    ```

1. En la función **Main**, en el código que acaba de agregar, busque el comentario **Obtener caras** y agregue el código siguiente para imprimir la información de características faciales y llamar a una función que anota la imagen con el rectángulo de selección para cada cara detectada (en función de la propiedad **face_rectangle** de cada cara):

    ```Python
   # Get faces
   with open(image_file, mode="rb") as image_data:
        detected_faces = face_client.detect(
            image_content=image_data.read(),
            detection_model=FaceDetectionModel.DETECTION01,
            recognition_model=FaceRecognitionModel.RECOGNITION01,
            return_face_id=False,
            return_face_attributes=features,
        )

   face_count = 0
   if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')
        for face in detected_faces:
    
            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))
            print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
            print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
            print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
            print(' - Forehead occluded?: {}'.format(face.face_attributes.occlusion["foreheadOccluded"]))
            print(' - Eye occluded?: {}'.format(face.face_attributes.occlusion["eyeOccluded"]))
            print(' - Mouth occluded?: {}'.format(face.face_attributes.occlusion["mouthOccluded"]))
            print(' - Accessories:')
            for accessory in face.face_attributes.accessories:
                print('   - {}'.format(accessory.type))
            # Annotate faces in the image
            annotate_faces(image_file, detected_faces)
    ```

1. Examine el código que agregó a la función **Main**. Analiza un archivo de imagen y detecta las caras que contiene, incluidos los atributos relativos a la posición de la cabeza, la oclusión y la presencia de accesorios, como gafas de sol. Además, se llama a una función para anotar la imagen original con un rectángulo de selección para cada cara detectada.
1. Guarde los cambios (*CTRL+S*), pero mantenga abierto el editor de código por si tiene que corregir los errores tipográficos.

1. Cambie el tamaño de los paneles para que pueda ver más de la consola y, a continuación, escriba el siguiente comando para ejecutar el programa con el argumento *images/face1.jpg*:

    ```
   python analyze-faces.py images/face1.jpg
    ```

    La aplicación se ejecuta y analiza la siguiente imagen:

    ![Fotografía de una estatua de una persona.](../media/face1.jpg)

1. Observe la salida, que debe incluir el identificador y los atributos de cada cara detectada. 
1. Tenga en cuenta que también se genera un archivo de imagen denominado **detected_faces.jpg**. Use el comando **download** (específico de Azure Cloud Shell) para descargarlo:

    ```
   download detected_faces.jpg
    ```

    El comando download crea un vínculo emergente en la parte inferior derecha del explorador, que puedes seleccionar para descargar y abrir el archivo. La imagen debe tener un aspecto similar al siguiente:

    ![Imagen con la cara resaltada.](../media/detected_faces1.jpg)

1. Vuelva a ejecutar el programa, pero esta vez especificando el parámetro *images/face2.jpg* para extraer el texto de la imagen siguiente:

    ![Imagen de otra persona.](../media/face2.jpg)

    ```
   python analyze-faces.py images/face2.jpg
    ```

1. Descargue y vea el archivo **detected_faces.jpg** resultante:

    ```
   download detected_faces.jpg
    ```

    La imagen resultante debe tener este aspecto:

    ![Otra imagen con una cara resaltada.](../media/detected_faces2.jpg)

1. Ejecute el programa una vez más, pero esta vez especificando el parámetro *images/faces.jpg* para extraer el texto de esta imagen:

    ![Fotografía de ambas personas.](../media/faces.jpg)

    ```
   python analyze-faces.py images/faces.jpg
    ```

1. Descargue y vea el archivo **detected_faces.jpg** resultante:

    ```
   download detected_faces.jpg
    ```

    La imagen resultante debe tener este aspecto:

    ![Otra imagen con una cara resaltada.](../media/detected_faces3.jpg)

## Limpieza de recursos

Si ha terminado de explorar Visión de Azure AI, debe eliminar los recursos que ha creado en este ejercicio para evitar incurrir en costes innecesarios de Azure:

1. Abre Azure Portal en `https://portal.azure.com` y, en la barra de búsqueda superior, busca los recursos que creaste en este laboratorio.

1. En la página del recurso, selecciona **Eliminar** y sigue las instrucciones para eliminar el recurso. Una alternativa es eliminar todo el grupo de recursos para limpiar todos los recursos al mismo tiempo.
