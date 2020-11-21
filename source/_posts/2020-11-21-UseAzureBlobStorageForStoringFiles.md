---
title: 使用Azure Blob Storage來存儲檔案(代替Firebase Storage)
tags:
    - .NET
    - Azure
    - Flutter
    - Firebase
  
categories: App Development 
photos:
    - https://imgur.com/A8NZfXI.png
---


# 前言
我們之前原本是使用Flutter+Firebase來做Prototype的, 所以很理所當然地使用Firebase Storage來存儲使用者上傳的影片以及照片, 但Firebase Storage的上傳速度不慎理想, 所以我們打算使用Azure Blob Storage來試試看; 當我們以27.39MB的檔案做測試時, 測試的結果Azure blob storage的檔案傳輸速度約為Firebase Storage的1.94倍

所以我們馬上將所有的檔案傳輸功能放到Azure上, 但同時我們也保留前端Firebase Storage的代碼, 所以當我們要改回使用Firebase Storage時, 我們就只需要更改一個前端的參數即可

![SpeedShowing](https://imgur.com/A8NZfXI.png)

# 上傳流程[[參考]](https://docs.microsoft.com/en-us/azure/storage/common/storage-sas-overview)
1. 使用者將一個有效的Token傳送至伺服器
2. 伺服器確認該Token有效後, 為此使永用者建立一個有使用效期的User delegation SAS.
3. 將此SAS回傳至客戶端, 該客戶則可使用該SAS上傳檔案至Azure Blob Storage
4. 客戶端使用Put Request將檔案傳輸到Azure Blob storage
5. 再將下載該檔案的網址存取到數據庫中

# 實作流程
1. 需先在Azure Portal 中建立Azure Blob Storage, [[教學可看此]](https://xenby.com/b/238-%e6%95%99%e5%ad%b8-azure-blob-storage-%e4%bd%bf%e7%94%a8%e6%8c%87%e5%8d%97-%e5%89%b5%e5%bb%ba%e7%af%87)
2. 設置Azure Activity Directory, 並登記你的App (App Registeration) [[官方教學]](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal), 你必須需要取得的有 (1) Tenant/Directory ID (2) Application/Client ID (3) Client secret
3. 為Azure Blob Storage中為剛創立的App設置所有的Permission, 進到我們創建的Azure blob storage中, 將剛剛在步驟2所創建的App, 在IAM中設置為Storage Blob Delegatior & Storage Blob Data Contributor & Owner. (在指派角色時, 需要用剛剛註冊的App名稱去搜尋, 才可以搜尋到)
   
#### 當前面這些需要在Portal設定好的許可設定好後, 後面的部分就只是在前後端加上代碼而已
4. 在 .NET Core後端新增一個用於給客戶端取得SAS的Controller, 此Controller使用Token Auth確認身分後, 將帶有SAS的上傳網址回傳給客戶端:
```csharp
        [Authorize()]
        [HttpGet("SAS")]
        public async Task<IActionResult> GetAzureBlobSASForUploading()
        {

            // Auth checking.
            Guid loginUserId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
            User loginUser = await _userRepository.GetUser(loginUserId);

            // Checking the user is currently in the database.
            if (loginUser == null )
            {
                return Unauthorized();
            }

            // Example from: https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-user-delegation-sas-create-dotnet
            string accountName = "yourStorageAccountName";
            string blobEndpoint = $"https://{accountName}.blob.core.windows.net";

            string tenantId = "targetIdOrDirectoryIdFromAzureAppRegisteration";
            string applicationId = "applicationIdOrClientIdFromAzureAppRegisteration";
            string clientSecret = "ClientSecretForYouRegisteredApp";  //https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal#option-2-create-a-new-application-secret

            // Connecting to the stroage.
            BlobServiceClient blobClient = new BlobServiceClient(new Uri(blobEndpoint), new ClientSecretCredential(
                tenantId,
                applicationId,
                clientSecret
                )
            );


            // Get a user delegation key for the Blob service that's valid for seven days.
            // You can use the key to generate any number of shared access signatures over the lifetime of the key.
            DateTimeOffset expireAt = DateTimeOffset.UtcNow.AddDays(7);
            UserDelegationKey key = await blobClient.GetUserDelegationKeyAsync(startsOn: null, expiresOn: expireAt);

            // Read the key's properties.
            Console.WriteLine("User delegation key properties:");
            Console.WriteLine("Key signed start: {0}", key.SignedStartsOn);
            Console.WriteLine("Key signed expiry: {0}", key.SignedExpiresOn);
            Console.WriteLine("Key signed object ID: {0}", key.SignedObjectId);
            Console.WriteLine("Key signed tenant ID: {0}", key.SignedTenantId);
            Console.WriteLine("Key signed service: {0}", key.SignedService);
            Console.WriteLine("Key signed version: {0}", key.SignedVersion);

            string containerName = "gofunsportcontainer";

            // Create a SAS token that's valid for one hour.
            BlobSasBuilder sasBuilder = new BlobSasBuilder()
            {
                BlobContainerName = containerName,
                Resource = "c",
                ExpiresOn = expireAt
            };

            // Specify read and write permissions for the SAS.
            //sasBuilder.SetPermissions(BlobSasPermissions.Read);
            //sasBuilder.SetPermissions(BlobSasPermissions.Write);
            sasBuilder.SetPermissions(BlobAccountSasPermissions.All);

            // Use the key to get the SAS token.
            string sas = sasBuilder.ToSasQueryParameters(key, accountName).ToString();

            // Construct the full URI, including the SAS token.
            UriBuilder fullUri = new UriBuilder()
            {
                Scheme = "https",
                Host = $"{accountName}.blob.core.windows.net",
                Path = $"{containerName}",
                Query = sas
            };

            Console.WriteLine("User delegation SAS URI: {0}", fullUri);
            Console.WriteLine();
            return Ok(new { 
                sas,
                expireAt,
                fullUri = fullUri.Uri,
            });
        }

```
5. 在前端加上取得SAS的代碼, 並將SAS存取於客戶端中, 若SAS過期則重新取得一次; 並加上上傳bytes的代碼:
```dart
  String _sasToken;
  DateTime _sasExpireAt;

  Future<void> getSAS(BuildContext context) async {
    String url = apiUrl + "/sas";

    final res = await http.get(
      url,
      headers: await HttpRequestHelpers.getHeader(
        context,
        token: token,
      ),
    );

    if (res.statusCode >= 400) {
      throw HttpException(res.body);
    }

    if (res.body == null || res.body.isEmpty) {
      return null;
    }

    var resData = json.decode(res.body);

    DateTime returnedExpireAt = DateTime.tryParse(resData["expireAt"]).toUtc();
    String returnedSasToken = resData["sas"];
    String fullUri = resData["fullUri"];

    setSASTokenAndExpirationTime(
      returnedSasToken,
      returnedExpireAt,
      fullUri,
    );
  }

  void setSASTokenAndExpirationTime(
    String inputSasToken,
    DateTime inputExpireAt,
    String inputUri,
  ) {
    sasExpireAt = inputExpireAt;
    sasToken = inputSasToken;
    fullUri = inputUri;
  }

  bool get sasExpired {
    return sasExpireAt.isBefore(DateTime.now().toUtc());
  }

  bool get sasValid {
    return sasToken != null && sasExpireAt != null && sasExpired == false;
  }

  Future<void> ensureSasStatus(
    BuildContext context,
  ) async {
    if (sasValid != true) {
      await getSAS(context);
    }
  }

  Future<String> _uploadUint8ListToAzureBlobUrl(
    String uploadingUrl,
    Uint8List bytes,
  ) async {
    http.Response res = await http.put(
      uploadingUrl,
      body: bytes,
      headers: {"x-ms-blob-type": "BlockBlob"},
    );

    if (res.statusCode >= 400) {
      throw Exception(res.reasonPhrase);
    }

    String downloadUrl = uploadingUrl.split("?").elementAt(0);

    // Remove the params.
    return downloadUrl;
  }

```

# 結語 
在轉成Azure blob storage的過程中, 最頭痛的無非是Azure Active Directory跟Storage中IAM的設定, 因為沒有把創建的App設置為應該設置的角色(只指派角色給了自己的Microsoft Account, 而不是在Azure Activity Directory), 所以後端始終都無法與Azure blob storage對接去取得User delegation key; 但在轉換完成後上傳的速度真的加快很多; 客戶端的使用體驗也好多了. (原本使用Firebase storage有時候會慢到以為已經斷線了). 