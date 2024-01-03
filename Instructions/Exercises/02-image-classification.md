---
lab:
  title: Clasificación de imágenes con el modelo personalizado de Visión de Azure AI
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Clasificación de imágenes con el modelo personalizado de Visión de Azure AI

Visión de Azure AI te permite entrenar modelos personalizados para clasificar y detectar objetos con las etiquetas que especifiques. En este laboratorio, compilaremos un modelo de clasificación de imágenes personalizado para clasificar imágenes de fruta.

## Clonación del repositorio para este curso

Si aún no has clonado el repositorio de código de **Visión de Azure AI** en el entorno en el que estás trabajando en este laboratorio, sigue estos pasos para hacerlo. De lo contrario, abra la carpeta clonada en Visual Studio Code.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-vision` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**. Si aparece el mensaje *Se ha detectado un proyecto de Azure Functions en la carpeta*, puedes cerrar ese mensaje de forma segura.

## Aprovisionamiento de los recursos de Azure

Si aún no tienes uno en tu suscripción, deberás aprovisionar un recurso de **Servicios de Azure AI**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. En la barra de búsqueda superior, busca *Servicios de Azure AI*, selecciona **Servicios de Azure AI** y crea un recurso de cuenta multiservicio de servicios de Azure AI con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elige entre Este de EE. UU., Oeste de Europa, Oeste de EE. UU. 2\**
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: estándar S0

    \*Por el momento, las etiquetas de modelo personalizadas de Visión de Azure AI solo están disponibles en estas regiones.

3. Active las casillas necesarias y cree el recurso.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

También necesitamos una cuenta de almacenamiento para almacenar las imágenes de entrenamiento.

1. En Azure Portal, busca y selecciona **Cuentas de almacenamiento** y crea una cuenta de almacenamiento con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elige el mismo grupo de recursos en el que creaste el recurso de Servicios de Azure AI*
    - **Nombre de cuenta de almacenamiento**: customclassifySUFFIX 
        - *nota: Reemplaza el token `SUFFIX` por tus iniciales u otro valor para asegurarte de que el nombre del recurso sea globalmente único.*
    - **Región**: *elige la misma región que usaste para el recurso de Servicios de Azure AI*
    - **Rendimiento**: Estándar
    - **Redundancia**: almacenamiento con redundancia local (LRS)
1. Mientras se crea la cuenta de almacenamiento, ve a Visual Studio Code y expande la carpeta **Labfiles/02-image-classification**.
1. En esa carpeta, selecciona **replace.ps1** y revisa el código. Verás que reemplaza el nombre de la cuenta de almacenamiento para el marcador de posición en un archivo JSON (el archivo COCO) que usamos en un paso posterior. Reemplaza el marcador *solamente en la primera línea* del archivo por el nombre de la cuenta de almacenamiento. Guarde el archivo.
1. Haz clic con el botón derecho en la carpeta **02-image-classification** y abre un terminal integrado. Ejecute el siguiente comando:

    ```powershell
    ./replace.ps1
    ```

1. Puedes revisar el archivo COCO para asegurarte de que el nombre de la cuenta de almacenamiento esté ahí. Selecciona **training-images/training_labels.json** y examina las primeras entradas. En el campo *absolute_url*, deberías ver algo similar a *"https://myStorage.blob.core.windows.net/fruit/...*. Si no ves el cambio previsto, asegúrate de actualizar solo el primer marcador de posición en el script de PowerShell.
1. Cierra el archivo JSON y PowerShell, y vuelve a la ventana del explorador.
1. La cuenta de almacenamiento debería estar completa. Vaya a la cuenta de almacenamiento.
1. Habilita el acceso público en la cuenta de almacenamiento. En el panel izquierdo, navega hasta **Configuración** en el grupo **Configuración** y habilita *Permitir acceso anónimo de blobs*. Seleccione **Guardar**.
1. En el panel izquierdo, selecciona **Contenedores**, crea un nuevo contenedor denominado `fruit` y establece **Nivel de acceso anónimo** en *Contenedor (acceso de lectura anónimo para contenedores y blobs).*

    > **Nota**: Si la opción **Nivel de acceso anónimo** está deshabilitada, actualiza la página del explorador.

1. Navega hasta `fruit` y carga las imágenes (y el archivo JSON) en **Labfiles/02-image-classification/training-images** en ese contenedor.

## Creación de un proyecto de entrenamiento de modelo personalizado

Ahora, crearás un nuevo proyecto de entrenamiento para la clasificación de imágenes personalizada en Vision Studio.

1. En el explorador web, navega hasta `https://portal.vision.cognitive.azure.com/` e inicia sesión con la cuenta Microsoft en la que creaste el recurso de Azure AI.
1. Selecciona el icono **Personalizar modelos con imágenes** (lo puedes encontrar en la pestaña **Análisis de imágenes** si no se muestra en la vista predeterminada) y, si te lo solicitan, selecciona el recurso de Azure AI que creaste.
1. En el proyecto, selecciona **Agregar nuevo conjunto de datos** en la parte superior. Configure  con la siguiente configuración:
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