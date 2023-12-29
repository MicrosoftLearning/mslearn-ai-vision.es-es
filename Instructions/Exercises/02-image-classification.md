---
lab:
  title: Clasificar imágenes con un modelo personalizado de Visión de Azure AI
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Clasificar imágenes con un modelo personalizado de Visión de Azure AI

Visión de Azure AI permite entrenar modelos personalizados para clasificar y detectar objetos con etiquetas que especifique. En este laboratorio, compilaremos un modelo de clasificación de imágenes personalizado para clasificar imágenes de fruta.

## Clonación del repositorio para este curso

Si aún no ha clonado el repositorio de código de **Visión de Azure AI** en el entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada en Visual Studio Code.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-vision` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**. Si aparece el mensaje *Se ha detectado un proyecto de función de Azure en la carpeta*, puede cerrarlo de forma segura.

## Aprovisionamiento de los recursos de Azure

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso de **Servicios de Azure AI**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. En la barra de búsqueda superior, busque *Servicios de Azure AI*, seleccione **Servicios de Azure AI** y cree un recurso de cuenta de varios servicios de Azure AI con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elija entre Este de EE. UU., Oeste de Europa, Oeste de EE. UU., Oeste de EE. UU. 2\**
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: estándar S0

    \*Las etiquetas de modelo personalizadas de Visión de Azure AI 4.0 solo están disponibles actualmente en estas regiones.

3. Active las casillas necesarias y cree el recurso.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

También necesitamos una cuenta de almacenamiento para almacenar las imágenes de entrenamiento.

1. De nuevo en Azure Portal, busque y seleccione **Cuentas de almacenamiento** y cree una nueva cuenta de almacenamiento con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija el mismo grupo de recursos en el que creó el recurso lógico de Servicios de Azure AI*
    - **Nombre de la cuenta de almacenamiento**: customclassifySUFFIX 
        - *nota: reemplace el token `SUFFIX`por sus iniciales u otro valor para asegurarse de que el nombre del recurso sea único globalmente.*
    - **Región**: *elija la misma región que usó para el recurso Servicios de Azure AI.*
    - **Rendimiento**: Estándar
    - **Redundancia**: almacenamiento con redundancia local (LRS)
1. Mientras se crea la cuenta de almacenamiento, vaya a Visual Studio Code y expanda la carpeta **02-image-classification**.
1. En esa carpeta, seleccione **replace.ps1** y revise el código. Verá que reemplaza el nombre de la cuenta de almacenamiento para el marcador de posición en un archivo JSON (el archivo COCO) que usamos en un paso posterior. Reemplace el marcador de posición de la primera línea por el nombre de la cuenta de almacenamiento. Guarde el archivo.
1. Haga clic con el botón derecho, abra un terminal integrado en **02-image-classification** y ejecute el siguiente comando.

    ```powershell
    ./replace.ps1
    ```

1. Puede revisar el archivo COCO para asegurarse de que el nombre de la cuenta de almacenamiento esté ahí. Seleccione **training-images/training_labels.json** y vea las primeras entradas. En el campo *absolute_url*, debería ver algo similar a `"https://myStorage.blob.core.windows.net/fruit/...`.
1. Cierre los archivos JSON y PowerShell y vuelva a la ventana del explorador.
1. La cuenta de almacenamiento debe completarse. Vaya a la cuenta de almacenamiento.
1. Habilitar el acceso público en la cuenta de almacenamiento. En el panel izquierdo, vaya a **Configuración** en el grupo **Configuración** y habilite *Permitir el acceso anónimo de blobs*. Seleccione **Guardar**.
1. En el panel izquierdo, seleccione **Contenedores** y cree un nuevo contenedor denominado `fruit` y establezca **Nivel de acceso anónimo** como *Contenedor (acceso de lectura anónimo para contenedores y blobs)*.

    > **Nota**: Si el **nivel de acceso anónimo** está deshabilitado, actualice la página del explorador.

1. Vaya a `fruit` y cargue las imágenes (y el archivo JSON) en **02-image-classification\training-images** en ese contenedor.

## Crear un proyecto de entrenamiento de modelo personalizado

A continuación, creará un nuevo proyecto de entrenamiento para la clasificación de imágenes personalizada en Vision Studio.

1. En el explorador web, vaya a `https://portal.vision.cognitive.azure.com/` e inicie sesión con la cuenta Microsoft donde creó el recurso de Azure AI.
1. Seleccione el icono **Personalizar modelos con imágenes** (puede encontrarse en la pestaña **Análisis de imágenes** si no se muestra en la vista predeterminada) y, si se le solicita, seleccione el recurso de Azure AI que creó.
1. En el proyecto, seleccione **Agregar nuevo conjunto de datos** en la parte superior. Configure con la siguiente configuración:
    - **Nombre del conjunto de datos**: training_images
    - **Tipo de modelo**: clasificación de imágenes
    - **Seleccione el contenedor de Azure Blob Storage**: seleccione **Seleccionar contenedor**
        - **Suscripción**: *suscripción de Azure*
        - *Cuenta de almacenamiento*: **la cuenta de almacenamiento que creó**
        - **Contenedor de blobs**: fruta
    - Seleccione la casilla para "Permitir que Vision Studio lea y escriba en el Blob Storage".
1. Seleccione el conjunto de datos **training_images**.

En este momento de la creación del proyecto, normalmente seleccionaría **Crear proyecto de etiquetado de datos de Azure ML** y etiquetaría las imágenes, lo que generaría un archivo COCO. Le recomendamos que pruebe esto si tiene tiempo, pero para los fines de este laboratorio ya hemos etiquetado las imágenes por usted y le proporcionamos el archivo COCO resultante.

1. Seleccione **Agregar archivo COCO**
1. En la lista desplegable, seleccione **Importar archivo COCO desde un contenedor de Blobs.**
1. Como ya ha conectado el contenedor denominado `fruit`, Vision Studio busca un archivo COCO. Seleccione **training_labels.json** en la lista desplegable y agregue el archivo COCO.
1. Vaya a **Modelos personalizados** a la izquierda y seleccione **Entrenar un nuevo modelo**. Use la configuración siguiente:
    - **Nombre del modelo**: classifyfruit
    - **Tipo de modelo**: clasificación de imágenes
    - **Elija el conjunto de datos de entrenamiento**: training_images
    - Deje el resto de manera predeterminada y seleccione **Entrenar modelo**

El entrenamiento puede tardar algún tiempo: el presupuesto predeterminado es de hasta una hora, pero para conjuntos de datos tan pequeños suele ser mucho más rápido. Seleccione el botón **Actualizar** cada dos minutos hasta que el estado del trabajo sea *Correcto*. Seleccione el modelo.

Aquí puede ver el rendimiento del trabajo de entrenamiento. Revise la precisión del modelo entrenado.

## Probar el modelo personalizado

El modelo se ha entrenado y está listo para probarlo.

1. En la parte superior de la página del modelo personalizado, seleccione **Probarlo**.
1. Seleccione el modelo **classifyfruit** de la lista desplegable que especifica el modelo que desea usar y examine la carpeta **02-image-classification\test-images**.
1. Seleccione cada imagen y vea los resultados. Seleccione la pestaña **JSON** en el cuadro de resultados para examinar la respuesta JSON completa.

<!-- Option coding example to run-->
## Limpieza de recursos

Si no usa los recursos de Azure creados en este laboratorio para otros módulos de entrenamiento, puede eliminarlos para evitar incurrir en cargos adicionales.

1. Abra Azure Portal en `https://portal.azure.com` y, en la barra de búsqueda superior, busque los recursos que creó en este laboratorio.

2. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso. Como alternativa, puede eliminar todo el grupo de recursos para limpiar todos los recursos al mismo tiempo.