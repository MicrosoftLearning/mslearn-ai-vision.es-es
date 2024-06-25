---
lab:
  title: "Análisis de imágenes con Visión de Azure\_AI"
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Análisis de imágenes con Visión de Azure AI

Visión de Azure AI es una capacidad de inteligencia artificial que permite a los sistemas de software interpretar la entrada visual mediante el análisis de imágenes. En Microsoft Azure, el servicio **Visión** de Azure AI proporciona modelos predefinidos para tareas comunes de Computer Vision, incluido el análisis de imágenes para sugerir subtítulos y etiquetas, la detección de objetos comunes y de personas. También puede usar el servicio Visión de Azure AI para quitar el fondo o crear un mate en primer plano de las imágenes.

## Clonación del repositorio para este curso

Si aún no has clonado el repositorio de código de **Visión de Azure AI** en el entorno en el que estás trabajando en este laboratorio, sigue estos pasos para hacerlo. De lo contrario, abra la carpeta clonada en Visual Studio Code.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-vision` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**. Si aparece el mensaje *Se ha detectado un proyecto de función de Azure en la carpeta*, puede cerrarlo de forma segura.

## Aprovisionamiento de un recurso de Servicios de Azure AI

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso de **Servicios de Azure AI**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. En la barra de búsqueda superior, busque *Servicios de Azure AI*, seleccione **Servicios de Azure AI** y cree un recurso de cuenta de varios servicios de Azure AI con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elija entre Este de EE. UU., Centro de Francia, Centro de Corea, Norte de Europa, Sudeste asiático, Oeste de Europa, Oeste de EE. UU. o Este de Asia\**
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: estándar S0

    \*Las características de Visión de Azure AI 4.0 solo están disponibles actualmente en estas regiones.

3. Active las casillas necesarias y cree el recurso.
4. Espere a que se complete la implementación y, a continuación, consulte los detalles.
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Claves y punto de conexión**. Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## Prepararse para usar el SDK de Visión de Azure AI

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa el SDK de Visión de Azure AI para analizar imágenes.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **Labfiles/01-analyze-images** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **image-analysis** y abra un terminal integrado. A continuación, instale el paquete del SDK de Visión de Azure AI mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **Nota**: Si se le pide que instale extensiones del kit de desarrollo, puede cerrar el mensaje de forma segura.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```
    
3. Consulte el contenido de la carpeta **image-analysis** y observe que contiene un archivo para las opciones de configuración:
    - **C#**: appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Servicios de Azure AI. Guarde los cambios.
4. Observe que la carpeta **image-analysis** contiene un archivo de código para la aplicación cliente:

    - **C#**: Program.cs
    - **Python**: image-analysis.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Import namespaces (Importar espacios de nombres)**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Visión de Azure AI:

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```
    
## Visualización de las imágenes que va a analizar

En este ejercicio, usará el servicio Visión de Azure AI para analizar varias imágenes.

1. En Visual Studio Code, expanda la carpeta **image-analysis** y la carpeta **images** que contiene.
2. Seleccione los archivos de imagen de uno en uno para verlos en Visual Studio Code.

## Análisis de una imagen para sugerir un título

Ahora ya puede usar el SDK para llamar al servicio de Visión y analizar una imagen.

1. En el archivo de código de la aplicación cliente (**Program.cs** o **image-analysis.py**), en la función **Main**, observe que se ha proporcionado el código para cargar los valores de configuración. A continuación, busque el comentario **Authenticate Azure AI Vision client (Autenticar cliente de Computer Vision)**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para crear y autenticar un objeto de cliente de Visión de Azure AI:

**C#**

```C#
// Authenticate Azure AI Vision client
ImageAnalysisClient client = new ImageAnalysisClient(
    new Uri(aiSvcEndpoint),
    new AzureKeyCredential(aiSvcKey));
```

**Python**

```Python
# Authenticate Azure AI Vision client
cv_client = ImageAnalysisClient(
    endpoint=ai_endpoint,
    credential=AzureKeyCredential(ai_key)
)
```

2. En la función **Main (Principal)**, en el código que acaba de agregar, observe que el código especifica la ruta de acceso a un archivo de imagen y, a continuación, pasa la ruta de acceso de la imagen a otras dos funciones (**AnalyzeImage** y **BackgroundForeground**). Estas funciones aún no están totalmente implementadas.

3. En la función **AnalyzeImage**, en el comentario **Obtener resultado con especificar las características que se van a recuperar**, agregue el siguiente código:

**C#**

```C#
// Get result with specified features to be retrieved
ImageAnalysisResult result = client.Analyze(
    BinaryData.FromStream(stream),
    VisualFeatures.Caption | 
    VisualFeatures.DenseCaptions |
    VisualFeatures.Objects |
    VisualFeatures.Tags |
    VisualFeatures.People);
```

**Python**

```Python
# Get result with specified features to be retrieved
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
    
4. En la función **AnalyzeImage**, en el comentario **Mostrar resultados de análisis**, agregue el siguiente código (incluidos los comentarios que indican dónde agregará más código más adelante):

**C#**

```C#
// Display analysis results
// Get image captions
if (result.Caption.Text != null)
{
    Console.WriteLine(" Caption:");
    Console.WriteLine($"   \"{result.Caption.Text}\", Confidence {result.Caption.Confidence:0.00}\n");
}

// Get image dense captions
Console.WriteLine(" Dense Captions:");
foreach (DenseCaption denseCaption in result.DenseCaptions.Values)
{
    Console.WriteLine($"   Caption: '{denseCaption.Text}', Confidence: {denseCaption.Confidence:0.00}");
}

// Get image tags


// Get objects in the image


// Get people in the image
```

**Python**

```Python
# Display analysis results
# Get image captions
if result.caption is not None:
    print("\nCaption:")
    print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))

# Get image dense captions
if result.dense_captions is not None:
    print("\nDense Captions:")
    for caption in result.dense_captions.list:
        print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get objects in the image


# Get people in the image

```
    
5. Guarde los cambios y vuelva al terminal integrado de la carpeta **image-analysis** y escriba el siguiente comando para ejecutar el programa con el argumento **images/street.jpg**:

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. Observe la salida, que debe incluir un título sugerido para la imagen **street.jpg**.
7. Vuelva a ejecutar el programa, esta vez con el argumento **images/building.jpg** para ver el título que se genera para la imagen **building.jpg**.
8. Repita el paso anterior para generar un título para el archivo **images/person.jpg**.

## Obtención de etiquetas sugeridas para una imagen

A veces puede ser útil identificar *etiquetas* relevantes que proporcionan pistas sobre el contenido de una imagen.

1. En la función **AnalyzeImage**, en el comentario **Get image tags** (Obtener etiquetas de imagen), agregue el código siguiente:

**C#**

```C#
// Get image tags
if (result.Tags.Values.Count > 0)
{
    Console.WriteLine($"\n Tags:");
    foreach (DetectedTag tag in result.Tags.Values)
    {
        Console.WriteLine($"   '{tag.Name}', Confidence: {tag.Confidence:F2}");
    }
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags.list:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images**, y observe que, además del título de la imagen, se muestra una lista de etiquetas sugeridas.

## Detección y localización de objetos en una imagen

La *detección de objetos* es una forma específica de Computer Vision en la que se identifican los objetos individuales dentro de una imagen y su ubicación se indica mediante un rectángulo delimitador.

1. En la función **AnalyzeImage**, en el comentario **Get objects in the image** (Obtener objetos en la imagen), agregue el código siguiente:

**C#**

```C#
// Get objects in the image
if (result.Objects.Values.Count > 0)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    stream.Close();
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedObject detectedObject in result.Objects.Values)
    {
        Console.WriteLine($"   \"{detectedObject.Tags[0].Name}\"");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Tags[0].Name,font,brush,(float)r.X, (float)r.Y);
    }

    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get objects in the image
if result.objects is not None:
    print("\nObjects in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects.list:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height)) 
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.tags[0].name,(r.x, r.y), backgroundcolor=color)

    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images**, y observe los objetos que se detectan. Después de cada ejecución, consulte el archivo **objects.jpg** que se genera en la misma carpeta que el archivo de código para ver las caras anotadas.

## Detectar y localizar personas en una imagen

La *detección de personas* es una forma específica de Computer Vision en la que se identifican personas individuales dentro de una imagen y su ubicación se indica mediante un cuadro de límite.

1. En la función **AnalyzeImage**, en el comentario **Get people in the image (Obtener personas en la imagen)**, agregue el siguiente código:

**C#**

```C#
// Get people in the image
if (result.People.Values.Count > 0)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedPerson person in result.People.Values)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        
        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }

    // Save annotated image
    String output_file = "persons.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get people in the image
if result.people is not None:
    print("\nPeople in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people.list:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
        draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
        
    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'people.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. (Opcional) Quite la marca de comentario del comando **Console.Writeline** en la sección **Devolver la confianza de la persona detectada** para revisar el nivel de confianza devuelto de que se detectara una persona en una posición determinada de la imagen.

3. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images**, y observe los objetos que se detectan. Después de cada ejecución, consulte el archivo **objects.jpg** que se genera en la misma carpeta que el archivo de código para ver las caras anotadas.

> **Nota**: En las tareas anteriores, usó un único método para analizar la imagen y, a continuación, agregó código incrementalmente para analizar y mostrar los resultados. El SDK también proporciona métodos individuales para sugerir títulos, identificar etiquetas, detectar objetos, entre otros, lo que significa que puede usar el método más adecuado para devolver solo la información que necesita, lo que reduce el tamaño de la carga de datos que se debe devolver. Consulte la [documentación del SDK de .NET](https://learn.microsoft.com/dotnet/api/overview/azure/cognitiveservices/computervision?view=azure-dotnet) o la [documentación del SDK de Python](https://learn.microsoft.com/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision) para más detalles.

## Quitar el fondo o generar un mate de primer plano de una imagen

En algunos casos, es posible que tenga que crear o eliminar el fondo de una imagen o puede que desee crear un mate en primer plano de esa imagen. Comencemos con la eliminación del fondo.

1. En el archivo de código, busque la función **BackgroundForeground** y, en el comentario **Remove the background from the image or generate a foreground matte (Eliminar el fondo de la imagen o generar un mate en primer plano)**, agregue el siguiente código:

**C#**

```C#
// Remove the background from the image or generate a foreground matte
Console.WriteLine($" Background removal:");
// Define the API version and mode
string apiVersion = "2023-02-01-preview";
string mode = "backgroundRemoval"; // Can be "foregroundMatting" or "backgroundRemoval"

string url = $"computervision/imageanalysis:segment?api-version={apiVersion}&mode={mode}";

// Make the REST call
using (var client = new HttpClient())
{
    var contentType = new MediaTypeWithQualityHeaderValue("application/json");
    client.BaseAddress = new Uri(endpoint);
    client.DefaultRequestHeaders.Accept.Add(contentType);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

    var data = new
    {
        url = $"https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{imageFile}?raw=true"
    };

    var jsonData = JsonSerializer.Serialize(data);
    var contentData = new StringContent(jsonData, Encoding.UTF8, contentType);
    var response = await client.PostAsync(url, contentData);

    if (response.IsSuccessStatusCode) {
        File.WriteAllBytes("background.png", response.Content.ReadAsByteArrayAsync().Result);
        Console.WriteLine("  Results saved in background.png\n");
    }
    else
    {
        Console.WriteLine($"API error: {response.ReasonPhrase} - Check your body url, key, and endpoint.");
    }
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemoving background from image...')
    
url = "{}computervision/imageanalysis:segment?api-version={}&mode={}".format(endpoint, api_version, mode)

headers= {
    "Ocp-Apim-Subscription-Key": key, 
    "Content-Type": "application/json" 
}

image_url="https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{}?raw=true".format(image_file)  

body = {
    "url": image_url,
}
    
response = requests.post(url, headers=headers, json=body)

image=response.content
with open("backgroundForeground.png", "wb") as file:
    file.write(image)
print('  Results saved in backgroundForeground.png \n')
```
    
2. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images** y abra el archivo **background.png** que se genera en la misma carpeta que el archivo de código para cada imagen.  Observe cómo se ha eliminado el fondo de cada una de las imágenes.

Ahora vamos a generar un mate en primer plano para nuestras imágenes.

3. En el archivo de código, busque la función **BackgroundForeground** y, en el comentario **Definir la versión y el modo** de la API, cambie la variable de modo a `foregroundMatting`.

4. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images** y abra el archivo **background.png** que se genera en la misma carpeta que el archivo de código para cada imagen.  Observe cómo se ha generado un mate en primer plano para las imágenes.

## Limpieza de recursos

Si no usa los recursos de Azure creados en este laboratorio para otros módulos de entrenamiento, puede eliminarlos para evitar incurrir en cargos adicionales. A continuación, se indica cómo puede hacerlo.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.

2. En la barra de búsqueda superior, busque *Cuenta de multiservicio de Servicios de Azure AI*, seleccione el recurso de la cuenta de multiservicio de Servicios de Azure AI que creó en este laboratorio.

3. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## Más información

En este ejercicio, ha explorado algunas de las capacidades de manipulación y análisis de imágenes del servicio Visión de Azure AI. El servicio también incluye capacidades para detectar objetos y personas y otras tareas de Computer Vision.

Para obtener más información sobre cómo usar el servicio **Visión de Azure AI**, consulte la [documentación de Visión de Azure AI](https://learn.microsoft.com/azure/ai-services/computer-vision/).
