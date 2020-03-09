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
# Exemplo do Microsoft Graph OneDrive Photo Browser

O exemplo do Microsoft Graph OneDrive Photo Browser é um exemplo de aplicativo do Windows Universal que usa a [Biblioteca de Clientes do Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para C#/.NET.
O exemplo do aplicativo exibe apenas as imagens do OneDrive de um usuário. Observe que esse exemplo não funciona com o OneDrive for Business.

O exemplo usa o terminal de autenticação versão 2.0, que permite aos usuários entrem com a conta pessoal, corporativa ou de estudante do Microsoft.


## Configurar

### Pré-requisitos

Para executar o exemplo, será necessário: 

* Visual Studio 2015, com Ferramentas de desenvolvimento de aplicativos universais do Windows **Observação:** Se você não tiver Ferramentas de desenvolvimento de aplicativos universais do Windows instaladas, abra **Painel de controle** | **Desinstalar um programa**. Em seguida, clique com o botão direito do mouse em **Microsoft Visual Studio** e clique em **Alterar**. Selecione **Modificar** e, em seguida, escolha **Ferramentas de desenvolvimento de aplicativos universais do Windows**. Clique em **Atualizar**. Para obter mais informações sobre como configurar seu computador para o desenvolvimento de plataformas universais do Windows, confira [criar aplicativos UWP com o Visual Studio](https://msdn.microsoft.com/en-us/library/windows/apps/dn609832.aspx).
* Windows 10 ([habilitado para o modo de desenvolvimento](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx))
* Tanto a conta do [Microsoft](www.outlook.com) quanto a do [Office 365 for business](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account).
* Conhecimento do desenvolvimento de aplicativos universais do Windows

### Baixar o exemplo

1. Baixe o exemplo de [GitHub](https://github.com/OneDrive/graph-sample-photobrowser-uwp), escolhendo **Clone na Área de trabalho** ou **Baixar Zip**. 
2. No Visual Studio, abra o arquivo **OneDrivePhotoBrowser. sln** e crie-o.

\##Registre e configure o aplicativo

1. Entre no [Portal de registro do aplicativo](https://apps.dev.microsoft.com/) usando sua conta pessoal, corporativa ou de estudante.  
2. Selecione**Adicionar um aplicativo**.  
3. Insira um nome para o aplicativo e selecione **Criar aplicativo**. A página de registro será exibida, listando as propriedades do seu aplicativo.  
4. Em **Plataformas**, selecione **Adicionar plataforma**.  
5. Selecione **Aplicativo móvel**.  
6. Copie o valor da ID do cliente (ID do aplicativo) para a área de transferência. Será necessário usá-la no aplicativo do exemplo. Essa ID do aplicativo é o identificador exclusivo do aplicativo.   
7. Selecione **Salvar**.  

Após carregar a solução no Visual Studio, configure o exemplo para usar a ID do cliente que você registrou adicionando-o como uma chave no nó **Application.Resources** do arquivo App.xaml.

```xml
    <x:String x:Key="ida:ClientID">your Client Id</x:String>
```

## Executar o exemplo

1. Com o exemplo aberto no Visual Studio, na parte superior, selecione **Depurar** para Configurações de solução e **x86** ou **x64** para Plataformas de solução, e **OneDrivePhotoBrowser** para o projeto de inicialização. 
2. Verifique se você está executando o exemplo no **Computador local**.
3. Pressione **F5** ou clique em **Iniciar** para executar o exemplo.

O aplicativo de exemplo do OneDrive Photo Browser abrirá o OneDrive pessoal do usuário conectado com apenas pastas e imagens exibidas. Se o arquivo não for uma imagem, ele não será exibido no aplicativo OneDrive Photo Browser. Selecione uma pasta para visualizar todas as imagens nessa pasta. Selecione uma imagem para visualizar uma exibição maior da imagem com a exibição rolagem.


## Recursos da API

### Entrar no MSAL

Usuários podem entrar tanto na conta do [Microsoft](www.outlook.com) quanto na do [Office 365 for business](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account).

Depois que o usuário entrar, a classe `AuthenticationHelper` retorna um MSAL `GraphServicesClient`.

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

### Obter miniaturas de uma imagem no OneDrive

Nesse exemplo, as miniaturas são retornadas para um item se forem imagens. `GetAsync()` é usado para obter as propriedades do item.

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

## Mais recursos

* [Biblioteca de clientes do Microsoft Graph .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
* [Aplicativos universais do Windows](https://msdn.microsoft.com/en-us/library/windows/apps/dn726767.aspx) \- Mais informações sobre aplicativos universais do Windows

## Licença

[Licença](LICENSE.txt)

Este projeto adotou o [Código de conduta de código aberto da Microsoft](https://opensource.microsoft.com/codeofconduct/).  Para saber mais, confira as [Perguntas frequentes sobre o Código de Conduta](https://opensource.microsoft.com/codeofconduct/faq/) ou entre em contato pelo [opencode@microsoft.com](mailto:opencode@microsoft.com) se tiver outras dúvidas ou comentários.
