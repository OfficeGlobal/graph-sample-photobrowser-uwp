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
# Microsoft Graph OneDrive 照片浏览器示例

Microsoft Graph OneDrive 照片浏览器示例是一个 Windows 通用应用示例，它使用面向 C# /.NET 的 [Microsoft Graph .NET 客户端库](https://github.com/microsoftgraph/msgraph-sdk-dotnet)。
示例应用仅显示用户的OneDrive 中的图像项目。请注意，此示例不适用于 OneDrive for Business。

本示例使用 v2.0 版身份验证终结点，使用户可以通过其个人或工作/学校 Microsoft 帐户进行登录。


## 设置

### 先决条件

若要运行示例，将需要： 

* 配有通用 Windows 应用开发工具的 Visual Studio 2015** 备注：**如果未安装通用 Windows 应用开发工具，打开**控制面板** | **卸载程序**。然后右键单击 **Microsoft Visual Studio** 并单击“**更改**”。选择“**修改**”，然后选择“**通用 Windows 应用开发工具**”。单击“**更新**”。有关设置通用 Windows 平台开发计算机的更多信息，请参阅“[使用 Visual Studio 生成 UWP 应用](https://msdn.microsoft.com/en-us/library/windows/apps/dn609832.aspx)”。
* Windows 10（[已启用开发模式](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx)）
* [Microsoft](www.outlook.com) 或 [Office 365 商业版帐户](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account)。
* Windows 通用应用开发知识

### 下载示例

1. 通过选择“**复制至桌面**”或“**下载压缩文件**”从 [GitHub](https://github.com/OneDrive/graph-sample-photobrowser-uwp) 中下载示例。 
2. 在 Visual Studio 中，打开 **OneDrivePhotoBrowser.sln** 文件并生成示例。

\##注册和配置应用

1. 使用个人或工作或学校帐户登录到[应用注册门户](https://apps.dev.microsoft.com/)。  
2. 选择“**添加应用**”。  
3. 为应用输入名称，并选择“**创建应用程序**”。将显示注册页，其中列出应用的属性。  
4. 在“**平台**”下，选择“**添加平台**”。  
5. 选择“**移动应用程序**”。  
6. 将客户端 ID（应用 ID）值复制到剪贴板。你将需要在示例应用程序中使用它。应用 ID 是应用的唯一标识符。   
7. 选择“**保存**”。  

载入解决方案至 Visual Studio 中后，配置示例，以使用通过添加而注册为 App.xaml 文件 **Application.Resources** 节点密钥的客户端 ID。

```xml
    <x:String x:Key="ida:ClientID">your Client Id</x:String>
```

## 运行示例

1. 随着示例在 Visual Studio 中打开，在顶端为解决方案配置选择“**调试**”，为解决方案平台选择 **x86** 或 **x64**，并为启动项目选择 **OneDrivePhotoBrowser**。 
2. 检查是否在**本地计算机**上运行该示例。
3. 按下 **F5** 或单击**开始**来运行示例。

OneDrive 照片浏览器示例应用程序将打开已登录用户的私人 OneDrive，只显示文件夹和图像。如果文件不是图像，将不在 OneDrive 照片浏览器应用中显示。选择查看所有图像的文件夹。使用滚动视图，选择一幅图像，以查看图像的更大视图。


## API 功能

### MSAL 登录

用户可以使用 [Microsoft 账户](www.outlook.com)或 [Office 365 商业版账户](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account)登录。

用户登录后，`AuthenticationHelper` 类返回 MSAL `GraphServicesClient`。

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

### 获取 OneDrive 中图像缩略图

在此例中，如果不是图像，缩略图为某项返回。`GetAsync()` 用于获取项的属性。

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

## 更多资源

* [Microsoft Graph.NET 客户端库](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
* [Windows 通用应用](https://msdn.microsoft.com/en-us/library/windows/apps/dn726767.aspx) \- 有关 Windows 通用应用的更多信息

## 许可证

[许可证](LICENSE.txt)

此项目已采用 [Microsoft 开放源代码行为准则](https://opensource.microsoft.com/codeofconduct/)。有关详细信息，请参阅[行为准则常见问题解答](https://opensource.microsoft.com/codeofconduct/faq/)。如有其他任何问题或意见，也可联系 [opencode@microsoft.com](mailto:opencode@microsoft.com)。
