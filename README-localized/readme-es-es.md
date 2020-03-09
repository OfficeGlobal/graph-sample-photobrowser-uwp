---
page_type: sample
products:
- office-onedrive
- ms-graph
languages:
- csharp
extensions:
  contentType: samples
  technologies:
  - Microsoft Graph
  services:
  - OneDrive
  createdDate: 2/21/2017 10:21:21 AM
---
# Ejemplo de explorador de fotos de OneDrive para Microsoft Graph

El ejemplo de explorador de fotos de OneDrive para Microsoft Graph es una aplicación universal de Windows que usa la [biblioteca del cliente .NET de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para C#/.NET.
La aplicación de ejemplo muestra solo los elementos que son imágenes del OneDrive de un usuario. Tenga en cuenta que este ejemplo no funciona con OneDrive para la Empresa.

En el ejemplo, se usa el punto de conexión de autenticación v2.0, que permite a los usuarios iniciar sesión con sus cuentas Microsoft personales, o bien con sus cuentas Microsoft profesionales o educativas.


## Configurar

### Requisitos previos

Para ejecutar el ejemplo, necesitará: 

* Visual Studio 2015, con las herramientas de desarrollo de aplicaciones universales de Windows **Nota:** Si no tiene instaladas las herramientas de desarrollo de aplicaciones universales de Windows, abra **Panel de control** | **desinstalar un programa**. Después, haga clic con el botón derecho en **Microsoft Visual Studio** y luego haga clic en **Cambiar**. Seleccione **Modificar** y, a continuación, **elija las herramientas de desarrollo de aplicaciones universales de Windows**. Haga clic en **actualizar**. Para obtener más información sobre cómo configurar su equipo para el desarrollo de la Plataforma Universal de Windows, vea[Crear Aplicaciones para UWP con Visual Studio](https://msdn.microsoft.com/en-us/library/windows/apps/dn609832.aspx).
* Windows 10 ([modo de desarrollo habilitado](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx))
* Ya sea una[](www.outlook.com)cuenta de ](www.outlook.com)Microsoft[](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account) o bien de Office 365 para empresa](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account).
* Conocimientos de desarrollo de aplicaciones universales de Windows

### Descarga del ejemplo

1. Descargue el ejemplo [de GitHub](https://github.com/OneDrive/graph-sample-photobrowser-uwp) eligiendo **clónico en el escritorio** o **descargue Zip**. 
2. En Visual Studio, abra el archivo **OneDrivePhotoBrowser.sln** y compile.

\##Registre y configure la aplicación

1. Inicie sesión en el [Portal de Registro de Aplicaciones](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.  
2. Seleccione **Agregar una aplicación**.  
3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**. Se muestra la página de registro, indicando las propiedades de la aplicación.  
4. En **Plataformas**, seleccione **Agregar plataforma**.  
5. Seleccione **Aplicación móvil**.  
6. Copie el valor del Id. de Cliente (Id. de la aplicación) en el portapapeles. Debe usarlo en la aplicación de ejemplo. El id. de la aplicación es un identificador único para su aplicación.   
7. Seleccione **Guardar**.  

Después de haber cargado la solución en Visual Studio, configure el ejemplo para usar el identificador de cliente que registró al agregarlo como clave en **Recursos de la Aplicación** nodo del archivo app. Xaml.

```xml
    <x:String x:Key="ida:ClientID">your Client Id</x:String>
```

## Ejecutar el ejemplo

1. Con el ejemplo abierto en Visual Studio, en la parte superior, seleccione**Depuración** de las Configuraciones de Soluciones y **x86** o **x64** para las plataformas de soluciones, y **OneDrivePhotoBrowser** para el proyecto de inicio. 
2. Compruebe que está ejecutando el ejemplo en el **Equipo Local**.
3. Presione **F5** o bien haga clic en**Inicio** para ejecutar el ejemplo.

La aplicación de ejemplo OneDrive Photo Browser se abrirá en la OneDrive personal del usuario que ha iniciado sesión, donde solo se mostrarán las carpetas y las imágenes. Si el archivo no es una imagen, no se mostrará en la Aplicación Explorador de Fotos de OneDrive. Seleccione una carpeta para ver todas las imágenes de esa carpeta. Seleccione una imagen para ver una presentación más grande de la imagen, con la vista de desplazamiento.


## Características de la API 

### Inicio de sesión MSAL

Los usuarios pueden iniciar sesión con una cuenta[de Microsoft](www.outlook.com) o [Office 365 para empresas](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account).

Después de que el usuario inicie sesión, la clase`AuthenticationHelper` devuelve unMSAL` GraphServicesClient`.

```csharp
        public static GraphServiceClient GetAuthenticatedClient()
        {
            if (graphClient == null)
            {
                // Create Microsoft Graph client.
                try
                {
                    graphClient = new GraphServiceClient(
                        "https://graph.microsoft.com/v1.0",
                        new DelegateAuthenticationProvider(
                            async (requestMessage) =>
                            {
                                var token = await GetTokenForUserAsync();
                                requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", token);
                                // This header has been added to identify our sample in the Microsoft Graph service.  If extracting this code for your project please remove.
                                requestMessage.Headers.Add("SampleID", "uwp-csharp-photobrowser-sample");

                            }));
                    return graphClient;
                }

                catch (Exception ex)
                {
                    Debug.WriteLine("Could not create a graph client: " + ex.Message);
                }
            }

            return graphClient;
        }
```

### Obtener miniaturas de una imagen en OneDrive

En este ejemplo, se devuelven miniaturas para un elemento, si se trata de una imagen. `GetAsync ()` se utiliza para obtener las propiedades del elemento.

```csharp
           IEnumerable<DriveItem> items;

            var expandString = "thumbnails, children($expand=thumbnails)";

            // If id isn't set, get the OneDrive root's photos and folders. Otherwise, get those for the specified item ID.
            // Also retrieve the thumbnails for each item if using a consumer client.
            var itemRequest = string.IsNullOrEmpty(id)
                ? this.graphClient.Me.Drive.Root.Request().Expand(expandString)
                : this.graphClient.Me.Drive.Items[id].Request().Expand(expandString);

            var item = await itemRequest.GetAsync();
            items = item.Children == null
                ? new List<DriveItem>()
                : item.Children.CurrentPage.Where(child => child.Folder != null || child.Image != null);
```

## Más recursos

* [Biblioteca cliente .NET de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
* [Aplicaciones Universales de Windows](https://msdn.microsoft.com/en-us/library/windows/apps/dn726767.aspx)Obtenga más información sobre las aplicaciones universales de Windows

## Licencia

[Licencia](LICENSE.txt)

Este proyecto ha adoptado el [Código de conducta de código abierto de Microsoft](https://opensource.microsoft.com/codeofconduct/). Para obtener más información, vea [Preguntas frecuentes sobre el código de conducta](https://opensource.microsoft.com/codeofconduct/faq/) o póngase en contacto con [opencode@microsoft.com](mailto:opencode@microsoft.com) si tiene otras preguntas o comentarios.
