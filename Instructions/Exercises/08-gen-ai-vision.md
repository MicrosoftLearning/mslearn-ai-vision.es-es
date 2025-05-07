---
lab:
  title: Desarrollo de una aplicación de chat habilitada para visión
  description: Aprende a usar Fundición de IA de Azure para crear una aplicación de IA generativa que admita entradas de imagen.
---

# Desarrollo de una aplicación de chat habilitada para visión

En este ejercicio, usarás el modelo de IA generativa *Phi-4-multimodal-instruct* para generar respuestas a indicaciones que incluyen imágenes. Desarrollarás una aplicación que proporcione asistencia de IA con los productos frescos en un supermercado mediante Fundición de IA de Azure y el servicio de inferencia del modelo de Azure AI.

Este ejercicio dura aproximadamente **30** minutos.

## Creación de un proyecto de Fundición de IA de Azure

Comencemos creando un proyecto de Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure. Cierra las sugerencias o paneles de inicio rápido que se abran la primera vez que inicias sesión y, si es necesario, usa el logotipo de **Fundición de IA de Azure** en la parte superior izquierda para navegar a la página principal, que es similar a la siguiente imagen:

    ![Captura de pantalla del Portal de la Fundición de IA de Azure.](../media/ai-foundry-home.png)

2. En la página principal, selecciona **+Crear proyecto**.
3. En el asistente para **crear un proyecto**, escribe un nombre válido y si se te sugiere un centro existente, elige la opción para crear uno nuevo. A continuación, revisa los recursos de Azure que se crearán automáticamente para admitir el centro y el proyecto.
4. Selecciona **Personalizar** y especifica la siguiente configuración para el centro:
    - **Nombre del centro**: *un nombre válido para el centro*
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Ubicación**: selecciona cualquiera de las siguientes regiones\*:
        - Este de EE. UU.
        - Este de EE. UU. 2
        - Centro-Norte de EE. UU
        - Centro-sur de EE. UU.
        - Centro de Suecia
        - Oeste de EE. UU.
        - Oeste de EE. UU. 3
    - **Conectar Servicios de Azure AI o Azure OpenAI**: *crea un nuevo recurso de servicios de IA*
    - **Conectar Búsqueda de Azure AI**: omite la conexión

    > \* En el momento de escribir esto, el modelo de Microsoft *Phi-4-multimodal-instruct* que usaremos en este ejercicio estaba disponible en estas regiones. Puedes comprobar la disponibilidad regional más reciente de los modelos específicos en la [documentación de Fundición de IA de Azure](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability). En caso de que se alcance un límite de cuota regional más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región.

5. Selecciona **Siguiente** y revisa tu configuración. Luego, selecciona **Crear** y espera a que se complete el proceso.
6. Cuando se cree el proyecto, cierra las sugerencias que se muestran y revisa la página del proyecto en el Portal de la Fundición de IA de Azure, que debe tener un aspecto similar a la siguiente imagen:

    ![Captura de pantalla de los detalles de un proyecto de Azure AI en el Portal de la Fundición de IA de Azure.](../media/ai-foundry-project.png)

## Implementación de un modelo multimodal

Ahora estás listo para implementar un modelo multimodal que pueda admitir la entrada basada en imágenes. Hay varios modelos entre los que puedes elegir, incluido el modelo *gpt-4o* de OpenAI. En este ejercicio, usaremos un modelo *Phi-4-multimodal-instruct* que admita indicaciones que incluyan imágenes.

1. En la barra de herramientas de la parte superior derecha de la página del proyecto de Fundición de IA de Azure, usa el icono **Características de versión preliminar** (**&#9215;**) para asegurarte de que está habilitada la característica **Implementar modelos en el servicio de inferencia de modelos de Azure AI**. Esta característica garantiza que la implementación del modelo esté disponible para el servicio de inferencia de Azure AI, que usarás en el código de aplicación.
2. En el panel de la izquierda de tu proyecto, en la sección **Mis recursos**, selecciona la página **Modelos y puntos de conexión**.
3. En la página **Modelos y puntos de conexión**, en la pestaña **Implementaciones de modelos**, en el menú **+ Implementar modelo**, selecciona **Implementar modelo base**.
4. Busca el modelo **Phi-4-multimodal-instruct** en la lista y, a continuación, selecciónalo y confírmalo.
5. Acepta el contrato de licencia si se te solicita y, a continuación, implementa el modelo con la siguiente configuración seleccionando **Personalizar** en los detalles de implementación:
    - **Nombre de implementación**: *un nombre válido para la implementación de modelo*
    - **Tipo de implementación**: estándar global
    - **Detalles de implementación**: *usa la configuración predeterminada*
6. Espera a que se **complete** el estado de aprovisionamiento de la implementación.

## Prueba del modelo en el área de juegos

Ahora que tienes una implementación de modelo multimodal, puedes probarla con una indicación basada en imágenes en el área de juegos de chat.

1. En el panel de navegación de la izquierda, selecciona la página **Área de juegos** y abre el área de juegos **Chat**.
1. 1. En una nueva pestaña del explorador, descarga [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/mango.jpeg) desde `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/mango.jpeg` y guárdala en una carpeta del sistema de archivos local.
1. En la página de área de juegos de chat, en el panel **Configuración**, asegúrate de que la implementación de modelo **Phi-4-multimodal-instruct** esté seleccionada.
1. En el panel de sesión del chat principal, en el cuadro de entrada del chat, usa el botón adjuntar (**&#128206;**) para cargar el archivo de imagen *mango.jpeg* y, a continuación, agrega el texto `What desserts could I make with this fruit?` y envía la indicación.

    ![Captura de pantalla del área de juegos de chat con una indicación basada en imagen.](../media/chat-playground-image.png)

1. Revisa la respuesta, que debería proporcionar instrucciones relevantes para los postres que puedes hacer con un mango.

## Creación de una aplicación cliente

Ahora que has implementado el modelo, puedes usar la implementación en una aplicación cliente.

> **Sugerencia**: puedes elegir desarrollar la solución mediante Phyton o Microsoft #C *(próximamente)*. Sigue las instrucciones de la sección adecuada para el idioma elegido.

### Preparación de la configuración de aplicación

1. En el Portal de la Fundición de IA de Azure, mira la página **Información general** del proyecto.
2. En el área **Detalles del proyecto**, anota la **Cadena de conexión del proyecto**. Usarás esta cadena de conexión para conectarte al proyecto en una aplicación cliente.
3. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente). En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.

    Cierra las notificaciones de bienvenida para ver la página principal de Azure Portal.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** sin almacenamiento en tu suscripción.

    Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Puedes cambiar el tamaño o maximizar este panel para facilitar el trabajo.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

5. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de Cloud Shell, escribe los siguientes comandos para clonar el repositorio de GitHub que contiene los archivos de código de este ejercicio (escribe el comando o cópialo en el Portapapeles y haz clic con el botón derecho en la línea de comandos y pega como texto sin formato):


    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

7. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    **Python**

    ```
   cd mslearn-ai-vision/Labfiles/08-gen-ai-vision/python
    ```

    **C#**

    ```
   cd mslearn-ai-vision/Labfiles/08-gen-ai-vision/c-sharp
    ```

8. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
    ```

9. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    El archivo se abre en un editor de código.

10. En el archivo de código, reemplaza el marcador de posición **your_project_connection_string** por la cadena de conexión del proyecto (copiado de la página **Información general** del proyecto en el Portal de la Fundición de IA de Azure) y el marcador de posición **your_model_deployment** por el nombre que asignaste a la implementación de modelo Phi-4-multimodal-instruct.
11. Después de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o usa la acción de **hacer clic con el botón derecho > Guardar** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** o la acción de **hacer clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Escritura de código para conectarte al proyecto y obtener un cliente de chat para el modelo

> **Sugerencia**: al agregar código, asegúrate de mantener la sangría correcta.

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    **Python**

    ```
   code chat-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

2. En el archivo de código, observa las instrucciones existentes que se han agregado en la parte superior del archivo para importar los espacios de nombres de SDK necesarios. Después, busca el comentario **Add references** y agrega el código siguiente para hacer referencia a los espacios de nombres de las bibliotecas que has instalado anteriormente:

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
        ImageContentItem,
        ImageUrl,
   )
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.Inference;
    ```

3. En la función **principal**, en el comentario **Obtener ajustes de configuración**, observa que el código carga los valores de la cadena de conexión del proyecto y del nombre de implementación del modelo que has definido en el archivo de configuración.
4. Busca el comentario **Initialize the project client** y agrega el siguiente código para conectarte a tu proyecto de Fundición de IA de Azure mediante las credenciales de Azure con las que has iniciado sesión:

    **Python**

    ```python
   # Initialize the project client
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
   // Initialize the project client
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

5. Busca el comentario **Get a chat client** y agrega el siguiente código para crear un objeto de cliente para chatear con el modelo:

    **Python**

    ```python
   # Get a chat client
   chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```

### Escritura de código para enviar una indicación de imagen basada en direcciones URL

1. Ten en cuenta que el código incluye un bucle para permitir que un usuario escriba una indicación hasta que escriba "salir". A continuación, en la sección loop, busca el comentario **Get a response to image input** y agrega el código siguiente para enviar una indicación que incluya la siguiente imagen:

    ![Una foto de un mango.](../media/orange.jpeg)

    **Python**

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(content=[
                TextContentItem(text=prompt),
                ImageContentItem(image_url=ImageUrl(url=data_url))
            ]),
        ]
   )
   print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imageUrl = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/orange.jpeg";
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
        Messages = {
           new ChatRequestSystemMessage(system_message),
           new ChatRequestUserMessage([
                new ChatMessageTextContentItem(prompt),
                new ChatMessageImageContentItem(new Uri(imageUrl))
            ]),
        },
        Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código; no lo cierres todavía.

3. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para ejecutar la aplicación:

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. Cuando se te solicite, escribe la siguiente indicación:

    ```
   Suggest some recipes that include this fruit
    ```

5. Revisa la respuesta. A continuación, escribe `quit` para salir del programa.

### Modificación del código para cargar un archivo de imagen local

1. En el editor de código del código de la aplicación, en la sección loop, busca el código que agregaste anteriormente en el comentario **Get a response to image input**. A continuación, modifica el código de la siguiente manera para cargar este archivo de imagen local:

    ![Una foto de una fruta de dragón.](../media/mystery-fruit.jpeg)

    **Python**

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
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(content=[
                TextContentItem(text=prompt),
                ImageContentItem(image_url=ImageUrl(url=data_url))
            ]),
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imagePath = "mystery-fruit.jpeg";
   string mimeType = "image/jpeg";
    
   // Read and encode the image file
   byte[] imageBytes = File.ReadAllBytes(imagePath);
   var binaryImage = new BinaryData(imageBytes);
    
   // Include the image file data in the prompt
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
        Messages = {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage([
                new ChatMessageTextContentItem(prompt),
                new ChatMessageImageContentItem(bytes: binaryImage, mimeType: mimeType) 
            ]),
        },
        Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código. También puedes cerrar el editor de código (**CTRL+Q**) si lo deseas.

3. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para ejecutar la aplicación:

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. Cuando se te solicite, escribe la siguiente indicación:

    ```
   What is this fruit? What recipes could I use it in?
    ```

5. Revisa la respuesta. A continuación, escribe `quit` para salir del programa.

    > **Nota**: en esta aplicación sencilla, no hemos implementado una lógica para conservar el historial de conversaciones, por lo que el modelo tratará cada indicación como una nueva solicitud sin contexto de la indicación anterior.

## Explora más: (Si el tiempo lo permite)

Has aprendido a usar el SDK de inferencia de Azure AI y un modelo multimodal para implementar una aplicación de IA generativa que pueda responder a indicaciones basadas en imágenes. Si tienes tiempo, estas son algunas ideas para una exploración adicional.

### Uso de un modelo multimodal diferente

Has usado un modelo *Phi-4-multimodal-instruct* para generar una respuesta a una indicación basada en imágenes. Ahora vamos a probar un modelo *gpt-4o* de OpenAI.

1. En Fundición de IA de Azure, implementa un modelo **gpt-4o** en un punto de conexión de inferencia del modelo de Azure AI (es posible que tengas que crear un nuevo recurso en una región diferente).
1. Actualiza el archivo de configuración de código de la aplicación (*.env* para Python, *appsettings.json* para C#) para especificar el nombre del modelo gpt-4o.
1. Ejecuta la aplicación como antes, con las mismas indicaciones (puedes revertir al código que usa una imagen basada en direcciones URL si lo deseas).

### Uso de las API de OpenAI

El código que usaste en este ejercicio se basa en el SDK de inferencia de Azure AI, que funciona con cualquier modelo implementado en un punto de conexión de inferencia del modelo de Azure AI. Al usar un modelo de OpenAI, también puedes usar el SDK de OpenAI.

En las instrucciones siguientes se supone que has completado este ejercicio y la tarea adicional anterior para implementar y probar un modelo **gpt-4o**.

1. Instala (o actualiza) los paquetes necesarios para la aplicación:

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects openai
    ```
    
    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Azure.AI.OpenAI --prerelease
    ```

1. Actualiza los espacios de nombres en el archivo de código (quita las referencias de *azure.ai-inference*):

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   import openai
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using OpenAI.Chat;
   using Azure.AI.OpenAI;
    ```

1. Modifica el código para obtener un cliente de chat:

    **Python**

    ```python
   # Get a chat client
   chat_client = project_client.inference.get_azure_openai_client(api_version="2024-10-21")
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatClient chatClient = projectClient.GetAzureOpenAIChatClient(model_deployment);
    ```

1. Modificación del código para obtener una finalización basada en un archivo de imagen local

    **Python**

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
   response = chat_client.chat.completions.create(
        model=model_deployment,
        messages=[
            { "role": "system", "content": system_message },
            { "role": "user", "content": [  
                { 
                    "type": "text", 
                    "text": prompt 
                },
                { 
                    "type": "image_url",
                    "image_url": {
                        "url": data_url
                    }
                }
            ] } 
        ]
   )
   completion = response.choices[0].message.content
   print(completion)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imagePath = "mystery-fruit.jpeg";
   string mimeType = "image/jpeg";
    
   // Read and encode the image file
   byte[] imageBytes = File.ReadAllBytes(imagePath);
   var binaryImage = new BinaryData(imageBytes);

   // Include the image file data in the prompt
   List<ChatMessage> messages =
   [
        new SystemChatMessage(system_message),
        new UserChatMessage(
            ChatMessageContentPart.CreateTextPart(prompt),
            ChatMessageContentPart.CreateImagePart(binaryImage, mimeType)),
   ];

   ChatCompletion completion = chatClient.CompleteChat(messages);
   Console.WriteLine(completion.Content[0].Text);
    ```

1. Guarda los cambios y ejecuta la aplicación para probarla con las mismas indicaciones que usaste anteriormente.

## Limpieza

Si has terminado de explorar Fundición de IA de Azure, deberías eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
