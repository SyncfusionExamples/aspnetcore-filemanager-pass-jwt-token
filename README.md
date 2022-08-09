# aspnetcore-filemanager-pass-jwt-token

This repository contains the Blazor FileManager component to send JWT token from client to server in the File Manager component.

## How to run this application?

To run this application, you need to first clone the [`blazor-filemanager-pass-jwt-token`](https://github.com/SyncfusionExamples/blazor-filemanager-pass-jwt-token) repository and then navigate to its appropriate path where it has been located in your system.

To do so, open the command prompt and run the below commands one after the other.

```
git clone https://github.com/SyncfusionExamples/blazor-filemanager-pass-jwt-token 

cd blazor-filemanager-pass-jwt-token

```

## Restore the NuGet package and build the application

To restore the NuGet package, run the following command in root folder of the application.

```
dotnet restore
```

To build the application, run the following command.

```
dotnet build
```

## Running application

After successful compilation, run the following command to run the application.

```
dotnet run
```

## File Manager authorization header for read and upload operation

To send the authorization header data from client side to server side use the below FileManager events by setting the folder in the GetBucketList method.

| **File Operations** | **Events** |
| --- | --- |
| Read, Delete, Upload, Rename, Copy         | [`BeforeSend`](https://help.syncfusion.com/cr/aspnetcore-js2/Syncfusion.EJ2.FileManager.FileManager.html#Syncfusion_EJ2_FileManager_FileManager_BeforeSend) |
| GetImage      | [`BeforeImageLoad`](https://help.syncfusion.com/cr/aspnetcore-js2/Syncfusion.EJ2.FileManager.FileManager.html#Syncfusion_EJ2_FileManager_FileManager_BeforeImageLoad) |
| Download     | [`BeforeDownload`](https://help.syncfusion.com/cr/aspnetcore-js2/Syncfusion.EJ2.FileManager.FileManager.html#Syncfusion_EJ2_FileManager_FileManager_BeforeDownload) |

Refer to the code snippet of `Index.cshtml` page

```
<ejs-filemanager id="filemanager" view="@Syncfusion.EJ2.FileManager.ViewType.Details" beforeSend="beforeSend" beforeImageLoad="beforeImageLoad" beforeDownload="beforeDownload">
...
</ejs-filemanager>

<script>

function beforeSend(args) {
    //Ajax beforeSend event 
    args.ajaxSettings.beforeSend = function (args) {
        //Setting authorization header              
        args.httpRequest.setRequestHeader("Authorization", "Pictures"); 
    }
}
function beforeImageLoad(args) {
    var rootFileName = "Pictures";
    args.imageUrl = args.imageUrl + '&rootName=' + rootFileName;
}
function beforeDownload(args) {
    var rootFileName = "Pictures" ;
    var includeCustomAttribute = args.data;
    includeCustomAttribute.rootName = rootFileName;
    args.data = includeCustomAttribute;
}
</script>
```

Refer to the code snippet of `HomeController.cs` page

```
public class HomeController : Controller
{
    public class FileManagerDirectoryContent1 : FileManagerDirectoryContent
    {
        public string RootName { get; set; }
    }
    public object AmazonS3FileOperations([FromBody] FileManagerDirectoryContent1 args)
    {
        this.operation.setRootName(args.RootName);
        ...
    }
    // uploads the file(s) into a specified path
    public IActionResult AmazonS3Upload(string path, IList<IFormFile> uploadFiles, string action, string data, string rootName)
    {
        this.operation.setRootName(rootName);
        ...
    }
    // downloads the selected file(s) and folder(s)
    public IActionResult AmazonS3Download(string downloadInput)
    {
        FileManagerDirectoryContent1 args = JsonConvert.  DeserializeObject<FileManagerDirectoryContent1>(downloadInput);
        this.operation.setRootName(args.RootName);
        return operation.Download(args.Path, args.Names);
    }
    // gets the image(s) from the given path
    public IActionResult AmazonS3GetImage(FileManagerDirectoryContent1 args)
    {
        this.operation.setRootName(args.RootName);
        return operation.GetImage(args.Path, args.Id, false, null, args.Data);
    }
}
```

Refer to the code snippet of `AmazonS3FileProvider.cs` page

```
public class AmazonS3FileProvider : IAmazonS3FileProviderBase
{
...
public string FolderName="";
...
    //customization to update the class variable with root folder name received from client side
    public void setRootName(string args)
    {
        FolderName = args;
    }
    //Define the root directory to the file manager
    public void GetBucketList()
    {
        ListingObjectsAsync("", "", false).Wait();
        //setting the root name with custom name attribute and reading the files/folders inside based on that root name
        if (FolderName != null && FolderName != "" && response.S3Objects.Where(x => x.Key.Contains(FolderName)).Select(x => x).ToArray().Length > 0
            && (response.S3Objects.Where(x => x.Key.Contains(FolderName)).Select(x => x).ToArray()).Where(y => y.Key.Split("/").Length == 2 && y.Key.Split("/")[1] == "").Select(y => y).ToArray().Length == 1)
        {
            //It display the particular folder as root folder. 
            RootName = (response.S3Objects.Where(x => x.Key.Contains(FolderName)).Select(x => x).ToArray()).Where(y => y.Key.Split("/").Length == 2 && y.Key.Split("/")[1] == "").Select(y => y).ToArray()[0].Key;
        }
        else
        {
            //Or else it display the first folder as root folder 
            RootName = response.S3Objects.First().Key;
        }
    }
}
```

