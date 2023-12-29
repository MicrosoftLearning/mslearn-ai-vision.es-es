---
lab:
  title: Análisis de vídeo con Video Analyzer
  module: Module 8 - Getting Started with Azure AI Vision
---

# Análisis de vídeo con Video Analyzer

Una gran proporción de los datos creados y consumidos hoy en día tiene el formato de vídeo. **Video Indexer de Azure AI** es un servicio con tecnología de IA que puede usar para indexar vídeos y extraer información de ellos.

> **Nota**: A partir del 21 de junio de 2022, las capacidades de los Servicios de Azure AI que devuelven información de identificación personal están restringidas a los clientes a los que se les ha concedido [acceso limitado](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Sin obtener una aprobación de acceso limitada, no está disponible el reconocimiento de personas y celebridades con Video Analyzer para este laboratorio. Para obtener más detalles sobre los cambios realizados por Microsoft y la razón de estos, consulte el blog sobre [inversiones de inteligencia artificial y medidas de seguridad responsables para el reconocimiento facial](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## Clonación del repositorio para este curso

Si ha clonado recientemente el repositorio del código **mslearn-ai-vision** en el entorno en el que está trabajando en este laboratorio, ábralo en Visual Studio Code; en caso contrario, siga estos pasos para clonarlo ahora.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-vision` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## Cargar un vídeo a Video Analyzer

En primer lugar, deberá iniciar sesión en el portal de Video Analyzer y cargar un vídeo.

> **Sugerencia**: Si la página de Video Analyzer tarda en cargarse en el entorno de laboratorio hospedado, use el explorador instalado localmente. Puede volver a la máquina virtual hospedada para las tareas posteriores.

1. En el explorador, abra el portal de Video Analyzer en `https://www.videoindexer.ai`.
2. Si ya tiene una cuenta de Video Analyzer, inicie sesión. De lo contrario, regístrese para obtener una cuenta gratuita e inicie sesión con su cuenta Microsoft (o cualquier otro tipo de cuenta válido). Si tiene dificultades para iniciar sesión, intente abrir una sesión privada del explorador.
3. En Video Analyzer, seleccione la opción **Cargar**. A continuación, seleccione la opción para **escribir una URL**, escriba `https://aka.ms/responsible-ai-video` y haga clic en **Agregar**. Cambie el nombre predeterminado a **IA responsable**, revise la configuración predeterminada, active la casilla para comprobar el cumplimiento de las directivas de Microsoft para el reconocimiento facial y cargue el archivo.
4. Una vez cargado el archivo, espere unos minutos mientras Video Analyzer lo indexa automáticamente.

> **Nota**: En este ejercicio, vamos a usar este vídeo para explorar la funcionalidad de Video Analyzer; pero debe verlo en su totalidad cuando haya terminado el ejercicio, ya que contiene información útil y una guía para desarrollar aplicaciones habilitadas para la inteligencia artificial de forma responsable. 

## Revisión de la información de vídeo

El proceso de indexación extrae información del vídeo, que puede ver en el portal.

1. En el portal de Video Analyzer, cuando se indexe el vídeo, selecciónelo para verlo. Verá el reproductor de vídeo junto con un panel que muestra la información extraída del vídeo.

    > **Nota**: Debido a la directiva de acceso limitada para proteger las identidades de las personas, es posible que no vea nombres al indexar el vídeo.

![Video Analyzer con un reproductor de vídeo y el panel Información](../media/video-indexer-insights.png)

2. A medida que se reproduce el vídeo, seleccione la pestaña **Línea de tiempo** para ver una transcripción del audio del vídeo.

![Video Analyzer con un reproductor de vídeo y el panel Línea de tiempo que muestra la transcripción del vídeo.](../media/video-indexer-transcript.png)

3. En la parte superior derecha del portal, seleccione el símbolo **Ver** (que es similar a ) y, en la lista de información, además de **Transcripción**, seleccione **OCR** y **Hablantes**.

![Menú de vista de Video Analyzer con Transcripción, OCR y Hablantes seleccionados](../media/video-indexer-view-menu.png)

4. Fíjese en que el panel **Línea de tiempo** ahora incluye:
    - Transcripción de la narración de audio.
    - Texto visible en el vídeo.
    - Indicaciones de los hablantes que aparecen en el vídeo. Algunas personas conocidas se reconocen automáticamente por el nombre y otras se indican por número (por ejemplo, *Hablante n.º 1*).
5. Vuelva al panel **Información** y vea la información que se muestra allí. Incluyen:
    - Personas concretas que aparecen en el vídeo.
    - Temas tratados en el vídeo.
    - Etiquetas para los objetos que aparecen en el vídeo.
    - Entidades con nombre, como personas y marcas que aparecen en el vídeo.
    - Escenas clave.
6. Con el panel **Información** visible, vuelva a seleccionar el símbolo **Ver** y, en la lista de información, agregue **Palabras clave** y **Opiniones** al panel.

    La información que se encuentre puede ayudarle a determinar los temas principales del vídeo. Por ejemplo, los **temas** de este vídeo muestran que trata claramente de tecnología, responsabilidad social y ética.

## Búsqueda de información

Puede usar Video Analyzer para buscar información en el vídeo.

1. En el panel **Información**, en el cuadro **Buscar**, escriba *Abeja*. Es posible que tenga que desplazarse hacia abajo en el panel Información para ver los resultados de todos los tipos de información.
2. Observe que se encuentra una *etiqueta* coincidente, con su ubicación en el vídeo indicado debajo.
3. Seleccione el principio de la sección en la que se indica la presencia de una abeja y vea el vídeo en ese momento (es posible que tenga que pausarlo y seleccionarlo detenidamente; ¡la abeja solo aparece brevemente!).
4. Desactive el cuadro **Buscar** para mostrar toda la información del vídeo.

![Resultados de búsqueda de Video Analyzer para Abeja](../media/video-indexer-search.png)

## Uso de widgets de Video Analyzer

El portal de Video Analyzer es una interfaz útil para administrar proyectos de indexación de vídeo. Sin embargo, puede haber ocasiones en las que quiera que el vídeo y su información estén disponibles para las personas que no tienen acceso a su cuenta de Video Analyzer. Video Analyzer proporciona widgets que puede insertar en una página web para este propósito.

1. En Visual Studio Code, en la carpeta **06-video-indexer**, abra **analyze-video.html**. Se trata de una página HTML básica en la que agregará los widgets de **Reproductor** e **Información** de Video Analyzer. Fíjese en la referencia al script **vb.widgets.mediator.js** en el encabezado: este script permite que varios widgets de Video Analyzer de la página interactúen entre sí.
2. En el portal de Video Analyzer, vuelva a la página **Archivos multimedia** y abra el vídeo **IA responsable**.
3. En el reproductor de vídeo, seleccione **&lt;/&gt; Insertar** para ver el código iframe HTML para insertar los widgets.
4. En el cuadro de diálogo **Compartir e insertar**, seleccione el widget **Reproductor**, establezca el tamaño del vídeo en 560 x 315 y, a continuación, copie el código para insertar en el portapapeles.
5. En Visual Studio Code, en el archivo **analyze-video.html**, pegue el código copiado en el comentario **&lt;-- El widget Reproductor va aquí -- &gt;** .
6. En el cuadro de diálogo **Compartir e insertar**, seleccione el widget **Información** y, a continuación, copie el código para insertar en el portapapeles. A continuación, cierre el cuadro de diálogo **Compartir e insertar**, vuelva a Visual Studio Code y pegue el código copiado en el comentario **&lt;-- El widget Información va aquí -- &gt;** .
7. Guarde el archivo. A continuación, en el panel **Explorador**, haga clic con el botón derecho en **analyze-video.html** y seleccione **Mostrar en el Explorador de archivos**.
8. En el Explorador de archivos, abra **analyze-video.html** en el explorador para ver la página web.
9. Experimente con los widgets, usando el widget **Información** para buscar información y saltar a ella en el vídeo.

![Widgets de Video Analyzer en una página web](../media/video-indexer-widgets.png)

## Uso de la API de REST de Video Analyzer

Video Analyzer proporciona una API de REST que puede usar para cargar y administrar vídeos en su cuenta.

### Obtener los detalles de la API

Para usar la API de Video Analyzer, necesita cierta información para autenticar las solicitudes:

1. En el portal de Video Analyzer, expanda el panel izquierdo y seleccione la página **Configuración de la cuenta**.
2. Anote el **identificador de cuenta** en esta página; lo necesitará más adelante.
3. Abra una nueva pestaña del explorador y vaya al portal para desarrolladores de Video Analyzer en `https://api-portal.videoindexer.ai`, iniciando sesión con las credenciales de su cuenta de Video Analyzer.
4. En la página **Perfil**, vea las **Suscripciones** asociadas a su perfil.
5. En la página con sus suscripciones, observe que se le han asignado dos claves (una principal y una secundaria) para cada suscripción. A continuación, seleccione **Mostrar** en cualquiera de las claves para verla. Necesitará esta clave en breve.

### Uso de la API de REST

Ahora que tiene el identificador de cuenta y una clave de API, puede usar la API de REST para trabajar con vídeos en su cuenta. En este procedimiento, usará un script de PowerShell para realizar llamadas REST, pero se aplican los mismos principios con utilidades HTTP, como cURL o Postman, o cualquier lenguaje de programación capaz de enviar y recibir JSON a través de HTTP.

Todas las interacciones con la API de REST de Video Analyzer siguen el mismo patrón:

- Se usa una solicitud inicial al método **AccessToken** con la clave de API en el encabezado para obtener un token de acceso.
- Las solicitudes posteriores usan el token de acceso para autenticarse al llamar a métodos REST para trabajar con vídeos.

1. En Visual Studio Code, en la carpeta **06-video-indexer**, abra **get-videos.ps1**.
2. En el script de PowerShell, reemplace los marcadores de posición **YOUR_ACCOUNT_ID** y **YOUR_API_KEY** por los valores de identificador de cuenta y clave de API que identificó anteriormente.
3. Observe que la *ubicación* de una cuenta gratuita es "trial". Si ha creado una cuenta de Video Analyzer sin restricciones (con un recurso de Azure asociado), puede cambiarla a la ubicación donde se aprovisiona el recurso de Azure (por ejemplo, "eastus").
4. Revise el código del script, y tenga en cuenta que invoca dos métodos REST: uno para obtener un token de acceso y otro para enumerar los vídeos de su cuenta.
5. Guarde los cambios y, a continuación, en la parte superior derecha del panel de scripts, use el botón **▷** para ejecutar el script.
6. Vea la respuesta JSON del servicio REST, que debe contener detalles del vídeo **IA responsable** que indexó anteriormente.

## Más información

El reconocimiento de personas y celebridades sigue estando disponible pero, en función del [Estándar de inteligencia artificial responsable](https://aka.ms/aah91ff), está restringido por una directiva de acceso limitado. Estas características incluyen la identificación facial y el reconocimiento de celebridades. Para más información y solicitar acceso, consulte el [acceso limitado a los Servicios de Azure AI](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access).

Para obtener más información acerca de **Video Analyzer**, consulte la [documentación de Video Analyzer](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-for-media-docs/).
