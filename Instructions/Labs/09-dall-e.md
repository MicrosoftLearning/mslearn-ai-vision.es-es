---
lab:
  title: Generación de imágenes con IA
  description: Use un modelo DALL-E de OpenAI en Fundición de IA de Azure para generar imágenes.
---

# Generación de imágenes con IA

En este ejercicio, usarás el modelo de IA generativa DALL-E de OpenAI para generar imágenes. También usará el SDK de Python de OpenAI para crear una aplicación sencilla para generar imágenes basadas en sus indicaciones.

> **Nota**: Este ejercicio se basa en software de SDK de la versión preliminar, que podrían cambiar. Cuando ha sido necesario, hemos usado versiones específicas de paquetes; que pueden no ser las versiones disponibles más recientes. Puede que se produzcan algunos errores, comportamientos o advertencias inesperados.

Aunque este ejercicio se basa en el SDK de Python de OpenAI, puede desarrollar aplicaciones similares usando varios SDK específicos del lenguaje, como:

* [Proyectos de OpenAI para Microsoft .NET](https://www.nuget.org/packages/OpenAI)
* [Proyectos de OpenAI para JavaScript](https://www.npmjs.com/package/openai)

Este ejercicio dura aproximadamente **30** minutos.

## Apertura del Portal de la Fundición de IA de Azure

Comencemos por iniciar sesión en el Portal de la Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure. Cierra las sugerencias o paneles de inicio rápido que se abran la primera vez que inicias sesión y, si es necesario, usa el logotipo de **Fundición de IA de Azure** en la parte superior izquierda para navegar a la página principal, que es similar a la siguiente imagen (cierra el panel **Ayuda** si está abierto):

    ![Captura de pantalla del Portal de la Fundición de IA de Azure.](./media/ai-foundry-home.png)

1. Revisa la información en la página principal.

## Selección de un modelo para iniciar un proyecto

Un *proyecto* de Azure AI proporciona un área de trabajo de colaboración para el desarrollo de IA. Vamos a empezar eligiendo un modelo con el que queremos trabajar y creando un proyecto para usarlo.

> **Nota**: los proyectos de Fundición de IA se pueden basar en un recurso de *Fundición de IA de Azure*, que proporciona acceso a modelos de IA (incluido Azure OpenAI), Servicios de Azure AI y otros recursos para desarrollar agentes de IA y soluciones de chat. Como alternativa, los proyectos se pueden basar en los recursos del *Centro de IA*, que incluyen conexiones a los recursos de Azure para proteger el almacenamiento, el proceso y las herramientas especializadas. Los proyectos basados en Fundición de IA de Azure son ideales para los desarrolladores que desean administrar recursos para el desarrollo de aplicaciones de chat o agentes de IA. Los proyectos basados en el Centro de IA son más adecuados para los equipos de desarrollo empresarial que trabajan en soluciones de IA complejas.

1. En la página principal, en la sección **Explorar modelos y funcionalidades**, busca el modelo `dall-e-3`, que usaremos en nuestro proyecto.

1. En los resultados de la búsqueda, seleccione el modelo **dall-e-3** para ver sus detalles y, a continuación, en la parte superior de la página del modelo, seleccione **Usar este modelo**.

1. Cuando se te pida que crees un proyecto, escribe un nombre válido para el proyecto y expande **Opciones avanzadas**.

1. Selecciona **Personalizar** y especifica la siguiente configuración para el centro:
    - **Recurso de Fundición de IA de Azure**: *un nombre válido para el recurso de Fundición de IA de Azure*
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Región**: *seleccione cualquiera (se recomienda Fundición de IA\*).

    > \* Algunos de los recursos de Azure AI están restringidos por cuotas de modelo regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región.

1. Seleccione **Crear** y espere a que se cree el proyecto, junto con la implementación del modelo dall-e-3 que ha seleccionado.

    > Nota: Dependiendo de la selección del modelo, podría recibir mensajes adicionales durante el proceso de creación del proyecto. Acepte los términos y finalice la implementación.

1. Cuando se cree el proyecto, el modelo se mostrará en la página **Modelos y puntos de conexión**.

## Prueba del modelo en el área de juegos

Antes de crear una aplicación cliente, vamos a probar el modelo DALL-E en el área de juegos.

1. Seleccione **Áreas de juegos** y, a continuación, **Área de juegos de imágenes**.

1. Asegúrate de que la implementación de modelo DALL-E está seleccionada. A continuación, en el cuadro situado cerca de la parte inferior de la página, escriba una indicación, como `Create an image of an robot eating spaghetti` y seleccione **Generar**.

1. Revisa la imagen resultante en el área de juegos:

    ![Captura de pantalla del área de juegos de imágenes con una imagen generada.](../media/images-playground.png)

1. Escribe una solicitud de seguimiento, como `Show the robot in a restaurant` y revisa la imagen resultante.

1. Sigue probando con nuevas solicitudes para refinar la imagen hasta que estés satisfecho con ella. 

1. Seleccione el botón **\</\>Ver código** y asegúrese de que está en la pestaña **Autenticación de Entra ID**. A continuación, registre la siguiente información para usarla más adelante en el ejercicio. Tenga en cuenta que los valores son ejemplos; asegúrese de registrar la información de su implementación.

    * Punto de conexión de OpenAI: *https://dall-e-aus-resource.cognitiveservices.azure.com/*
    * Versión de API de OpenAI: *2024-04-01-preview*
    * Nombre de implementación (nombre del modelo): *dall-e-3*

## Creación de una aplicación cliente

El modelo parece funcionar en el área de juegos. Ahora puede usar el SDK de OpenAI en una aplicación cliente.

### Preparación de la configuración de aplicación

1. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente). En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

    > **Nota**: Si el portal le pide que seleccione un almacenamiento para conservar los archivos, elija **No se requiere una cuenta de almacenamiento**, seleccione la suscripción que usa y presione **Aplicar**.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de Cloud Shell, escribe los siguientes comandos para clonar el repositorio de GitHub que contiene los archivos de código de este ejercicio (escribe el comando o cópialo en el Portapapeles y, a continuación, haz clic con el botón derecho en la línea de comandos y pega como texto sin formato):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    ```
   cd mslearn-ai-vision/Labfiles/dalle-client/python
    ```

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity openai requests
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. Reemplace los marcadores de posición **your_endpoint**, **your_model_deployment** y **your_api_version** por los valores que registró desde el **área de juegos de imágenes**.

1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Escritura del código para conectarte al proyecto y chatear con el modelo

> **Sugerencia**: al agregar código, asegúrate de mantener la sangría correcta.

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    ```
   code dalle-client.py
    ```

1. En el archivo de código, observa las instrucciones existentes que se han agregado en la parte superior del archivo para importar los espacios de nombres de SDK necesarios. Después, en el comentario **Agregar referencias**, agrega el código siguiente para hacer referencia a los espacios de nombres de las bibliotecas que has instalado anteriormente:

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
   from openai import AzureOpenAI
   import requests
    ```

1. En la función **main**, en el comentario **Obtener opciones de configuración**, tenga en cuenta que el código carga los valores de punto de conexión, versión de API y nombre de implementación del modelo que definió en el archivo de configuración.

1. En el comentario **Inicializar el cliente**, agregue el código siguiente para conectarse al modelo mediante las credenciales de Azure con las que ha iniciado sesión actualmente:

    ```python
   # Initialize the client
   token_provider = get_bearer_token_provider(
       DefaultAzureCredential(exclude_environment_credential=True,
           exclude_managed_identity_credential=True), 
       "https://cognitiveservices.azure.com/.default"
   )
    
   client = AzureOpenAI(
       api_version=api_version,
       azure_endpoint=endpoint,
       azure_ad_token_provider=token_provider
   )
    ```

1. Ten en cuenta que el código incluye un bucle para permitir que un usuario escriba una indicación hasta que escriba "salir". A continuación, en la sección bucle, en el comentario **Generar una imagen**, agrega el código siguiente para enviar la indicación y recuperar la dirección URL para la imagen generada del modelo:

    **Python**

    ```python
   # Generate an image
   result = client.images.generate(
        model=model_deployment,
        prompt=input_text,
        n=1
    )

   json_response = json.loads(result.model_dump_json())
   image_url = json_response["data"][0]["url"] 
    ```

1. Ten en cuenta que el código del resto de la función **principal** pasa la dirección URL de la imagen y un nombre de archivo a una función proporcionada, que descarga la imagen generada y la guarda como un archivo .png.

1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Ejecución de la aplicación cliente

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para iniciar sesión en Azure.

    ```
   az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulta [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obtener más información.

1. Cuando se te solicite, sigue las instrucciones para abrir la página de inicio de sesión en una nueva pestaña y escribe el código de autenticación proporcionado y las credenciales de Azure. A continuación, completa el proceso de inicio de sesión en la línea de comandos y selecciona la suscripción que contiene el centro de Fundición de IA de Azure si se te solicita.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para ejecutar la aplicación:

    ```
   python dalle-client.py
    ```

1. Cuando se te solicite, escribe una solicitud para una imagen, como `Create an image of a robot eating pizza`. Tras unos instantes, la aplicación debe confirmar que la imagen se ha guardado.

1. Prueba algunas indicaciones más. Cuando hayas terminado, presiona `quit` para salir del programa.

    > **Nota**: en esta aplicación sencilla, no hemos implementado una lógica para conservar el historial de conversaciones, por lo que el modelo tratará cada indicación como una nueva solicitud sin contexto de la indicación anterior.

1. Para descargar y ver las imágenes generadas por la aplicación, usa el comando **download** de Cloud Shell, especificando el archivo .png que se generó:

    ```
   download ./images/image_1.png
    ```

    El comando download crea un vínculo emergente en la parte inferior derecha del explorador, que puedes seleccionar para descargar y abrir el archivo.

## Resumen

En este ejercicio, has usado Fundición de IA de Azure y el SDK de Azure OpenAI para crear una aplicación cliente que usa un modelo DALL-E para generar imágenes.

## Limpieza

Si has terminado de explorar DALL-E, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
