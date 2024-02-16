---
lab:
  title: Lectura de texto en imágenes
  module: Module 11 - Reading Text in Images and Documents
---

# Lectura de texto en imágenes

El reconocimiento óptico de caracteres (OCR) es un subconjunto de Computer Vision que se ocupa de leer texto en imágenes y documentos. El servicio **Visión de Azure AI** proporciona una API para leer texto, que explorará en este ejercicio.

## Clonación del repositorio para este curso

Si aún no lo ha hecho, debe clonar el repositorio de código para este curso:

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
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Keys and Endpoint** (Claves y punto de conexión). Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## Prepararse para usar el SDK de Visión de Azure AI

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa el SDK de Visión de Azure AI para leer texto.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los siguientes pasos, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **Labfiles\05-ocr** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **read-text** y abra un terminal integrado. A continuación, instale el paquete del SDK de Visión de Azure AI mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **Nota**: Si se le pide que instale extensiones del kit de desarrollo, puede cerrar el mensaje de forma segura.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```

3. Consulte el contenido de la carpeta **read-text** y fíjese en que contiene un archivo para los valores de configuración:

    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Servicios de Azure AI. Guarde los cambios.


## Usar el SDK de Visión de Azure AI para leer texto de una imagen

Una de las características del **SDK de Visión de Azure AI** es leer texto de una imagen. En este ejercicio, completará una aplicación cliente parcialmente implementada que usa el SDK de Visión de Azure AI para leer texto de una imagen.

1. La carpeta **read-text** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: read-text.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacios de nombres existentes, busque el comentario **Import namespaces** (Importar espacios de nombres). A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Visión de Azure AI:

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

2. En el archivo de código de la aplicación cliente, en la función **Main (Principal)**, se ha proporcionado el código para cargar la configuración. A continuación, busque el comentario **Authenticate Azure AI Vision client (Autenticar cliente de Computer Vision)**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para crear y autenticar un objeto de cliente de Visión de Azure AI:

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

3. En la función **Main (Principal)**, en el código que acaba de agregar, observe que el código especifica la ruta de acceso a un archivo de imagen y, a continuación, pasa la ruta de acceso de la imagen a una función denominada **GetTextRead**. Esta función aún no está totalmente implementada.

4. Vamos a agregar código al cuerpo de la función **GetTextRead**. Busque el comentario **Use Analyze image function to read text in image (Usar la función Analyze Image para leer texto en la imagen)**. Después, en este comentario, agregue el código específico del lenguaje siguiente, teniendo en cuenta que las características visuales se especifican al llamar a la función `Analyze`:

    **C#**

    ```C#
    // Use Analyze image function to read text in image
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        // Specify the features to be retrieved
        VisualFeatures.Read);
    
    stream.Close();
    
    // Display analysis results
    if (result.Read != null)
    {
        Console.WriteLine($"Text:");
    
        // Prepare image for drawing
        System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.Cyan, 3);
        
        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save image
        String output_file = "text.jpg";
        image.Save(output_file);
        Console.WriteLine("\nResults saved in " + output_file + "\n");   
    }
    ```
    
    **Python**
    
    ```Python
    # Use Analyze image function to read text in image
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ]
    )

    # Display the image and overlay it with the extracted text
    if result.read is not None:
        print("\nText:")

        # Prepare image for drawing
        image = Image.open(image_file)
        fig = plt.figure(figsize=(image.width/100, image.height/100))
        plt.axis('off')
        draw = ImageDraw.Draw(image)
        color = 'cyan'

        for line in result.read.blocks[0].lines:
            # Return the text detected in the image

            
        # Save image
        plt.imshow(image)
        plt.tight_layout(pad=0)
        outputfile = 'text.jpg'
        fig.savefig(outputfile)
        print('\n  Results saved in', outputfile)
    ```

5. En el código que acaba de agregar en la función **GetTextRead**, y en el comentario **Devuelve el texto detectado en la imagen**, agregue el código siguiente (este código imprime el texto de la imagen en la consola y genera la imagen **text.jpg** que resalta el texto de la imagen):

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    var drawLinePolygon = true;
    
    // Return each line detected in the image and the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
    }
    ```
    
    **Python**
    
    ```Python
    # Return the text detected in the image
    print(f"  {line.text}")    
    
    drawLinePolygon = True
    
    r = line.bounding_polygon
    bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
    
    # Return the position bounding box around each line
    
    
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    # Draw line bounding polygon
    if drawLinePolygon:
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

6. En la carpeta **read-text/images**, seleccione **Lincoln.jpg** para ver el archivo que procesa el código.

7. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **1**. Este código llama a la función **GetTextRead** y pasa la ruta de acceso al archivo de imagen *Lincoln.jpg*.

8. Guarde los cambios y vuelva al terminal integrado de la carpeta **read-text** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

9. Cuando se le solicite, escriba **1** y observe la salida, que es el texto extraído de la imagen.

10. En la carpeta **read-text**, seleccione la imagen **text.jpg** y observe cómo hay un polígono alrededor de cada *línea* de texto.

11. Vuelva al archivo de código en Visual Studio Code y busque el comentario **Devuelve el cuadro de límite de posición alrededor de cada línea**. A continuación, en este comentario, agregue el siguiente código:

    **C#**
    
    ```C#
    // Return the position bounding box around each line
    Console.WriteLine($"   Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");  
    ```
    
    **Python**
    
    ```Python
    # Return the position bounding box around each line
    print("   Bounding Polygon: {}".format(bounding_polygon))
    ```

12. Guarde los cambios y vuelva al terminal integrado de la carpeta **read-text** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

13. Cuando se le solicite, escriba **1** y observe la salida, que debe ser cada línea de texto de la imagen con su posición respectiva en la imagen.


14. Vuelva al archivo de código de Visual Studio Code y busque el comentario **Return each word detected in the image and the position bounding box around each word with the confidence level of each word (Devolver cada palabra detectada en la imagen y el cuadro de límite de posición alrededor de cada palabra con el nivel de confianza de cada una)**. A continuación, en este comentario, agregue el siguiente código:

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
    }
    ```
    
    **Python**
    
    ```Python
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    for word in line.words:
        r = word.bounding_polygon
        bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
        print(f"    Word: '{word.text}', Bounding Polygon: {bounding_polygon}, Confidence: {word.confidence:.4f}")
    
        # Draw word bounding polygon
        drawLinePolygon = False
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

15. Guarde los cambios y vuelva al terminal integrado de la carpeta **read-text** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

16. Cuando se le solicite, escriba **1** y observe la salida, que debe ser cada palabra de texto de la imagen con su posición respectiva en la imagen. Observe cómo también se devuelve el nivel de confianza de cada palabra.

17. En la carpeta **read-text**, seleccione la imagen **text.jpg** y observe cómo hay un polígono alrededor de cada *palabra*.

## Usar el SDK de Visión de Azure AI para leer texto escrito a mano de una imagen

En el ejercicio anterior, leyó texto bien definido de una imagen, pero a veces es posible que quiera leer texto de notas o papeles escritos a mano. La buena noticia es que el **SDK de Visión de Azure AI** también puede leer texto escrito a mano con el mismo código que usó para leer texto bien definido. Usaremos el mismo código del ejercicio anterior, pero esta vez usaremos una imagen diferente.

1. En la carpeta **read-text/images**, seleccione **Note.jpg** para ver el archivo que procesa el código.

2. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **2**. Este código llama a la función **GetTextRead** y pasa la ruta de acceso al archivo de imagen *Note.jpg*.

3. Desde el terminal integrado de la carpeta **read-text**, escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

4. Cuando se le solicite, escriba **2** y observe la salida, que es el texto extraído de la imagen de la nota.

5. En la carpeta **read-text**, seleccione la imagen **text.jpg** y observe cómo hay un polígono alrededor de cada *palabra * de la nota.

## Limpieza de recursos

Si no usa los recursos de Azure creados en este laboratorio para otros módulos de entrenamiento, puede eliminarlos para evitar incurrir en cargos adicionales. A continuación, se indica cómo puede hacerlo.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.

2. En la barra de búsqueda superior, busque *Cuenta de multiservicio de Servicios de Azure AI*, seleccione el recurso de la cuenta de multiservicio de Servicios de Azure AI que creó en este laboratorio

3. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## Más información

Para obtener más información sobre cómo usar el servicio **Visión de Azure AI** para leer texto, consulte la [documentación de Visión de Azure AI](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr).
