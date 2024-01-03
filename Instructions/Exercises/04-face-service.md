---
lab:
  title: Detección y análisis de caras
  module: 'Module 10 - Detecting, Analyzing, and Recognizing Faces'
---

# Detección y análisis de caras

La capacidad de detectar y analizar caras de personas es una funcionalidad básica de la inteligencia artificial. En este ejercicio, explorarás dos servicios de Azure AI que puedes usar para trabajar con caras de imágenes: el servicio **Visión de Azure AI** y el servicio **Face**.

> **Nota**: a partir del 21 de junio de 2022, las funcionalidades de servicios de Azure AI que devuelven información de identificación personal están restringidas a los clientes a los que se les ha concedido [acceso limitado](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Además, las funcionalidades que infieren el estado emocional ya no están disponibles. Estas restricciones pueden afectar a este ejercicio de laboratorio. Estamos trabajando para solucionar esto, pero, mientras tanto, es posible que experimente algunos errores al realizar los pasos siguientes; es por ello que nos disculpamos. Para obtener más detalles sobre los cambios realizados por Microsoft y la razón de estos, consulte el blog sobre [inversiones de inteligencia artificial y medidas de seguridad responsables para el reconocimiento facial](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## Clonación del repositorio para este curso

Si aún no lo ha hecho, debe clonar el repositorio de código para este curso:

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-vision` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## Aprovisionamiento de un recurso de Servicios de Azure AI

Si aún no tienes uno en tu suscripción, deberás aprovisionar un recurso de **Servicios de Azure AI**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. En la barra de búsqueda superior, busca *Servicios de Azure AI*, selecciona **Servicios de Azure AI** y crea un recurso de cuenta multiservicio de servicios de Azure AI con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: estándar S0
3. Active las casillas necesarias y cree el recurso.
4. Espere a que se complete la implementación y, a continuación, consulte los detalles.
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Claves y punto de conexión**. Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## Preparación para el uso del SDK de Visión de Azure AI

En este ejercicio, completarás una aplicación cliente parcialmente implementada que usa el SDK de Visión de Azure AI para analizar las caras de una imagen.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, navega hasta la carpeta **04-face** y expande la carpeta **C-Sharp** o **Python** según tus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **computer-vision** y abra un terminal integrado. Después, instala el paquete del SDK de Visión de Azure AI ejecutando el comando adecuado para tus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis --prerelease
    ```

    **Python**

    ```
    pip install azure-ai-vision
    ```
    
3. Consulte el contenido de la carpeta **computer-vision** y fíjese en que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

4. Abre el archivo de configuración y actualiza los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de servicios de Azure AI. Guarde los cambios.

5. Tenga en cuenta que la carpeta **computer-vision** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: detect-people.py

6. Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. Luego, en este comentario, agrega el siguiente código específico del lenguaje para importar los espacios de nombres que necesitarás para usar el SDK de Visión de Azure AI:

    **C#**

    ```C#
    // import namespaces
    using Azure.AI.Vision.Common;
    using Azure.AI.Vision.ImageAnalysis;
    ```

    **Python**

    ```Python
    # import namespaces
    import azure.ai.vision as sdk
    ```

## Visualización de la imagen que se va a analizar

En este ejercicio, usarás el servicio Visión de Azure AI para analizar una imagen de personas.

1. En Visual Studio Code, expanda la carpeta **computer-vision** y la carpeta **images** que contiene.
2. Seleccione la imagen **people.jpg** para verla.

## Detectar caras en una imagen

Ahora ya puedes usar el SDK para llamar al servicio Visión y detectar caras en una imagen.

1. En el archivo de código de la aplicación cliente (**Program.cs** o **detect-faces.py**), en la función **Main**, observa que se ha suministrado el código para cargar los valores de configuración. Después, busca el comentario **Autenticar cliente de Visión de Azure AI**. Después, en este comentario, agrega el siguiente código específico del lenguaje para crear y autenticar un objeto de cliente de Visión de Azure AI:

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

2. En la función **Main**, bajo el código que acabas de agregar, observa que el código especifica la ruta de acceso a un archivo de imagen y, luego, pasa la ruta de acceso de la imagen a una función denominada **AnalyzeImage**. Esta función aún no está totalmente implementada.

3. En la función **AnalyzeImage**, bajo el comentario **Specify features to be retrieved (PEOPLE)** (Especificar características que se recuperarán), añade el código siguiente:

    **C#**

    ```C#
    // Specify features to be retrieved (PEOPLE)
    Features =
        ImageAnalysisFeature.People
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (PEOPLE)
    analysis_options = sdk.ImageAnalysisOptions()
    
    features = analysis_options.features = (
        sdk.ImageAnalysisFeature.PEOPLE
    )    
    ```

4. En la función **AnalyzeFaces**, bajo del comentario **Get image analysis** (Obtener análisis de imágenes), añade el código siguiente:

    **C#**

    ```C
    // Get image analysis
    using var imageSource = VisionSource.FromFile(imageFile);
    
    using var analyzer = new ImageAnalyzer(serviceOptions, imageSource, analysisOptions);
    
    var result = analyzer.Analyze();
    
    if (result.Reason == ImageAnalysisResultReason.Analyzed)
    {
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
                // Draw object bounding box if confidence > 50%
                if (person.Confidence > 0.5)
                {
                    // Draw object bounding box
                    var r = person.BoundingBox;
                    Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
                    graphics.DrawRectangle(pen, rect);
        
                    // Return the confidence of the person detected
                    Console.WriteLine($"   Bounding box {person.BoundingBox}, Confidence {person.Confidence:0.0000}");
                }
            }
        
            // Save annotated image
            String output_file = "detected_people.jpg";
            image.Save(output_file);
            Console.WriteLine("  Results saved in " + output_file + "\n");
        }
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
                # Draw object bounding box if confidence > 50%
                if detected_people.confidence > 0.5:
                    # Draw object bounding box
                    r = detected_people.bounding_box
                    bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
                    draw.rectangle(bounding_box, outline=color, width=3)
            
                    # Return the confidence of the person detected
                    print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
                    
            # Save annotated image
            plt.imshow(image)
            plt.tight_layout(pad=0)
            outputfile = 'detected_people.jpg'
            fig.savefig(outputfile)
            print('  Results saved in', outputfile)
    
    else:
        error_details = sdk.ImageAnalysisErrorDetails.from_result(result)
        print(" Analysis failed.")
        print("   Error reason: {}".format(error_details.reason))
        print("   Error code: {}".format(error_details.error_code))
        print("   Error message: {}".format(error_details.message))
    ```

5. Guarde los cambios y vuelva al terminal integrado de la carpeta **computer-vision** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. Observe la salida, que debe indicar el número de caras detectadas.
7. Consulta el archivo **detected_faces.jpg** que se genera en la misma carpeta que el archivo de código para ver las caras anotadas. En este caso, el código ha usado los atributos de la cara para etiquetar la ubicación de la parte superior izquierda del rectángulo y las coordenadas del rectángulo delimitador para dibujar un rectángulo alrededor de cada cara.

## Preparación para el uso del SDK de Face

Aunque el servicio **Visión de Azure AI** ofrece detección de caras básica (junto con muchas otras funcionalidades de análisis de imágenes), el servicio **Face** proporciona una funcionalidad más completa de reconocimiento y análisis facial.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **19-face** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **face-api** y abra un terminal integrado. A continuación, instale el paquete del SDK de Face mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.8.0-preview.3
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.6.0
    ```
    
3. Consulte el contenido de la carpeta **face-api** y fíjese en que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

4. Abre el archivo de configuración y actualiza los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de servicios de Azure AI. Guarde los cambios.

5. Tenga en cuenta que la carpeta **face-api** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: analyze-faces.py

6. Abra el archivo de código y, en la parte superior, en las referencias de espacios de nombres existentes, busque el comentario **Import namespaces** (Importar espacios de nombres). Después, en este comentario, agrega el siguiente código específico del lenguaje para importar los espacios de nombres que necesitarás para usar el SDK de Visión:

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. En la función **Main**, observe que se ha proporcionado el código para cargar los valores de configuración. A continuación, busque el comentario **Authenticate Face client** (Autenticar cliente de Face). A continuación, en este comentario, agregue el siguiente código específico del lenguaje para crear y autenticar un objeto **FaceClient**:

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. En la función **Main**, en el código que acaba de agregar, tenga en cuenta que el código muestra un menú que le permite llamar a funciones en el código para explorar las funcionalidades del servicio Face. Implementará estas funciones en lo que queda de este ejercicio.

## Detección y análisis de caras

Una de las funcionalidades más fundamentales del servicio Face es la detección de caras de una imagen y la determinación de sus atributos, como la posición de la cabeza, el desenfoque, la presencia de gafas, etc.

1. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **1**. Este código llama a la función **DetectFaces** y pasa la ruta de acceso a un archivo de imagen.
2. Busque la función **DetectFaces** en el archivo de código y, debajo del comentario **Specify facial features to be retrieved** (Especificar características faciales que se recuperarán), agregue el código siguiente:

    **C#**

    ```C#
    // Specify facial features to be retrieved
    IList<FaceAttributeType> features = new FaceAttributeType[]
    {
        FaceAttributeType.Occlusion,
        FaceAttributeType.Blur,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.occlusion,
                FaceAttributeType.blur,
                FaceAttributeType.glasses]
    ```

3. En la función **DetectFaces**, en el código que acaba de agregar, busque el comentario **Get faces** (Obtener caras) y agregue el código siguiente:

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features, returnFaceId: false);

    if (detected_faces.Count() > 0)
    {
        Console.WriteLine($"{detected_faces.Count()} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.White);
        int faceCount=0;

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            faceCount++;
            Console.WriteLine($"\nFace number {faceCount}");
            
            // Get face properties
            Console.WriteLine($" - Mouth Occluded: {face.FaceAttributes.Occlusion.MouthOccluded}");
            Console.WriteLine($" - Eye Occluded: {face.FaceAttributes.Occlusion.EyeOccluded}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face number {faceCount}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features,                     return_face_id=False)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'
        face_count = 0

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))

            detected_attributes = face.face_attributes.as_dict()
            if 'blur' in detected_attributes:
                print(' - Blur:')
                for blur_name in detected_attributes['blur']:
                    print('   - {}: {}'.format(blur_name, detected_attributes['blur'][blur_name]))
                    
            if 'occlusion' in detected_attributes:
                print(' - Occlusion:')
                for occlusion_name in detected_attributes['occlusion']:
                    print('   - {}: {}'.format(occlusion_name, detected_attributes['occlusion'][occlusion_name]))

            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face number {}'.format(face_count)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. Examine el código que agregó a la función **DetectFaces**. Analiza un archivo de imagen y detecta las caras que contiene, incluidos los atributos de oclusión, desenfoque y la presencia de gafas. Se muestran los detalles de cada cara, incluido un identificador de cara único que se asigna a cada cara. La ubicación de las caras se indica en la imagen mediante un rectángulo delimitador.
5. Guarde los cambios y vuelva al terminal integrado de la carpeta **face-api** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    *La salida de C# puede mostrar advertencias sobre las funciones asincrónicas que ahora usan el operador **await**. No obstante, puede ignorarlas.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. Cuando se le solicite, escriba **1** y observe la salida, que debe incluir el id. y los atributos de cada cara detectada.
7. Consulte el archivo **detected_faces.jpg** que se genera en la misma carpeta que el archivo de código para ver las caras anotadas.

## Más información

Hay varias características adicionales disponibles en el servicio **Face** pero, en función del [Estándar de inteligencia artificial responsable](https://aka.ms/aah91ff), están restringidas por una directiva de acceso limitado. Estas características incluyen la identificación, comprobación y creación de modelos de reconocimiento facial. Para más información y solicitar acceso, consulta el [Acceso limitado para Servicios de Azure AI](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access).

Para obtener más información sobre cómo usar el servicio **Visión de Azure AI** para la detección de caras, consulta la [documentación de Visión de Azure AI](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Para obtener más información sobre el servicio **Face**, consulte la [documentación de Face](https://docs.microsoft.com/azure/cognitive-services/face/).
