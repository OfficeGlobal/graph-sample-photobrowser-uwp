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
# Пример браузера фотографий Microsoft Graph в OneDrive

Образец браузера Microsoft Graph OneDrive — это универсальное приложение Windows,
в котором используется [библиотека клиента Microsoft Graph .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) для C#/.нет. Пример приложения отображает только те элементы из библиотеки OneDrive пользователя, которые являются изображениями. Обратите внимание, что этот пример не работает с OneDrive для бизнеса.

В этом примере для проверки подлинности используется конечная точка версии 2.0, при помощи которой пользователи могут выполнять вход с использованием личных, рабочих и учебных учетных записей Майкрософт.


## Настройка

### Предварительные требования

Для запуска примера кода вам потребуются: 

* Visual Studio 2015 с инструментами разработки универсальных приложений для Windows **Примечание:** Если у вас не установлены универсальные средства разработки приложений для Windows, откройте **Панель управления** | **Удалите программу**. Щелкните правой кнопкой мыши **Microsoft Visual Studio** и выберите пункт **изменить**. Нажмите кнопку **изменить** а затем выберите **средств разработки универсальных приложений Windows**. Нажмите кнопку **Обновить**. Дополнительные сведения о настройке компьютера для универсальной разработки платформы Windows см. в статье [создание приложений UWP в среде Visual Studio](https://msdn.microsoft.com/en-us/library/windows/apps/dn609832.aspx).
* Windows 10 ([с включенным режимом разработки](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx)).
* Учетная запись [Microsoft](www.outlook.com) или [Office 365 для бизнеса](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account).
* Знание разработки универсального приложения для Windows

### Скачивание примера приложения

1. Загрузите образец с [GitHub](https://github.com/OneDrive/graph-sample-photobrowser-uwp), выбрав **Clone in Desktop** или **Download Zip**. 
2. В Visual Studio откройте файл **OneDrivePhotoBrowser.sln** и создайте его.

\## Зарегистрируйтесь и настройте приложение

1. Войдите на [портал регистрации приложений](https://apps.dev.microsoft.com/) с помощью личной, рабочей или учебной учетной записи.  
2. Выберите пункт**Добавить приложение**.  
3. Введите имя приложения и выберите пункт **Создать приложение**. Откроется страница регистрации со списком свойств приложения.  
4. В разделе**Платформы**, нажмите**Добавление платформы**.  
5. Выберите пункт**Мобильное приложение**.  
6. Скопируйте значение идентификатора клиента (App Id) в буфер обмена. Вам нужно будет использовать его в примере приложения. Идентификатор приложения является уникальным.   
7. Нажмите кнопку **Сохранить**.  

После загрузки решения в Visual Studio настройте образец для использования зарегистрированного идентификатора клиента, добавив его в качестве ключа в узел **Application.Resources** файла App.xaml.

```xml
    <x:String x:Key="ida:ClientID">your Client Id</x:String>
```

## Запустите пример

1. Открыв пример в Visual Studio, в верхней части экрана выберите **Отладка** для конфигураций решений и **x86** или **x64** для платформ решений и **OneDrivePhotoBrowser** для автозагружаемого проекта. 
2. Убедитесь, что вы запускаете образец на **локальном компьютере**.
3. Нажмите **F5** или нажмите **Пуск**, чтобы запустить образец.

Пример приложения OneDrive Photo Browser откроет личный OneDrive вошедшего в систему пользователя с отображением только папок и изображений. Если файл не является изображением, он не будет отображаться в приложении OneDrive Photo Browser. Выберите папку, чтобы увидеть все изображения в этой папке. Выберите изображение, чтобы увидеть увеличенное изображение с видом прокрутки.


## Особенности API

### Вход в MSAL

Пользователи могут войти в систему с помощью [Microsoft](www.outlook.com) или [Office 365 для бизнес-аккаунта](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account).

После входа пользователя класс `AuthenticationHelper` возвращает MSAL `GraphServicesClient`.

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

### Как получить эскизы изображения в OneDrive

В этом примере миниатюры возвращаются для элемента, если это изображение. `GetAsync()` используется для получения свойств элемента.

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

## Дополнительные ресурсы

* [Клиентская библиотека .NET Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
* [Универсальные приложения Windows](https://msdn.microsoft.com/en-us/library/windows/apps/dn726767.aspx) \- дополнительная информация о универсальных приложениях Windows

## Лицензия

[Лицензия](LICENSE.txt)

Этот проект соответствует [Правилам поведения разработчиков открытого кода Майкрософт](https://opensource.microsoft.com/codeofconduct/). Дополнительные сведения см. в разделе [часто задаваемых вопросов о правилах поведения](https://opensource.microsoft.com/codeofconduct/faq/). Если у вас возникли вопросы или замечания, напишите нам по адресу [opencode@microsoft.com](mailto:opencode@microsoft.com).
