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
# Microsoft Graph OneDrive 写真ブラウザー サンプル

Microsoft Graph OneDrive 写真ブラウザー サンプルは、C#/.NET. 用の [Microsoft Graph .NET クライアント ライブラリ](https://github.com/microsoftgraph/msgraph-sdk-dotnet)を使用する Windows ユニバーサル アプリのサンプルです。このサンプル アプリは、ユーザーの OneDrive
にある画像項目のみを表示します。このサンプルは OneDrive for Business で動作しないことに注意してください。

このサンプルは v2.0 の認証エンドポイントを使用します。このエンドポイントにより、ユーザーは個人用か、職場または学校の Microsoft アカウントでサインインできます。


## セットアップ

### 前提条件

サンプルを実行するには、以下のものが必要です。 

* ユニバーサル Windows アプリ開発ツールがインストールされた Visual Studio 2015 **注**:ユニバーサル Windows アプリ開発ツールがインストールされていない場合は、[**コントロール パネル**] | [**プログラムのアンインストール**] を開きます。次に、[**Microsoft Visual Studio**] を右クリックして、[**変更**] をクリックします。[**変更**] を選択し、[**ユニバーサル Windows アプリ開発ツール**] を選択します。[**更新**] をクリックします。ユニバーサル Windows プラットフォーム開発用コンピューターのセットアップの詳細については、「[Build UWP apps with Visual Studio (Visual Studio で UWP アプリを構築する)](https://msdn.microsoft.com/en-us/library/windows/apps/dn609832.aspx)」を参照してください。
* Windows 10 ([開発モードが有効](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx))
* [Microsoft](www.outlook.com) または [Office 365 for Business アカウント](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account)のいずれか。
* Windows ユニバーサル アプリの開発に関する知識

### サンプルのダウンロード

1. [**デスクトップの複製**] または [**Zip のダウンロード**] を選択して、[GitHub](https://github.com/OneDrive/graph-sample-photobrowser-uwp) からサンプルをダウンロードします。 
2. Visual Studio で **OneDrivePhotoBrowser.sln** ファイルを開き、ビルドします。

\##アプリを登録して構成する

1. 個人用アカウントか職場または学校アカウントのいずれかを使用して、[アプリ登録ポータル](https://apps.dev.microsoft.com/)にサインインします。  
2. [**アプリの追加**] を選択します。  
3. アプリの名前を入力して、[**アプリケーションの作成**] を選択します。登録ページが表示され、アプリのプロパティが一覧表示されます。  
4. [**プラットフォーム**] で、[**プラットフォームの追加**] を選択します。  
5. [**モバイル アプリケーション**] を選択します。  
6. クライアント ID (アプリ ID) の値をクリップボードにコピーします。この値はサンプル アプリで使用する必要があります。アプリ ID は、アプリの一意識別子です。   
7. [**保存**] を選択します。  

Visual Studio にソリューションを読み込んだ後、App.xaml ファイルの **Application.Resources** ノードにキーとして追加することにより、登録したクライアント ID を使用するようにサンプルを構成します。

```xml
    <x:String x:Key="ida:ClientID">your Client Id</x:String>
```

## サンプルの実行

1. Visual Studio でサンプルを開いた状態で、上部で、ソリューション構成用に **Debug**、ソリューション プラットフォーム用に **x86** または **x64**、スタートアップ プロジェクト用に **OneDrivePhotoBrowser** を選択します。 
2. **ローカル コンピューター**でサンプルを実行していることを確認します。
3. **F5** キーを押すか、[**開始**] をクリックしてサンプルを実行します。

OneDrive 写真ブラウザー サンプル アプリは、サインインしたユーザーの個人用 OneDrive を開き、フォルダーと画像のみを表示します。ファイルが画像でない場合、OneDrive 写真ブラウザー アプリには表示されません。フォルダーを選択すると、そのフォルダー内のすべての画像が表示されます。画像を選択すると、スクロール ビューで画像が大きく表示されます。


## API の機能

### MSAL サインイン

ユーザーは [Microsoft](www.outlook.com) または [Office 365 for Business アカウント](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account)のいずれかを使用してログインできます。

ユーザーがサインインすると、`AuthenticationHelper` クラスは MSAL `GraphServicesClient` を返します。

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

### OneDrive で画像のサムネイルを取得する

この例では、アイテムが画像の場合に、サムネイルが返されます。`GetAsync()` は、アイテムのプロパティを取得するために使用されます。

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

## その他のリソース

* [Microsoft Graph .NET クライアント ライブラリ](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
* [Windows ユニバーサル アプリ](https://msdn.microsoft.com/en-us/library/windows/apps/dn726767.aspx) \- Windows ユニバーサル アプリに関する詳細情報

## ライセンス

[ライセンス](LICENSE.txt)

このプロジェクトでは、[Microsoft オープン ソース倫理規定](https://opensource.microsoft.com/codeofconduct/) が採用されています。詳細については、「[Code of Conduct の FAQ (倫理規定の FAQ)](https://opensource.microsoft.com/codeofconduct/faq/)」を参照してください。また、その他の質問やコメントがあれば、[opencode@microsoft.com](mailto:opencode@microsoft.com) までお問い合わせください。
