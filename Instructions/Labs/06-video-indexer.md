---
lab:
  title: Analizar vídeos
  description: Usar Video Indexer de Azure AI para analizar un vídeo.
---

# Analizar vídeos

Una gran proporción de los datos creados y consumidos hoy en día tiene el formato de vídeo. **Video Indexer de Azure AI** es un servicio con tecnología de IA que puede usar para indexar vídeos y extraer información de ellos.

> **Nota**: A partir del 21 de junio de 2022, las capacidades de los Servicios de Azure AI que devuelven información de identificación personal están restringidas a los clientes a los que se les ha concedido [acceso limitado](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Sin obtener una aprobación de acceso limitada, no está disponible el reconocimiento de personas y celebridades con Video Indexer para este laboratorio. Para obtener más detalles sobre los cambios realizados por Microsoft y la razón de estos, consulte el blog sobre [inversiones de inteligencia artificial y medidas de seguridad responsables para el reconocimiento facial](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## Carga de un vídeo en Video Indexer

En primer lugar, inicie sesión en el portal de Video Indexer y cargue un vídeo.

1. En el explorador, abra el [portal de Video Indexer](https://www.videoindexer.ai) en `https://www.videoindexer.ai`.
1. Si ya tiene una cuenta de Video Indexer, inicie sesión. De lo contrario, regístrese para obtener una cuenta gratuita e inicie sesión con su cuenta Microsoft (o cualquier otro tipo de cuenta válido). Si tiene dificultades para iniciar sesión, intente abrir una sesión privada del explorador.

    > Nota: Si esta es la primera vez que inicia sesión, es posible que vea un formulario emergente en el que se le pide que compruebe cómo va a usar el servicio. 

1. En una nueva pestaña, descargue el vídeo IA responsable visitando `https://aka.ms/responsible-ai-video`. Guarde el archivo.
1. En Video Indexer, seleccione la opción **Cargar**. A continuación, seleccione la opción **Buscar archivos**, seleccione el vídeo descargado y haga clic en **Agregar**. Cambie el texto del campo **Nombre de archivo** a **IA responsable**. Seleccione **Revisar y cargar**, revise la información general de resumen, active la casilla para comprobar el cumplimiento de las directivas de Microsoft sobre reconocimiento facial y cargue el archivo.
1. Una vez cargado el archivo, espere unos minutos mientras Video Indexer lo indexa automáticamente.

> **Nota**: En este ejercicio, vamos a usar este vídeo para explorar la funcionalidad de Video Indexer; pero debe verlo en su totalidad cuando haya terminado el ejercicio, ya que contiene información útil y una guía para desarrollar aplicaciones habilitadas para la inteligencia artificial de forma responsable. 

## Revisión de la información de vídeo

El proceso de indexación extrae información del vídeo, que puede ver en el portal.

1. En el portal de Video Indexer, cuando se indexe el vídeo, selecciónelo para verlo. Verá el reproductor de vídeo junto con un panel que muestra la información extraída del vídeo.

    > **Nota**: Debido a la directiva de acceso limitada para proteger las identidades de las personas, es posible que no vea nombres al indexar el vídeo.

    ![Video Indexer con un reproductor de vídeo y el panel Información](../media/video-indexer-insights.png)

1. A medida que se reproduce el vídeo, seleccione la pestaña **Línea de tiempo** para ver una transcripción del audio del vídeo.

    ![Video Indexer con un reproductor de vídeo y el panel Línea de tiempo que muestra la transcripción del vídeo.](../media/video-indexer-transcript.png)

1. En la parte superior derecha del portal, seleccione el símbolo **Ver** (que es similar a ) y, en la lista de información, además de **Transcripción**, seleccione **OCR** y **Hablantes**.

    ![Menú de vista de Video Indexer con Transcripción, OCR y Hablantes seleccionados](../media/video-indexer-view-menu.png)

1. Fíjese en que el panel **Línea de tiempo** ahora incluye:
    - Transcripción de la narración de audio.
    - Texto visible en el vídeo.
    - Indicaciones de los hablantes que aparecen en el vídeo. Algunas personas conocidas se reconocen automáticamente por el nombre y otras se indican por número (por ejemplo, *Hablante n.º 1*).
1. Vuelva al panel **Información** y vea la información que se muestra allí. Incluyen:
    - Personas concretas que aparecen en el vídeo.
    - Temas tratados en el vídeo.
    - Etiquetas para los objetos que aparecen en el vídeo.
    - Entidades con nombre, como personas y marcas que aparecen en el vídeo.
    - Escenas clave.
1. Con el panel **Información** visible, vuelva a seleccionar el símbolo **Ver** y, en la lista de información, agregue **Palabras clave** y **Opiniones** al panel.

    La información que se encuentre puede ayudarle a determinar los temas principales del vídeo. Por ejemplo, los **temas** de este vídeo muestran que trata claramente de tecnología, responsabilidad social y ética.

## Búsqueda de información

Puede usar Video Indexer para buscar información en el vídeo.

1. En el panel **Información**, en el cuadro **Buscar**, escriba *Abeja*. Es posible que tenga que desplazarse hacia abajo en el panel Información para ver los resultados de todos los tipos de información.
1. Observe que se encuentra una *etiqueta* coincidente, con su ubicación en el vídeo indicado debajo.
1. Seleccione el principio de la sección en la que se indica la presencia de una abeja y vea el vídeo en ese momento (es posible que tenga que pausarlo y seleccionarlo detenidamente; ¡la abeja solo aparece brevemente!).

    ![Resultados de búsqueda de Video Indexer para "abeja"](../media/video-indexer-search.png)

1. Desactive el cuadro **Buscar** para mostrar toda la información del vídeo.

## Uso de la API de REST de Video Indexer

Video Indexer proporciona una API de REST que puede usar para cargar y administrar vídeos en su cuenta.

1. En una nueva pestaña del explorador, abra [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicie sesión con sus credenciales de Azure. Mantenga abierta la pestaña existente con el portal de Video Indexer.
1. En Azure Portal, use el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, y seleccione un entorno de ***PowerShell*** sin almacenamiento en su suscripción.

    Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

    > **Nota**: Si el portal le pide que seleccione un almacenamiento para conservar los archivos, elija **No se requiere una cuenta de almacenamiento**, seleccione la suscripción que usa y presione **Aplicar**.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. Cambie el tamaño del panel de Cloud Shell para que pueda ver una mayor superficie.

    > **Sugerencia**: Puede cambiar el tamaño del panel arrastrando el borde superior. También puede usar los botones de minimizar y maximizar para cambiar entre Cloud Shell y la interfaz principal del portal.

1. En el panel de Cloud Shell, escribe los siguientes comandos para clonar el repositorio de GitHub que contiene los archivos de código de este ejercicio (escribe el comando o cópialo en el Portapapeles y, a continuación, haz clic con el botón derecho en la línea de comandos y pega como texto sin formato):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repositorio, vaya a la carpeta que contiene el archivo de código de la aplicación para este ejercicio:  

    ```
   cd mslearn-ai-vision/Labfiles/video-indexer
    ```

### Obtener los detalles de la API

Para usar la API de Video Indexer, necesita cierta información para autenticar las solicitudes:

1. En el portal de Video Indexer, expanda el panel de la izquierda y seleccione la página **Configuración de la cuenta**.
1. Anote el **identificador de cuenta** en esta página; lo necesitará más adelante.
1. Abra una nueva pestaña del explorador y vaya al [Portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai) en https://api-portal.videoindexer.ai e inicie sesión con sus credenciales de Azure.
1. En la página **Perfil**, vea las **Suscripciones** asociadas a su perfil.
1. En la página con sus suscripciones, observe que se le han asignado dos claves (una principal y una secundaria) para cada suscripción. A continuación, seleccione **Mostrar** en cualquiera de las claves para verla. Necesitará esta clave en breve.

### Uso de la API de REST

Ahora que tiene el identificador de cuenta y una clave de API, puede usar la API de REST para trabajar con vídeos en su cuenta. En este procedimiento, usará un script de PowerShell para realizar llamadas REST, pero se aplican los mismos principios con utilidades HTTP, como cURL o Postman, o cualquier lenguaje de programación capaz de enviar y recibir JSON a través de HTTP.

Todas las interacciones con la API de REST de Video Indexer siguen el mismo patrón:

- Se usa una solicitud inicial al método **AccessToken** con la clave de API en el encabezado para obtener un token de acceso.
- Las solicitudes posteriores usan el token de acceso para autenticarse al llamar a métodos REST para trabajar con vídeos.

1. En Cloud Shell, use el siguiente comando para abrir el script de PowerShell:

    ```
   code get-videos.ps1
    ```
    
1. En el script de PowerShell, reemplace los marcadores de posición **YOUR_ACCOUNT_ID** y **YOUR_API_KEY** por los valores de identificador de cuenta y clave de API que identificó anteriormente.
1. Observe que la *ubicación* de una cuenta gratuita es "trial". Si ha creado una cuenta de Video Indexer sin restricciones (con un recurso de Azure asociado), puede cambiarla a la ubicación donde se aprovisiona el recurso de Azure (por ejemplo, "eastus").
1. Revise el código del script, y tenga en cuenta que invoca dos métodos REST: uno para obtener un token de acceso y otro para enumerar los vídeos de su cuenta.
1. Guarde los cambios (presione *CTRL+S*), cierre el editor de código (presione *CTRL+Q*) y, a continuación, ejecute el siguiente comando para ejecutar el script:

    ```
   ./get-videos.ps1
    ```
    
1. Vea la respuesta JSON del servicio REST, que debe contener detalles del vídeo **IA responsable** que indexó anteriormente.

## Usar widgets de Video Indexer

El portal de Video Indexer es una interfaz útil para administrar proyectos de indexación de vídeo. Sin embargo, puede haber ocasiones en las que quiera que el vídeo y su información estén disponibles para las personas que no tienen acceso a su cuenta de Video Indexer. Video Indexer proporciona widgets que puede insertar en una página web para este propósito.

1. Use el comando `ls` para ver el contenido de la carpeta **video-indexer**. Observe que contiene un archivo **analyze-video.html**. Se trata de una página HTML básica en la que agregará los widgets de **Reproductor** e **Información** de Video Indexer.
1. Escriba el siguiente comando para editar el archivo:

    ```
   code analyze-video.html
    ```

    El archivo se abre en un editor de código.
   
1. Fíjese en la referencia al script **vb.widgets.mediator.js** en el encabezado: este script permite que varios widgets de Video Indexer de la página interactúen entre sí.
1. En el portal de Video Indexer, vuelva a la página **Archivos multimedia** y abra el vídeo **IA responsable**.
1. En el reproductor de vídeo, seleccione **&lt;/&gt; Insertar** para ver el código iframe HTML para insertar los widgets.
1. En el cuadro de diálogo **Compartir e insertar**, seleccione el widget **Reproductor**, establezca el tamaño del vídeo en 560 x 315 y, a continuación, copie el código para insertar en el portapapeles.
1. En Cloud Shell de Azure Portal, en el editor de código del archivo **analyze-video.html**, pegue el código copiado en el comentario **&lt;"El widget del Reproductor va aquí"&gt;**.
1. De nuevo en el portal de Video Indexer, en el cuadro de diálogo **Compartir e insertar**, seleccione el widget **Insights** y, a continuación, copie el código para insertar en el Portapapeles. A continuación, cierre el cuadro de diálogo **Compartir e insertar**, cambie a Azure Portal y pegue el código copiado en el comentario **&lt;"El widget de Insights va aquí"&gt;**.
1. Después de editar el archivo, en el editor de código, guarde los cambios (*CTRL+S*) y cierre el editor de código (*CTRL+Q*) mientras mantiene abierta la línea de comandos de Cloud Shell.
1. En la barra de herramientas de Cloud Shell, escriba el siguiente comando (específico de Cloud Shell) para descargar el archivo HTML que ha editado:

    ```
    download analyze-video.html
    ```

    El comando "download" crea un vínculo emergente en la parte inferior derecha del explorador, que puede seleccionar para descargar y abrir el archivo. Debería tener este aspecto:

    ![Widgets de Video Indexer en una página web](../media/video-indexer-widgets.png)

1. Experimente con los widgets, usando el widget **Información** para buscar información y saltar a ella en el vídeo.

