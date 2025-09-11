---
lab:
  title: Desarrollo de una aplicación de chat habilitada para visión
  description: Use Fundición de IA de Azure para crear una aplicación de IA generativa que admita la entrada de imágenes.
---

# Desarrollo de una aplicación de chat habilitada para visión

En este ejercicio, usarás el modelo de IA generativa *Phi-4-multimodal-instruct* para generar respuestas a indicaciones que incluyen imágenes. Desarrollarás una aplicación que proporcione asistencia de IA con los productos frescos en un supermercado mediante Fundición de IA de Azure y el servicio de inferencia del modelo de Azure AI.

> **Nota**: Este ejercicio se basa en software de SDK de la versión preliminar, que podrían cambiar. Cuando ha sido necesario, hemos usado versiones específicas de paquetes; que pueden no ser las versiones disponibles más recientes. Puede que se produzcan algunos errores, comportamientos o advertencias inesperados.

Aunque este ejercicio se basa en el SDK de Python de Fundición de IA de Azure, puede desarrollar aplicaciones de chat de IA mediante varios SDK específicos del lenguaje; incluidos los siguientes:

- [Proyectos de Azure AI para Python](https://pypi.org/project/azure-ai-projects)
- [Proyectos de Azure AI para Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Projects)
- [Proyectos de Azure AI para JavaScript](https://www.npmjs.com/package/@azure/ai-projects)

Este ejercicio dura aproximadamente **30** minutos.

## Apertura del Portal de la Fundición de IA de Azure

Comencemos por iniciar sesión en el Portal de la Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure. Cierra las sugerencias o paneles de inicio rápido que se abran la primera vez que inicias sesión y, si es necesario, usa el logotipo de **Fundición de IA de Azure** en la parte superior izquierda para navegar a la página principal, que es similar a la siguiente imagen (cierra el panel **Ayuda** si está abierto):

    ![Captura de pantalla del Portal de la Fundición de IA de Azure.](./media/ai-foundry-home.png)

1. Revisa la información en la página principal.

## Selección de un modelo para iniciar un proyecto

Un *proyecto* de Azure AI proporciona un área de trabajo de colaboración para el desarrollo de IA. Vamos a empezar eligiendo un modelo con el que queremos trabajar y creando un proyecto para usarlo.

> **Nota**: los proyectos de Fundición de IA se pueden basar en un recurso de *Fundición de IA de Azure*, que proporciona acceso a modelos de IA (incluido Azure OpenAI), Servicios de Azure AI y otros recursos para desarrollar agentes de IA y soluciones de chat. Como alternativa, los proyectos se pueden basar en los recursos del *Centro de IA*, que incluyen conexiones a los recursos de Azure para proteger el almacenamiento, el proceso y las herramientas especializadas. Los proyectos basados en Fundición de IA de Azure son ideales para los desarrolladores que desean administrar recursos para el desarrollo de aplicaciones de chat o agentes de IA. Los proyectos basados en el Centro de IA son más adecuados para los equipos de desarrollo empresarial que trabajan en soluciones de IA complejas.

1. En la página principal, en la sección **Explorar modelos y funcionalidades**, busca el modelo `Phi-4-multimodal-instruct`, que usaremos en nuestro proyecto.

1. En los resultados de la búsqueda, seleccione el modelo **Phi-4-multimodal-instruct** para ver sus detalles y, a continuación, en la parte superior de la página del modelo, seleccione **Usar este modelo**.

1. Cuando se te pida que crees un proyecto, escribe un nombre válido para el proyecto y expande **Opciones avanzadas**.

1. Selecciona **Personalizar** y especifica la siguiente configuración para el centro:
    - **Recurso de Fundición de IA de Azure**: *un nombre válido para el recurso de Fundición de IA de Azure*
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Región**: *seleccione cualquiera (se recomienda Fundición de IA\*).

    > \* Algunos de los recursos de Azure AI están restringidos por cuotas de modelo regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región.

1. Seleccione **Crear** y espere a que se cree el proyecto, incluida la implementación del modelo Phi-4-multimodal-instruct que ha seleccionado.

    > Nota: Dependiendo de la selección del modelo, podría recibir mensajes adicionales durante el proceso de creación del proyecto. Acepte los términos y finalice la implementación.

1. Cuando se cree el proyecto, el modelo se mostrará en la página **Modelos y puntos de conexión**:

    ![Recorte de pantalla de la página de implementación del modelo.](./media/ai-foundry-model-deployment.png)

## Prueba del modelo en el área de juegos

Ahora puede probar la implementación de modelos multimodales con una indicación basada en imágenes en el área de juegos de chat.

1. Seleccione **Abrir en el área de juegos** en la página de implementación del modelo.

1. En una nueva pestaña del explorador, descarga [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg) desde `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg` y guárdala en una carpeta del sistema de archivos local.

1. En la página de área de juegos de chat, en el panel **Configuración**, asegúrate de que la implementación de modelo **Phi-4-multimodal-instruct** esté seleccionada.

1. En el panel de sesión del chat principal, en el cuadro de entrada del chat, usa el botón adjuntar (**&#128206;**) para cargar el archivo de imagen *mango.jpeg* y, a continuación, agrega el texto `What desserts could I make with this fruit?` y envía la indicación.

    ![Recorte de pantalla de la página del área de juegos de chat.](../media/chat-playground-image.png)

1. Revisa la respuesta, que debería proporcionar instrucciones relevantes para los postres que puedes hacer con un mango.

## Creación de una aplicación cliente

Ahora que has implementado el modelo, puedes usar la implementación en una aplicación cliente.

### Preparación de la configuración de aplicación

1. En el Portal de la Fundición de IA de Azure, mira la página **Información general** del proyecto.

1. En el área **Puntos de conexión y claves**, asegúrese de que está seleccionada la biblioteca de **Fundición de IA de Azure** y anote el **punto de conexión del proyecto de Fundición de IA de Azure**. Usarás esta cadena de conexión para conectarte al proyecto en una aplicación cliente.

1. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente). En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.

    Cierra las notificaciones de bienvenida para ver la página principal de Azure Portal.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** sin almacenamiento en tu suscripción.

    Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Puedes cambiar el tamaño o maximizar este panel para facilitar el trabajo.

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
   cd mslearn-ai-vision/Labfiles/gen-ai-vision/python
    ```

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplace el marcador de posición **your_project_endpoint** por el punto de conexión del proyecto de Fundición (copiado de la página **Información general** del proyecto en el portal de Fundición de IA de Azure) y el marcador de posición **your_model_deployment** por el nombre que asignó a la implementación del modelo Phi-4-multimodal-instruct.

1. Después de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o usa la acción de **hacer clic con el botón derecho > Guardar** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** o la acción de **hacer clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Escritura de código para conectarte al proyecto y obtener un cliente de chat para el modelo

> **Sugerencia**: al agregar código, asegúrate de mantener la sangría correcta.

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    ```
   code chat-app.py
    ```

1. En el archivo de código, observa las instrucciones existentes que se han agregado en la parte superior del archivo para importar los espacios de nombres de SDK necesarios. Después, busca el comentario **Add references** y agrega el código siguiente para hacer referencia a los espacios de nombres de las bibliotecas que has instalado anteriormente:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from openai import AzureOpenAI
    ```

1. En la función **principal**, en el comentario **Obtener ajustes de configuración**, observa que el código carga los valores de la cadena de conexión del proyecto y del nombre de implementación del modelo que has definido en el archivo de configuración.
1. En la función **principal**, en el comentario **Obtener ajustes de configuración**, observa que el código carga los valores de la cadena de conexión del proyecto y del nombre de implementación del modelo que has definido en el archivo de configuración.
1. Busque el comentario **Inicializar el cliente del proyecto** y agregue el siguiente código para conectarse al proyecto de Fundición de IA de Azure:

    > **Sugerencia**: Ten cuidado de mantener el nivel de sangría correcto del código.

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True
            ),
            endpoint=project_endpoint,
        )
    ```

1. Busca el comentario **Get a chat client** y agrega el siguiente código para crear un objeto de cliente para chatear con un modelo:

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

### Escritura de código para enviar una indicación de imagen basada en direcciones URL

1. Ten en cuenta que el código incluye un bucle para permitir que un usuario escriba una indicación hasta que escriba "salir". A continuación, en la sección loop, busca el comentario **Get a response to image input** y agrega el código siguiente para enviar una indicación que incluya la siguiente imagen:

    ![Una foto de un mango.](../media/orange.jpeg)

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = openai_client.chat.completions.create(
        model=model_deployment,
        messages=[
            {"role": "system", "content": system_message},
            { "role": "user", "content": [  
                { "type": "text", "text": prompt},
                { "type": "image_url", "image_url": {"url": data_url}}
            ] } 
        ]
   )
   print(response.choices[0].message.content)
    ```

1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código; no lo cierres todavía.

## Inicie sesión en Azure y ejecuta la aplicación.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para iniciar sesión en Azure.

    ```
   az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulta [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obtener más información.
    
1. Cuando se te solicite, sigue las instrucciones para abrir la página de inicio de sesión en una nueva pestaña y escribe el código de autenticación proporcionado y las credenciales de Azure. A continuación, completa el proceso de inicio de sesión en la línea de comandos y selecciona la suscripción que contiene el centro de Fundición de IA de Azure si se te solicita.

1. Después de iniciar sesión, escribe el siguiente comando para ejecutar la aplicación:

    ```
   python chat-app.py
    ```

1. Cuando se te solicite, escribe la siguiente indicación:

    ```
   Suggest some recipes that include this fruit
    ```

1. Revisa la respuesta. A continuación, escribe `quit` para salir del programa.

### Modificación del código para cargar un archivo de imagen local

1. En el editor de código del código de la aplicación, en la sección loop, busca el código que agregaste anteriormente en el comentario **Get a response to image input**. A continuación, modifica el código de la siguiente manera para cargar este archivo de imagen local:

    ![Una foto de una fruta de dragón.](../media/mystery-fruit.jpeg)

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

   # Read and encode the image file
   with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

   # Include the image file data in the prompt
   data_url = f"data:{mime_type};base64,{base64_encoded_data}"
   response = openai_client.chat.completions.create(
            model=model_deployment,
            messages=[
                {"role": "system", "content": system_message},
                { "role": "user", "content": [  
                    { "type": "text", "text": prompt},
                    { "type": "image_url", "image_url": {"url": data_url}}
                ] } 
            ]
   )
   print(response.choices[0].message.content)
    ```

1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código. También puedes cerrar el editor de código (**CTRL+Q**) si lo deseas.

1. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para ejecutar la aplicación:

    ```
   python chat-app.py
    ```

1. Cuando se te solicite, escribe la siguiente indicación:

    ```
   What is this fruit? What recipes could I use it in?
    ```

15. Revisa la respuesta. A continuación, escribe `quit` para salir del programa.

    > **Nota**: en esta aplicación sencilla, no hemos implementado una lógica para conservar el historial de conversaciones, por lo que el modelo tratará cada indicación como una nueva solicitud sin contexto de la indicación anterior.

## Limpieza

Si has terminado de explorar el Portal de la Fundición de IA de Azure, deberías eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Abre [Azure Portal](https://portal.azure.com) y ve el contenido del grupo de recursos donde implementaste los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
