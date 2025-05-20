---
lab:
  title: Detección y análisis de caras
  module: Module 4 - Detecting and Analyze Faces
---

# Detección y análisis de caras

La capacidad de detectar y analizar caras de personas es una funcionalidad básica de la inteligencia artificial. En este ejercicio, explorará dos servicios de Azure AI que puede usar para trabajar con caras en imágenes: el servicio **Azure AI Vision** y el servicio **Face**.

> **Importante**: Este laboratorio se puede completar sin solicitar acceso adicional a características restringidas.

> **Nota**: A partir del 21 de junio de 2022, las capacidades de los Servicios de Azure AI que devuelven información de identificación personal están restringidas a los clientes a los que se les ha concedido [acceso limitado](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Además, las funcionalidades que infieren el estado emocional ya no están disponibles. Para obtener más detalles sobre los cambios realizados por Microsoft y la razón de estos, consulte el blog sobre [inversiones de inteligencia artificial y medidas de seguridad responsables para el reconocimiento facial](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## Clonación del repositorio para este curso

Si aún no lo ha hecho, debe clonar el repositorio de código para este curso:

1. Inicie Visual Studio Code.
2. Abre la paleta (Mayús + Ctrl + P) y ejecuta un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-vision` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## Aprovisionamiento de un recurso de Servicios de Azure AI

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso de **Servicios de Azure AI**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. En la barra de búsqueda superior, busque *Servicios de Azure AI*, seleccione **Servicios de Azure AI** y cree un recurso de cuenta de varios servicios de Azure AI con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: estándar S0
3. Active las casillas necesarias y cree el recurso.
4. Espere a que se complete la implementación y, a continuación, consulte los detalles.
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Claves y punto de conexión**. Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## Prepararse para usar el SDK de Visión de Azure AI

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa el SDK de Visión de Azure AI para analizar las caras en una imagen.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorer**, navega hasta la carpeta **Labfiles/04-face** y expande la carpeta **C-Sharp** o **Python** según tus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **computer-vision** y abra un terminal integrado. A continuación, instale el paquete del SDK de Visión de Azure AI mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.3
    ```

    **Python**

    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b3
    ```
    
3. Consulte el contenido de la carpeta **computer-vision** y fíjese en que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

4. Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Servicios de Azure AI. Guarde los cambios.

5. Tenga en cuenta que la carpeta **computer-vision** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: detect-people.py

6. Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Import namespaces (Importar espacios de nombres)**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Visión de Azure AI:

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

## Visualización de la imagen que se va a analizar

En este ejercicio, usará el servicio Visión de Azure AI para analizar una imagen de personas.

1. En Visual Studio Code, expanda la carpeta **computer-vision** y la carpeta **images** que contiene.
2. Seleccione la imagen **people.jpg** para verla.

## Detectar caras en una imagen

Ahora ya puede usar el SDK para llamar al servicio Visión y detectar caras en una imagen.

1. En el archivo de código de la aplicación cliente (**Program.cs** o **detect-faces.py**), en la función **Main**, observa que se ha suministrado el código para cargar los valores de configuración. A continuación, busque el comentario **Authenticate Azure AI Vision client (Autenticar cliente de Computer Vision)**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para crear y autenticar un objeto de cliente de Visión de Azure AI:

    **C#**

    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient cvClient = new ImageAnalysisClient(
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

2. En la función **Main**, bajo el código que acabas de agregar, observa que el código especifica la ruta de acceso a un archivo de imagen y, luego, pasa la ruta de acceso de la imagen a una función denominada **AnalyzeImage**. Esta función aún no está totalmente implementada.

3. En la función **AnalyzeImage**, bajo el comentario **Obtener el resultado con las características especificadas que se recuperarán (PERSONAS)**, añade el código siguiente:

    **C#**

    ```C#
    // Get result with specified features to be retrieved (PEOPLE)
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        VisualFeatures.People);
    ```

    **Python**

    ```Python
    # Get result with specified features to be retrieved (PEOPLE)
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.PEOPLE],
    )
    ```

4. En la función **AnalyzeImage**, bajo el comentario **Dibujar cuadro de límite alrededor de las personas detectadas**, añade el código siguiente:

    **C#**

    ```C
    // Draw bounding box around detected people
    foreach (DetectedPerson person in result.People.Values)
    {
        if (person.Confidence > 0.5) 
        {
            // Draw object bounding box
            var r = person.BoundingBox;
            Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
        }

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }
    ```

    **Python**
    
    ```Python
    # Draw bounding box around detected people
    for detected_people in result.people.list:
        if(detected_people.confidence > 0.5):
            # Draw object bounding box
            r = detected_people.bounding_box
            bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
            draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
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
7. Consulta el archivo **people.jpg** que se genera en la misma carpeta que el archivo de código para ver las caras anotadas. En este caso, el código ha usado los atributos de la cara para etiquetar la ubicación de la parte superior izquierda del rectángulo y las coordenadas del rectángulo delimitador para dibujar un rectángulo alrededor de cada cara.

Si quieres ver la puntuación de confianza de todas las personas detectadas por el servicio, puedes quitar la marca de comentario de la línea de código bajo el comentario `Return the confidence of the person detected` y volver a ejecutar el código.

## Preparación para el uso del SDK de Face

Aunque el servicio **Visión de Azure AI** ofrece detección de caras básica (junto con muchas otras capacidades de análisis de imágenes), el servicio **Face** proporciona una funcionalidad más completa de reconocimiento y análisis facial.

1. En Visual Studio Code, en el panel **Explorer**, navega hasta la carpeta **Labfiles/04-face** y expande la carpeta **C-Sharp** o **Python** según tus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **face-api** y abra un terminal integrado. A continuación, instale el paquete del SDK de Face mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Azure.AI.Vision.Face -v 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-vision-face==1.0.0b2
    ```
    
3. Consulte el contenido de la carpeta **face-api** y fíjese en que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

4. Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Servicios de Azure AI. Guarde los cambios.

5. Tenga en cuenta que la carpeta **face-api** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: analyze-faces.py

6. Abra el archivo de código y, en la parte superior, en las referencias de espacios de nombres existentes, busque el comentario **Import namespaces** (Importar espacios de nombres). A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Visión:

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Vision.Face;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.ai.vision.face import FaceClient
    from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection03
    from azure.core.credentials import AzureKeyCredential
    ```

7. En la función **Main**, observe que se ha proporcionado el código para cargar los valores de configuración. A continuación, busque el comentario **Authenticate Face client** (Autenticar cliente de Face). A continuación, en este comentario, agregue el siguiente código específico del lenguaje para crear y autenticar un objeto **FaceClient**:

    **C#**

    ```C#
    // Authenticate Face client
    faceClient = new FaceClient(
        new Uri(cogSvcEndpoint),
        new AzureKeyCredential(cogSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Face client
    face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key)
    )
    ```

8. En la función **Main (Principal)**, en el código que acaba de agregar, tenga en cuenta que el código muestra un menú que le permite llamar a funciones en el código para explorar las capacidades del servicio Face. Implementará estas funciones en lo que queda de este ejercicio.

## Detección y análisis de caras

Una de las funcionalidades más fundamentales del servicio Face es la detección de caras de una imagen y la determinación de sus atributos, como la posición de la cabeza, el desenfoque, la presencia de una máscara, etc.

1. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **1**. Este código llama a la función **DetectFaces** y pasa la ruta de acceso a un archivo de imagen.
2. Busque la función **DetectFaces** en el archivo de código y, debajo del comentario **Specify facial features to be retrieved** (Especificar características faciales que se recuperarán), agregue el código siguiente:

    **C#**

    ```C#
    // Specify facial features to be retrieved
    FaceAttributeType[] features = new FaceAttributeType[]
    {
        FaceAttributeType.Detection03.HeadPose,
        FaceAttributeType.Detection03.Blur,
        FaceAttributeType.Detection03.Mask
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeTypeDetection03.HEAD_POSE,
                FaceAttributeTypeDetection03.BLUR,
                FaceAttributeTypeDetection03.MASK]
    ```

3. En la función **DetectFaces**, en el código que acaba de agregar, busque el comentario **Get faces** (Obtener caras) y agregue el código siguiente:

**C#**

```C#
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var response = await faceClient.DetectAsync(
        BinaryData.FromStream(imageData),
        FaceDetectionModel.Detection03,
        FaceRecognitionModel.Recognition04,
        returnFaceId: false,
        returnFaceAttributes: features);
    IReadOnlyList<FaceDetectionResult> detected_faces = response.Value;

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
            Console.WriteLine($" - Head Pose (Yaw): {face.FaceAttributes.HeadPose.Yaw}");
            Console.WriteLine($" - Head Pose (Pitch): {face.FaceAttributes.HeadPose.Pitch}");
            Console.WriteLine($" - Head Pose (Roll): {face.FaceAttributes.HeadPose.Roll}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Mask: {face.FaceAttributes.Mask.Type}");

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
    detected_faces = face_client.detect(
        image_content=image_data.read(),
        detection_model=FaceDetectionModel.DETECTION03,
        recognition_model=FaceRecognitionModel.RECOGNITION04,
        return_face_id=False,
        return_face_attributes=features,
    )

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

            print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
            print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
            print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
            print(' - Blur: {}'.format(face.face_attributes.blur.blur_level))
            print(' - Mask: {}'.format(face.face_attributes.mask.type))

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

4. Examine el código que agregó a la función **DetectFaces**. Analiza un archivo de imagen y detecta las caras que contiene, incluidos atributos como la posición de la cabeza, el desenfoque y la presencia de una máscara. Se muestran los detalles de cada cara, incluido un identificador de cara único que se asigna a cada cara. La ubicación de las caras se indica en la imagen mediante un rectángulo delimitador.
5. Guarde los cambios y vuelva al terminal integrado de la carpeta **face-api** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    *La salida de C# puede mostrar advertencias sobre las funciones asincrónicas que no usan el operador **await**. No obstante, puedes ignorarlas.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. Cuando se le solicite, escriba **1** y observe la salida, que debe incluir el id. y los atributos de cada cara detectada.
7. Consulte el archivo **detected_faces.jpg** que se genera en la misma carpeta que el archivo de código para ver las caras anotadas.

## Más información

Hay varias características adicionales disponibles en el servicio **Face** pero, en función del [Estándar de inteligencia artificial responsable](https://aka.ms/aah91ff), están restringidas por una directiva de acceso limitado. Estas características incluyen la identificación, comprobación y creación de modelos de reconocimiento facial. Para más información y solicitar acceso, consulte el [acceso limitado a los Servicios de Azure AI](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access).

Para obtener más información sobre cómo usar el servicio **Visión de Azure AI** para la detección de caras, consulte la [documentación de Visión de Azure AI](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Para obtener más información sobre el servicio **Face**, consulte la [documentación de Face](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-identity).
