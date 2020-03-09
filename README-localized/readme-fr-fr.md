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
# Exemple de navigateur de photos Microsoft Graph OneDrive

L’exemple de navigateur de photos Microsoft Graph OneDrive est un exemple d’application universelle Windows qui utilise la [bibliothèque cliente .NET Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
pour C#/.NET. L’exemple d’application affiche uniquement les éléments qui sont des images provenant de l’application OneDrive d’un utilisateur. Notez que cet exemple ne fonctionne pas avec OneDrive Entreprise.

L’exemple utilise le point de terminaison d’authentification v2.0, qui permet aux utilisateurs de se connecter avec leur compte Microsoft personnel, professionnel ou scolaire.


## Configurer

### Conditions préalables

Pour exécuter l’exemple, vous avez besoin des éléments suivants : 

* Visual Studio 2015, avec les outils de développement de l’application Windows universel **Remarque :** Si vous n’avez pas installé les outils de développement de l’application Windows universel, ouvrez **Panneau de configuration** | **Désinstaller un programme**. Cliquez ensuite avec le bouton droit sur **Microsoft Visual Studio**, puis cliquez sur **Modifier**. Sélectionnez **Modifier**, puis **Outils de développement d’applications Windows universelles**. Cliquez sur **Mettre à jour**. Pour plus d’informations sur la configuration de votre ordinateur pour le développement de plateformes Windows universelles, voir [Création d’applications UWP avec Visual Studio](https://msdn.microsoft.com/en-us/library/windows/apps/dn609832.aspx).
* Windows 10 ([avec mode de développement](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx))
* Un compte [Microsoft](www.outlook.com) ou un [compte Office 365 pour les entreprises](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account).
* Connaissance du développement d’applications universelles Windows

### Télécharger l’exemple

1. Téléchargez l’exemple à partir de [GitHub](https://github.com/OneDrive/graph-sample-photobrowser-uwp) en sélectionnant **Clone dans le bureau** ou **Télécharger le fichier zip**. 
2. Dans Visual Studio, ouvrez le fichier **OneDrivePhotoBrowser.sln** et générez-le.

\##Enregistrement et configuration de l’application

1. Connectez-vous au [portail d’inscription des applications](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.  
2. Sélectionnez **Ajouter une application**.  
3. Entrez un nom pour l’application, puis sélectionnez **Créer une application**. La page d’inscription s’affiche, répertoriant les propriétés de votre application.  
4. Sous **Plateformes**, sélectionnez **Ajouter une plateforme**.  
5. Sélectionnez **Application mobile**.  
6. Copiez la valeur d’ID client (Id d’application) dans le Presse-papiers. Vous devez l’utiliser dans l’exemple d’application. L’ID d’application est un identificateur unique pour votre application.   
7. Sélectionnez **Enregistrer**.  

Une fois la solution chargée dans Visual Studio, configurez l’exemple pour utiliser l’ID de client que vous avez enregistré en l’ajoutant comme clé dans le nœud **Application.Resources** du fichier App.xaml.

```xml
    <x:String x:Key="ida:ClientID">your Client Id</x:String>
```

## Exécuter l’exemple

1. Avec l’exemple Open dans Visual Studio, dans la partie supérieure, sélectionnez **Déboguer** pour les configurations de solution et **x86** ou **x64** pour les plateformes de solution et **OneDrivePhotoBrowser** pour le projet de démarrage. 
2. Vérifiez que vous exécutez l’exemple sur la **machine locale**.
3. Appuyez sur **F5** ou cliquez sur **démarrer** pour exécuter l’exemple.

L’exemple d’application de navigateur de photos OneDrive ouvre le OneDrive personnel de l’utilisateur connecté, avec uniquement les dossiers et les images affichés. Si le fichier n’est pas une image, il n’apparaît pas dans l’application de navigateur de photos OneDrive. Sélectionnez un dossier pour afficher les messages qu’il contient. Sélectionnez une image pour l’afficher en plus grand format, avec le mode de défilement.


## Fonctionnalités de l’API

### Connexion à la bibliothèque MSAL

Les utilisateurs peuvent se connecter avec un compte [Microsoft](www.outlook.com) ou un [compte Office 365 pour les entreprises](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account).

Une fois que l’utilisateur s’est connecté, la classe `AuthenticationHelper` renvoie MSAL `GraphServicesClient`.

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

### Obtenir des miniatures pour une image dans OneDrive

Dans cet exemple, des miniatures sont renvoyées pour un élément s’il s’agit d’une image. `GetAsync()` permet d’obtenir les propriétés de l’élément.

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

## Autres ressources

* [Bibliothèque cliente .NET Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
* [Applications universelles Windows](https://msdn.microsoft.com/en-us/library/windows/apps/dn726767.aspx) : plus d’informations sur les applications universelles Windows

## Licence

[Licence](LICENSE.txt)

Ce projet a adopté le [Code de conduite Open Source de Microsoft](https://opensource.microsoft.com/codeofconduct/). Pour en savoir plus, reportez-vous à la [FAQ relative au code de conduite](https://opensource.microsoft.com/codeofconduct/faq/) ou contactez [opencode@microsoft.com](mailto:opencode@microsoft.com) pour toute question ou tout commentaire.
