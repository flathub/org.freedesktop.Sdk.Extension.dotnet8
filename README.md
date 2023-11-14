# .NET 8 SDK extension

## How to use
You need to add following lines to flatpak manifest:

```json
"sdk-extensions": [
    "org.freedesktop.Sdk.Extension.dotnet8"
],
"build-options": {
    "append-path": "/usr/lib/sdk/dotnet8/bin",
    "append-ld-library-path": "/usr/lib/sdk/dotnet8/lib",
    "env": {
        "PKG_CONFIG_PATH": "/app/lib/pkgconfig:/app/share/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig:/usr/lib/sdk/dotnet8/lib/pkgconfig"
    }
},
```

###  Scripts
* `install.sh` - copies dotnet runtime to package.
* `install-sdk.sh` - copies dotnet SDK to package.

### Publishing project

```json
"build-commands": [
    "install.sh",
    "dotnet publish -c Release YourProject.csproj",
    "cp -r --remove-destination /run/build/YourProject/bin/Release/net8.0/publish/ /app/bin/",
]
```

### Using nuget packages
If you want to use nuget packages it is recommended to use the [Flatpak .NET Generator](https://github.com/flatpak/flatpak-builder-tools/tree/master/dotnet) tool. It generates sources file that can be included in manifest.

```json
"build-commands": [
    "install.sh",
    "dotnet publish -c Release --source ./nuget-sources YourProject.csproj",
    "cp -r --remove-destination /run/build/YourProject/bin/Release/net8.0/publish/ /app/bin/"
],
"sources": [
    "sources.json",
    "..."
]
```

### Publishing self contained app and trimmed binaries
.NET 8 gives option to include runtime in published application and trim their binaries. This allows you to significantly reduce the size of the package and get rid of `/usr/lib/sdk/dotnet8/bin/install.sh`. 

First you need to have following lines in `sources.json`. These packages are needed to build a project for specific runtime. 

```json
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/8.0.0/microsoft.aspnetcore.app.runtime.linux-arm64.8.0.0.nupkg",
    "sha512": "670fde6af5e976062ceded5dbabcfb772e292662b2a583665807ca7aa81243b41c054661cfad2c7c928db3f6d87a1eefb2ef26d2beb5b8e8c278b4ef0e6a310d",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm64.8.0.0.nupkg",
    "x-checker-data": {
        "type": "html",
        "url": "https://dotnetcli.blob.core.windows.net/dotnet/aspnetcore/Runtime/8.0/latest.version",
        "version-pattern": "^([\\d\\.a-z-]+)$",
        "url-template": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/$version/microsoft.aspnetcore.app.runtime.linux-arm64.$version.nupkg"
    }
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/8.0.0/microsoft.aspnetcore.app.runtime.linux-x64.8.0.0.nupkg",
    "sha512": "741c423cd7f1e919f292f049fe3edf42a086c44529230617a7577929804765e3500174da79e09d3027276059b143df400f934e0c521d8682e10256f13c10dc25",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-x64.8.0.0.nupkg",
    "x-checker-data": {
        "type": "html",
        "url": "https://dotnetcli.blob.core.windows.net/dotnet/aspnetcore/Runtime/8.0/latest.version",
        "version-pattern": "^([\\d\\.a-z-]+)$",
        "url-template": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/$version/microsoft.aspnetcore.app.runtime.linux-x64.$version.nupkg"
    }
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/8.0.0/microsoft.netcore.app.runtime.linux-arm64.8.0.0.nupkg",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm64.8.0.0.nupkg",
    "x-checker-data": {
        "type": "html",
        "url": "https://dotnetcli.blob.core.windows.net/dotnet/aspnetcore/Runtime/8.0/latest.version",
        "version-pattern": "^([\\d\\.a-z-]+)$",
        "url-template": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/$version/microsoft.netcore.app.runtime.linux-arm64.$version.nupkg"
    }
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/8.0.0/microsoft.netcore.app.runtime.linux-x64.8.0.0.nupkg",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-x64.8.0.0.nupkg",
    "x-checker-data": {
        "type": "html",
        "url": "https://dotnetcli.blob.core.windows.net/dotnet/aspnetcore/Runtime/8.0/latest.version",
        "version-pattern": "^([\\d\\.a-z-]+)$",
        "url-template": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/$version/microsoft.netcore.app.runtime.linux-x64.$version.nupkg"
    }
},
```

Then add build options:

```json
"build-options": {
    "arch": {
        "aarch64": {
            "env" : {
                "RUNTIME": "linux-arm64"
            }
        },
        "x86_64": {
            "env" : {
                "RUNTIME": "linux-x64"
            }
        }
    }
},
"build-commands": [
    "mkdir -p /app/bin",
    "dotnet publish -c Release --source ./nuget-sources YourProject.csproj --runtime $RUNTIME --self-contained true",
    "cp -r --remove-destination /run/build/YourProject/bin/Release/net8.0/$RUNTIME/publish/* /app/bin/",
],
```
