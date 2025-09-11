---
lab:
  title: Clasificación de imágenes con un modelo personalizado de Visión de Azure AI
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Clasificación de imágenes con un modelo personalizado de Visión de Azure AI

Visión de Azure AI te permite entrenar modelos personalizados para clasificar y detectar objetos con las etiquetas que especifiques. En este laboratorio, compilaremos un modelo de clasificación de imágenes personalizado para clasificar imágenes de fruta.

## Aprovisionar un recurso de Computer Vision

Si aún no tiene una en la suscripción, deberá aprovisionar un recurso de **Computer Vision**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
1. Seleccione **Crear un recurso**.
1. En la barra de búsqueda, busque *Computer Vision*, seleccione **Computer Vision** y cree el recurso con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elige entre Este de EE. UU., Oeste de EE. UU., Centro de Francia, Centro de Corea del Sur, Norte de Europa, Sudeste Asiático, Oeste de Europa o Este de Asia\**
    - **Nombre**: *escribe un nombre único*
    - **Plan de tarifa**: F0 gratis.

    \*Los conjuntos de características completas de Visión de Azure AI 4.0 solo están disponibles actualmente en estas regiones.

1. Active las casillas necesarias y cree el recurso.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

También necesitamos una cuenta de almacenamiento para almacenar las imágenes de entrenamiento.

1. En Azure Portal, busca y selecciona **Cuentas de almacenamiento** y crea una cuenta de almacenamiento con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *Elija el mismo grupo de recursos en el que creó el recurso de Custom Vision*.
    - **Nombre de cuenta de almacenamiento**: customclassifySUFFIX 
        - *Nota: Reemplace el token `SUFFIX` por sus iniciales u otro valor para asegurarse de que el nombre del recurso sea globalmente único.*
    - **Región**: *elige la misma región que usaste para el recurso de Servicios de Azure AI*
    - **Servicio principal**: Azure Blob Storage o Azure Data Lake Storage Gen 2
    - **Carga de trabajo principal**: otras
    - **Rendimiento**: Estándar
    - **Redundancia**: almacenamiento con redundancia local (LRS)

1. Cuando se haya implementado el recurso, seleccione **Ir al recurso**.
1. Habilita el acceso público en la cuenta de almacenamiento. En el panel izquierdo, navega hasta **Configuración** en el grupo **Configuración** y habilita *Permitir acceso anónimo de blobs*. Seleccione **Guardar**.
1. En el panel izquierdo, en **Almacenamiento de datos**, selecciona **Contenedores**, crea un nuevo contenedor denominado `fruit` y establece **Nivel de acceso anónimo** en *Contenedor (acceso de lectura anónimo para contenedores y blobs)*.

    > **Nota**: Si la opción **Nivel de acceso anónimo** está deshabilitada, actualiza la página del explorador.
   
## Clonación del repositorio para este curso

Los archivos de imagen para entrenar el modelo se han proporcionado en un repositorio de GitHub. Clonará el repositorio y cargará las imágenes en la cuenta de almacenamiento mediante Cloud Shell desde Azure Portal. 

> **Sugerencia**: Si ya ha clonado recientemente el repositorio **mslearn-ai-vision**, puede omitir la tarea de clonación. De lo contrario, siga estos pasos para clonar el repositorio en el entorno de desarrollo.

1. En Azure Portal, usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. En el panel de PowerShell, escribe los siguientes comandos para clonar el repo de GitHub para este ejercicio:

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. Una vez clonado el repositorio, vaya a la carpeta que contiene los archivos del ejercicio:  

    ```
   cd mslearn-ai-vision/Labfiles/02-image-classification
    ```

1. Ejecute el comando `code replace.ps1` y revise el código. Verás que reemplaza el nombre de la cuenta de almacenamiento para el marcador de posición en un archivo JSON (el archivo COCO) que usamos en un paso posterior.
1. Reemplaza el marcador *solamente en la primera línea* del archivo por el nombre de la cuenta de almacenamiento.
1. Después de reemplazar el marcador de posición, en el editor de código, use el comando **CTRL+S** o **haga clic con el botón derecho y seleccione Guardar** para guardar los cambios y, a continuación, use el comando **CTRL+Q** o **haga clic con el botón derecho y seleccione Salir** para cerrar el editor de código mientras mantiene abierta la línea de comandos de Cloud Shell.
1. Ejecute el script con el siguiente comando:

    ```powershell
    ./replace.ps1
    ```

1. Puedes revisar el archivo COCO para asegurarte de que el nombre de la cuenta de almacenamiento esté ahí. Ejecute `code training-images/training_labels.json` y vea las primeras entradas. En el campo *absolute_url*, deberías ver algo similar a *"https://myStorage.blob.core.windows.net/fruit/...*. Si no ves el cambio previsto, asegúrate de actualizar solo el primer marcador de posición en el script de PowerShell.
1. Cierre el editor de código.
1. Ejecute el comando siguiente, reemplazando `<your-storage-account>` por el nombre de la cuenta de almacenamiento, para cargar el contenido de la carpeta **training-images** en el contenedor `fruit` que creó anteriormente.

    ```powershell
    az storage blob upload-batch --account-name <your-storage-account> -d fruit -s ./training-images/
    ```

1. Abra el contenedor `fruit` y compruebe que los archivos se cargaron correctamente.

## Creación de un proyecto de entrenamiento de modelo personalizado

Ahora, crearás un nuevo proyecto de entrenamiento para la clasificación de imágenes personalizada en Vision Studio.

1. En el explorador web, navega hasta `https://portal.vision.cognitive.azure.com/` e inicia sesión con la cuenta Microsoft en la que creaste el recurso de Azure AI.
1. Selecciona el icono **Personalizar modelos con imágenes** (lo puedes encontrar en la pestaña **Análisis de imágenes** si no se muestra en la vista predeterminada).
1. Selecciona la cuenta de Servicios de Azure AI que has creado.
1. En el proyecto, selecciona **Agregar nuevo conjunto de datos** en la parte superior. Configure con la siguiente configuración:
    - **Nombre del conjunto de datos**: training_images
    - **Tipo de modelo**: clasificación de imágenes
    - **Selecciona Contenedor de Azure Blob Storage**: selecciona **Seleccionar contenedor**
        - **Suscripción**: *suscripción de Azure*
        - **Cuenta de almacenamiento**: *la cuenta de almacenamiento que creaste*
        - **Contenedor de blobs**: fruta
    - Selecciona la casilla para permitir que Vision Studio se lea y escriba en Blob Storage.
1. Selecciona el conjunto de datos **training_images**.

En este momento de la creación del proyecto, normalmente seleccionas **Crear etiquetado de datos de Azure ML** y etiquetas las imágenes, lo que genera un archivo COCO. Te animamos a que lo intentes si tienes tiempo, pero para este laboratorio ya hemos etiquetado las imágenes para ti y hemos facilitado el archivo COCO resultante.

1. Selecciona **Agregar archivo COCO**
1. En el menú desplegable, selecciona **Importar archivo COCO desde un contenedor de blobs**
1. Puesto que ya conectaste el contenedor denominado `fruit`, Vision Studio lo busca para un archivo COCO. Selecciona **training_labels.json** en el menú desplegable y agrega el archivo COCO.
1. Navega hasta **Modelos personalizados** a la izquierda y selecciona **Entrenar un nuevo modelo**. Use la configuración siguiente:
    - **Nombre del modelo**: classifyfruit
    - **Tipo de modelo**: clasificación de imágenes
    - **Elegir conjunto de datos de entrenamiento**: training_images
    - Deja el resto tal como está y haz clic en **Entrenar modelo**

El entrenamiento puede tardar algún tiempo: el presupuesto predeterminado es de hasta una hora, pero, para este conjunto de datos pequeño suele ser mucho más rápido que eso. Selecciona el botón  **Actualizar** cada dos minutos hasta que el estado del trabajo sea *Correcto*. Seleccione el modelo.

Aquí, puedes ver el rendimiento del trabajo de entrenamiento. Revisa la precisión del modelo entrenado.

## Prueba del modelo personalizado

El modelo se ha entrenado y está listo para probarse.

1. En la parte superior de la página de tu modelo personalizado, selecciona **Probarlo**.
1. Selecciona el modelo **classifyfruit** en la lista desplegable especificando el modelo que deseas usar y ve a la carpeta **02-image-classification\test-images**.
1. Selecciona cada imagen y visualiza los resultados. Selecciona la pestaña **JSON** en el cuadro de resultados para examinar la respuesta JSON completa.

<!-- Option coding example to run-->
## Limpieza de recursos

Si no usas los recursos de Azure creados en este laboratorio para otros módulos de entrenamiento, puedes eliminarlos para evitar incurrir en cargos adicionales.

1. Abre Azure Portal en `https://portal.azure.com` y, en la barra de búsqueda superior, busca los recursos que creaste en este laboratorio.

2. En la página del recurso, selecciona **Eliminar** y sigue las instrucciones para eliminar el recurso. Una alternativa es eliminar todo el grupo de recursos para limpiar todos los recursos al mismo tiempo.
