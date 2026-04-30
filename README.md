# ASP.NET Core FileManager Pass JWT Token

**Repository Description**  
This repository contains an **ASP.NET Core sample** that demonstrates how to **pass a JWT token (authorization data) from the client to the server** when using the Syncfusion **Essential JS 2 File Manager** component.

The sample shows how File Manager events can be used to attach authorization information for read, upload, download, and image retrieval operations.

## Project Overview
The purpose of this project is to help developers understand how to secure File Manager operations in an ASP.NET Core application by sending JWT tokens or custom authorization values from the client side to the server side.

It demonstrates how different File Manager events can be leveraged to pass authentication context and dynamically control access to storage resources.

## Features
- Integration of Syncfusion **Essential JS 2 File Manager**
- Pass JWT or custom authorization values from client to server
- Secure **read**, **upload**, **download**, **rename**, **copy**, and **delete** operations
- Custom handling for image preview requests
- Server‑side file operation control based on authorization data

## Prerequisites
Ensure the following requirements are met before running this project:
- Visual Studio 2022 or Visual Studio Code  
- .NET SDK compatible with ASP.NET Core  
- Syncfusion Essential JS 2 packages  


# Installation

### Clone the Repository
Clone the repository and navigate to the project directory:

```bash
git clone https://github.com/SyncfusionExamples/blazor-filemanager-pass-jwt-token
cd blazor-filemanager-pass-jwt-token
```
### Restore and Build the Application
Restore NuGet packages:
```bash
dotnet restore
```
Build the application:
```bash
dotnet build
```
### Running the Application
After a successful build, run the application using:
```bash
dotnet run
```
The ASP.NET Core File Manager service starts and is ready to accept authorized client requests.

## Usage

### File Manager authorization header for read and upload operation
To send the authorization header data from client side to server side use the below FileManager events by setting the folder in the GetBucketList method.
| **File Operations** | **Events** |
| --- | --- |
| Read, Delete, Upload, Rename, Copy         | [`BeforeSend`](https://help.syncfusion.com/cr/aspnetcore-js2/Syncfusion.EJ2.FileManager.FileManager.html#Syncfusion_EJ2_FileManager_FileManager_BeforeSend) |
| GetImage      | [`BeforeImageLoad`](https://help.syncfusion.com/cr/aspnetcore-js2/Syncfusion.EJ2.FileManager.FileManager.html#Syncfusion_EJ2_FileManager_FileManager_BeforeImageLoad) |
| Download     | [`BeforeDownload`](https://help.syncfusion.com/cr/aspnetcore-js2/Syncfusion.EJ2.FileManager.FileManager.html#Syncfusion_EJ2_FileManager_FileManager_BeforeDownload) |
Find the changes in `Index.cshtml` page
```
<ejs-filemanager id="filemanager" view="@Syncfusion.EJ2.FileManager.ViewType.Details" beforeSend="beforeSend" beforeImageLoad="beforeImageLoad" beforeDownload="beforeDownload">
...
</ejs-filemanager>

<script>

function beforeSend(args) {
    //Ajax beforeSend event 
    args.ajaxSettings.beforeSend = function (args) {
        //Setting authorization header              
        args.httpRequest.setRequestHeader("Authorization", "RootFolder"); 
    }
}
function beforeImageLoad(args) {
    var rootFileName = "RootFolder";
    args.imageUrl = args.imageUrl + '&rootName=' + rootFileName;
}
function beforeDownload(args) {
    var rootFileName = "RootFolder" ;
    var includeCustomAttribute = args.data;
    includeCustomAttribute.rootName = rootFileName;
    args.data = includeCustomAttribute;
}
</script>
```
Find the changes in `HomeController.cs` page
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
Find the changes in `AmazonS3FileProvider.cs` page
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

## Configuration
Authorization handling is configured through:
- Client‑side File Manager events
- ASP.NET Core controller methods
- Custom file provider logic (for example, updating the root folder dynamically)

## Documentation
- General Syncfusion documentation:
https://help.syncfusion.com/
- ASP.NET Core Introduction:
https://ej2.syncfusion.com/aspnetcore/documentation/introduction
- ASP.NET Core File Manager – Getting Started:
https://ej2.syncfusion.com/aspnetcore/documentation/file-manager/getting-started

## Additional Resources
- Syncfusion File Manager product overview:
https://www.syncfusion.com/javascript-ui-controls/js-file-manager

## Troubleshooting
- Ensure authorization data is sent through the correct File Manager events.
- Verify server‑side controller methods are receiving expected parameters.
- Rebuild the solution if configuration changes are not reflected.
- Check application logs for authentication or routing errors.

## Support
For detailed API references, security customization guidance, and advanced File Manager scenarios, refer to the Syncfusion ASP.NET Core File Manager documentation links above.