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
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 0.15.1-beta.1
    ```

    > **Nota**: Si se le pide que instale extensiones del kit de desarrollo, puede cerrar el mensaje de forma segura.

    **Python**
    
    ```
    pip install azure-ai-vision==0.15.1b1
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
using Azure.AI.Vision.Common;
using Azure.AI.Vision.ImageAnalysis;
```

**Python**

```Python
# import namespaces
import azure.ai.vision as sdk
```
    
## Visualización de las imágenes que va a analizar

En este ejercicio, usará el servicio Visión de Azure AI para analizar varias imágenes.

1. En Visual Studio Code, expanda la carpeta **image-analysis** y la carpeta **images** que contiene.
2. Seleccione cada uno de los archivos de imagen por turnos para verlos en Visual Studio Code.

## Análisis de una imagen para sugerir un título

Ahora ya puede usar el SDK para llamar al servicio de Visión y analizar una imagen.

1. En el archivo de código de la aplicación cliente (**Program.cs** o **image-analysis.py**), en la función **Main**, observe que se ha proporcionado el código para cargar los valores de configuración. A continuación, busque el comentario **Authenticate Azure AI Vision client (Autenticar cliente de Computer Vision)**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para crear y autenticar un objeto de cliente de Visión de Azure AI:

**C#**

```C#
// Authenticate Azure AI Vision client
var cvClient = new VisionServiceOptions(
    aiSvcEndpoint,
    new AzureKeyCredential(aiSvcKey));
```

**Python**

```Python
# Authenticate Azure AI Vision client
cv_client = sdk.VisionServiceOptions(ai_endpoint, ai_key)
```

2. En la función **Main (Principal)**, en el código que acaba de agregar, observe que el código especifica la ruta de acceso a un archivo de imagen y, a continuación, pasa la ruta de acceso de la imagen a otras dos funciones (**AnalyzeImage** y **BackgroundForeground**). Estas funciones aún no están totalmente implementadas.

3. En la función **AnalyzeImage**, en el comentario **Specify features to be retrieved** (Especificar características que se recuperarán), agregue el código siguiente:

**C#**

```C#
// Specify features to be retrieved
Features =
    ImageAnalysisFeature.Caption
    | ImageAnalysisFeature.DenseCaptions
    | ImageAnalysisFeature.Objects
    | ImageAnalysisFeature.People
    | ImageAnalysisFeature.Text
    | ImageAnalysisFeature.Tags
```

**Python**

```Python
# Specify features to be retrieved
analysis_options = sdk.ImageAnalysisOptions()

features = analysis_options.features = (
    sdk.ImageAnalysisFeature.CAPTION |
    sdk.ImageAnalysisFeature.DENSE_CAPTIONS |
    sdk.ImageAnalysisFeature.TAGS |
    sdk.ImageAnalysisFeature.OBJECTS |
    sdk.ImageAnalysisFeature.PEOPLE
)
```
    
4. En la función **AnalyzeImage**, en el comentario **Get image analysis** (Obtener análisis de imágenes), agregue el código siguiente (incluidos los comentarios que indican dónde agregará más código con posterioridad):

**C#**

```C#
// Get image analysis
using var imageSource = VisionSource.FromFile(imageFile);

using var analyzer = new ImageAnalyzer(serviceOptions, imageSource, analysisOptions);

var result = analyzer.Analyze();

if (result.Reason == ImageAnalysisResultReason.Analyzed)
{
    // get image captions
    if (result.Caption != null)
    {
        Console.WriteLine(" Caption:");
        Console.WriteLine($"   \"{result.Caption.Content}\", Confidence {result.Caption.Confidence:0.0000}");
    }

    //get image dense captions
    if (result.DenseCaptions != null)
    {
        Console.WriteLine(" Dense Captions:");
        foreach (var caption in result.DenseCaptions)
        {
            Console.WriteLine($"   \"{caption.Content}\", Confidence {caption.Confidence:0.0000}");
        }
        Console.WriteLine($"\n");
    }

    // Get image tags


    // Get objects in the image


    // Get people in the image

}
else
{
    var errorDetails = ImageAnalysisErrorDetails.FromResult(result);
    Console.WriteLine(" Analysis failed.");
    Console.WriteLine($"   Error reason : {errorDetails.Reason}");
    Console.WriteLine($"   Error code : {errorDetails.ErrorCode}");
    Console.WriteLine($"   Error message: {errorDetails.Message}\n");
}
```

**Python**

```Python
# Get image analysis
image = sdk.VisionSource(image_file)

image_analyzer = sdk.ImageAnalyzer(cv_client, image, analysis_options)

result = image_analyzer.analyze()

if result.reason == sdk.ImageAnalysisResultReason.ANALYZED:
    # Get image captions
    if result.caption is not None:
        print("\nCaption:")
        print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.content, result.caption.confidence * 100))

    # Get image dense captions
    if result.dense_captions is not None:
        print("\nDense Captions:")
        for caption in result.dense_captions:
            print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.content, caption.confidence * 100))

    # Get image tags


    # Get objects in the image


    # Get people in the image


else:
    error_details = sdk.ImageAnalysisErrorDetails.from_result(result)
    print(" Analysis failed.")
    print("   Error reason: {}".format(error_details.reason))
    print("   Error code: {}".format(error_details.error_code))
    print("   Error message: {}".format(error_details.message))
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
if (result.Tags != null)
{
    Console.WriteLine($" Tags:");
    foreach (var tag in result.Tags)
    {
        Console.WriteLine($"   \"{tag.Name}\", Confidence {tag.Confidence:0.0000}");
    }
    Console.WriteLine($"\n");
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images**, y observe que, además del título de la imagen, se muestra una lista de etiquetas sugeridas.

## Detección y localización de objetos en una imagen

La *detección de objetos* es una forma específica de Computer Vision en la que se identifican los objetos individuales dentro de una imagen y su ubicación se indica mediante un rectángulo delimitador.

1. En la función **AnalyzeImage**, en el comentario **Get objects in the image** (Obtener objetos en la imagen), agregue el código siguiente:

**C#**

```C#
// Get objects in the image
if (result.Objects != null)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (var detectedObject in result.Objects)
    {
        Console.WriteLine($"   \"{detectedObject.Name}\", Confidence {detectedObject.Confidence:0.0000}");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Name,font,brush,r.X, r.Y);
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
    image = Image.open(image_file)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.name, detected_object.confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.name,(r.x, r.y), backgroundcolor=color)

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
if (result.People != null)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (var person in result.People)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox}, Confidence {person.Confidence:0.0000}");
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
    image = Image.open(image_file)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
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
Console.WriteLine($"\nRemove the background from the image or generate a foreground matte");

using var imageSource = VisionSource.FromFile(imageFile);

var analysisOptions = new ImageAnalysisOptions()
{
    // Set the image analysis segmentation mode to background or foreground
    SegmentationMode = ImageSegmentationMode.BackgroundRemoval
};

using var analyzer = new ImageAnalyzer(serviceOptions, imageSource, analysisOptions);

var result = analyzer.Analyze();

// Remove the background or generate a foreground matte
if (result.Reason == ImageAnalysisResultReason.Analyzed)
{
    using var segmentationResult = result.SegmentationResult;

    var imageBuffer = segmentationResult.ImageBuffer;
    Console.WriteLine($"\n Segmentation result:");
    Console.WriteLine($"   Output image buffer size (bytes) = {imageBuffer.Length}");
    Console.WriteLine($"   Output image height = {segmentationResult.ImageHeight}");
    Console.WriteLine($"   Output image width = {segmentationResult.ImageWidth}");

    string outputImageFile = "newimage.jpg";
    using (var fs = new FileStream(outputImageFile, FileMode.Create))
    {
        fs.Write(imageBuffer.Span);
    }
    Console.WriteLine($"   File {outputImageFile} written to disk\n");
}
else
{
    var errorDetails = ImageAnalysisErrorDetails.FromResult(result);
    Console.WriteLine(" Analysis failed.");
    Console.WriteLine($"   Error reason : {errorDetails.Reason}");
    Console.WriteLine($"   Error code : {errorDetails.ErrorCode}");
    Console.WriteLine($"   Error message: {errorDetails.Message}");
    Console.WriteLine(" Did you set the computer vision endpoint and key?\n");
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemove the background from the image or generate a foreground matte')

image = sdk.VisionSource(image_file)

analysis_options = sdk.ImageAnalysisOptions()

# Set the image analysis segmentation mode to background or foreground
analysis_options.segmentation_mode = sdk.ImageSegmentationMode.BACKGROUND_REMOVAL
    
image_analyzer = sdk.ImageAnalyzer(cv_client, image, analysis_options)

result = image_analyzer.analyze()

if result.reason == sdk.ImageAnalysisResultReason.ANALYZED:

    image_buffer = result.segmentation_result.image_buffer
    print(" Segmentation result:")
    print("   Output image buffer size (bytes) = {}".format(len(image_buffer)))
    print("   Output image height = {}".format(result.segmentation_result.image_height))
    print("   Output image width = {}".format(result.segmentation_result.image_width))

    output_image_file = "newimage.jpg"
    with open(output_image_file, 'wb') as binary_file:
        binary_file.write(image_buffer)
    print("   File {} written to disk".format(output_image_file))

else:

    error_details = sdk.ImageAnalysisErrorDetails.from_result(result)
    print(" Analysis failed.")
    print("   Error reason: {}".format(error_details.reason))
    print("   Error code: {}".format(error_details.error_code))
    print("   Error message: {}".format(error_details.message))
    print(" Did you set the computer vision endpoint and key?")
```
    
2. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images** y abra el archivo **newimage.jpg** que se genera en la misma carpeta que el archivo de código para cada imagen.  Observe cómo se ha eliminado el fondo de cada una de las imágenes.

Ahora vamos a generar un mate en primer plano para nuestras imágenes.

3. En el archivo de código, busque la función **BackgroundForeground** y, en el comentario **Set the image analysis segmentation mode to background or foreground (Establecer el modo de segmentación de análisis de imágenes en segundo plano o en primer plano)**, reemplace el código que agregó anteriormente por el siguiente código:

**C#**

```C#
// Set the image analysis segmentation mode to background or foreground
SegmentationMode = ImageSegmentationMode.ForegroundMatting
```

**Python**

```Python
# Set the image analysis segmentation mode to background or foreground
analysis_options.segmentation_mode = sdk.ImageSegmentationMode.FOREGROUND_MATTING
```

4. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images** y abra el archivo **newimage.jpg** que se genera en la misma carpeta que el archivo de código para cada imagen.  Observe cómo se ha generado un mate en primer plano para las imágenes.

## Limpieza de recursos

Si no usa los recursos de Azure creados en este laboratorio para otros módulos de entrenamiento, puede eliminarlos para evitar incurrir en cargos adicionales. A continuación, se indica cómo puede hacerlo.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.

2. En la barra de búsqueda superior, busque *Cuenta de multiservicio de Servicios de Azure AI*, seleccione el recurso de la cuenta de multiservicio de Servicios de Azure AI que creó en este laboratorio.

3. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## Más información

En este ejercicio, ha explorado algunas de las capacidades de manipulación y análisis de imágenes del servicio Visión de Azure AI. El servicio también incluye capacidades para detectar objetos y personas y otras tareas de Computer Vision.

Para obtener más información sobre cómo usar el servicio **Visión de Azure AI**, consulte la [documentación de Visión de Azure AI](https://learn.microsoft.com/azure/ai-services/computer-vision/).
