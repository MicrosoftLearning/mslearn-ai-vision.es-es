---
lab:
  title: Lectura de texto en imágenes
  module: Module 11 - Reading Text in Images and Documents
---

# Lectura de texto en imágenes

El reconocimiento óptico de caracteres (OCR) es un subconjunto de Computer Vision que se ocupa de leer texto en imágenes y documentos. El servicio **Visión de Azure AI** proporciona una API para leer texto, que explorará en este ejercicio.

## Aprovisionar un recurso de Computer Vision

Si aún no tiene una en la suscripción, deberá aprovisionar un recurso de **Computer Vision**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
1. Seleccione **Crear un recurso**.
1. En la barra de búsqueda, busque *Computer Vision*, seleccione **Computer Vision** y cree el recurso con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elige entre Este de EE. UU., Oeste de EE. UU., Centro de Francia, Centro de Corea del Sur, Norte de Europa, Sudeste Asiático, Oeste de Europa o Este de Asia\**
    - **Nombre**: *escribe un nombre único*
    - **Plan de tarifa**: F0 gratis.

    \*Los conjuntos de características completas de Visión de Azure AI 4.0 solo están disponibles actualmente en estas regiones.

1. Active las casillas necesarias y cree el recurso.
1. Espere a que se complete la implementación y, a continuación, consulte los detalles.
1. Cuando se haya implementado el recurso, vaya a él y vea su página **Claves y punto de conexión**. Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## Clonación del repositorio para este curso

Vas a desarrollar el código mediante Cloud Shell desde Azure Portal. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

> **Sugerencia**: Si ya ha clonado el repositorio **mslearn-ai-vision** recientemente, puede omitir esta tarea. De lo contrario, sigue estos pasos para clonarlo en el entorno de desarrollo.

1. En Azure Portal, usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. En el panel de PowerShell, escribe los siguientes comandos para clonar el repo de GitHub para este ejercicio:

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    ```
   cd mslearn-ai-vision/Labfiles/05-ocr
    ```

## Prepararse para usar el SDK de Visión de Azure AI

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa el SDK de Visión de Azure AI para leer texto.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. Vaya a la carpeta que contiene los archivos de código de la aplicación para su lenguaje preferido:  

    **C#**

    ```
   cd C-Sharp/read-text
    ```
    
    **Python**

    ```
   cd Python/read-text
    ```

1. Instale el paquete del SDK de Visión de Azure AI y las dependencias necesarias mediante la ejecución de los comandos adecuados para sus preferencias de lenguaje:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0
    dotnet add package SkiaSharp --version 3.116.1
    dotnet add package SkiaSharp.NativeAssets.Linux --version 3.116.1
    ``` 

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0
    pip install dotenv
    pip install matplotlib
    ```

1. Con el  comando `ls`, puede ver el contenido de la carpeta **computer-vision**. Observe que contiene un archivo para los valores de configuración:

    - **C#**: appsettings.json
    - **Python**: .env

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    **C#**

    ```
   code appsettings.json
    ```

    **Python**

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Computer Vision.
1. Después de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o usa la acción de **hacer clic con el botón derecho > Guardar** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** o la acción de **hacer clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

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
    using SkiaSharp;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

1. En el archivo de código de la aplicación cliente, en la función **Main (Principal)**, se ha proporcionado el código para cargar la configuración. A continuación, busque el comentario **Authenticate Azure AI Vision client (Autenticar cliente de Computer Vision)**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para crear y autenticar un objeto de cliente de Visión de Azure AI:

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

1. En la función **Main (Principal)**, en el código que acaba de agregar, observe que el código especifica la ruta de acceso a un archivo de imagen y, a continuación, pasa la ruta de acceso de la imagen a una función denominada **GetTextRead**. Esta función aún no está totalmente implementada.

1. Vamos a agregar código al cuerpo de la función **GetTextRead**. Busque el comentario **Use Analyze image function to read text in image (Usar la función Analyze Image para leer texto en la imagen)**. Después, en este comentario, agregue el código específico del lenguaje siguiente, teniendo en cuenta que las características visuales se especifican al llamar a la función `Analyze`:

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
    
        // Load the image using SkiaSharp
        using SKBitmap bitmap = SKBitmap.Decode(imageFile);
        // Create canvas to draw on the bitmap
        using SKCanvas canvas = new SKCanvas(bitmap);

        // Create paint for drawing polygons (bounding boxes)
        SKPaint paint = new SKPaint
        {
            Color = SKColors.Cyan,
            StrokeWidth = 3,
            Style = SKPaintStyle.Stroke,
            IsAntialias = true
        };

        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save the annotated image using SkiaSharp
        using (SKFileWStream output = new SKFileWStream("text.jpg"))
        {
            // Encode the bitmap into JPEG format with full quality (100)
            bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
        }

        Console.WriteLine("\nResults saved in text.jpg\n");
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

1. En el código que acaba de agregar en la función **GetTextRead**, y en el comentario **Devuelve el texto detectado en la imagen**, agregue el código siguiente (este código imprime el texto de la imagen en la consola y genera la imagen **text.jpg** que resalta el texto de la imagen):

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    bool drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

    DrawPolygon(canvas, polygonPoints, paint);
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

1. Solo para el archivo de programa de C#, se necesita una función auxiliar para dibujar un polígono. En el comentario **Método auxiliar para dibujar un polígono dada una matriz de SKPoints**, agregue el código siguiente:

    **C#**
   
    ```C#
    // Helper method to draw a polygon given an array of SKPoints
    static void DrawPolygon(SKCanvas canvas, SKPoint[] points, SKPaint paint)
    {
        if (points == null || points.Length == 0)
            return;

        using (var path = new SKPath())
        {
            path.MoveTo(points[0]);
            for (int i = 1; i < points.Length; i++)
            {
                path.LineTo(points[i]);
            }
            path.Close();
            canvas.DrawPath(path, paint);
        }
    }
    ```

1. En la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **1**. Este código llama a la función **GetTextRead** y pasa la ruta de acceso al archivo de imagen *Lincoln.jpg*.

1. Guarde los cambios y cierre el editor de código.

1. En la barra de herramientas de Cloud Shell, seleccione **Cargar o descargar archivos** y, a continuación, elija **Descargar**. En el cuadro de diálogo nuevo, escriba la siguiente ruta de acceso del archivo y seleccione **Descargar**:

    **C#**
   
    ```
    mslearn-ai-vision/Labfiles/05-ocr/C-Sharp/read-text/images/Lincoln.jpg
    ```

    **Python**

    ```
    mslearn-ai-vision/Labfiles/05-ocr/Python/read-text/images/Lincoln.jpg
    ```
       
1. Abra la imagen **Lincoln.jpg** para verla.

1. Después de ver la imagen que procesa el código, escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Cuando se le solicite, escriba **1** y observe la salida, que es el texto extraído de la imagen.

1. En la carpeta **read-text**, se ha creado una imagen **text.jpg**. Puede descargarla mediante la ruta de acceso del archivo `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/text.jpg` y observar que hay un polígono alrededor de cada *línea* de texto.

1. Vuelva a abrir el archivo de código y busque el comentario **Devolver a su posición el rectángulo de selección que rodea cada línea**. A continuación, en este comentario, agregue el siguiente código:

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

1. Guarde los cambios, cierre el editor de código y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Cuando se le solicite, escriba **1** y observe la salida, que debe ser cada línea de texto de la imagen con su posición respectiva en la imagen.

1. Vuelva a abrir el archivo de código y busque el comentario **Devolver a su posición cada palabra detectada en la imagen y el rectángulo de selección que rodea cada palabra con el nivel de confianza de cada una**. A continuación, en este comentario, agregue el siguiente código:

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        // Convert the bounding polygon into an array of SKPoints    
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

        // Draw the word polygon on the canvas
        DrawPolygon(canvas, polygonPoints, paint);
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

1. Guarde los cambios, cierre el editor de código y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Cuando se le solicite, escriba **1** y observe la salida, que debe ser cada palabra de texto de la imagen con su posición respectiva en la imagen. Observe cómo también se devuelve el nivel de confianza de cada palabra.

1. Descargue de nuevo la imagen **text.jpg** y observe que hay un polígono alrededor de cada *palabra*.

## Usar el SDK de Visión de Azure AI para leer texto escrito a mano de una imagen

En el ejercicio anterior, leyó texto bien definido de una imagen, pero a veces es posible que quiera leer texto de notas o papeles escritos a mano. La buena noticia es que el **SDK de Visión de Azure AI** también puede leer texto escrito a mano con el mismo código que usó para leer texto bien definido. Usaremos el mismo código del ejercicio anterior, pero esta vez usaremos una imagen diferente.

1. Descargue **Note.jpg** mediante la ruta de acceso del archivo `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/images/Note.jpg` para ver la siguiente imagen que procesa el código.

1. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **2**. Este código llama a la función **GetTextRead** y pasa la ruta de acceso al archivo de imagen *Note.jpg*.

1. En el terminal, escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Cuando se le solicite, escriba **2** y observe la salida, que es el texto extraído de la imagen de la nota.

1. Descargue de nuevo la imagen **text.jpg** y observe que hay un polígono alrededor de cada *palabra* de la nota.

## Limpieza de recursos

Si no usa los recursos de Azure creados en este laboratorio para otros módulos de entrenamiento, puede eliminarlos para evitar incurrir en cargos adicionales. A continuación, se indica cómo puede hacerlo.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.

1. En la barra de búsqueda superior, busque *Computer Vision* y seleccione el recurso de Computer Vision que creó en este laboratorio.

1. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## Más información

Para obtener más información sobre cómo usar el servicio **Visión de Azure AI** para leer texto, consulte la [documentación de Visión de Azure AI](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr).
