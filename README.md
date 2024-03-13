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
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/8.0.3/microsoft.aspnetcore.app.runtime.linux-arm64.8.0.3.nupkg",
    "sha512": "28b09764aed05b097bbf7e03af5f05ab27ec992fc1476296f8c8d7d8e0f1bb6597594000e95cb8a4787019d4bd289b2a53b5c9a52ed224a8da94cf1d4b2ed694",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm64.8.0.3.nupkg",
    "x-checker-data": {
        "type": "html",
        "url": "https://dotnetcli.blob.core.windows.net/dotnet/aspnetcore/Runtime/8.0/latest.version",
        "version-pattern": "^([\\d\\.a-z-]+)$",
        "url-template": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/$version/microsoft.aspnetcore.app.runtime.linux-arm64.$version.nupkg"
    }
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/8.0.3/microsoft.aspnetcore.app.runtime.linux-x64.8.0.3.nupkg",
    "sha512": "b4a7945b58402ad226e28a6e4a0f36aa770f72c27ad3a97aa2292f531ea581e7de9cc9d5e97550784e945250c0f9fa70d82338ce5e08834403b386d50455205f",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-x64.8.0.3.nupkg",
    "x-checker-data": {
        "type": "html",
        "url": "https://dotnetcli.blob.core.windows.net/dotnet/aspnetcore/Runtime/8.0/latest.version",
        "version-pattern": "^([\\d\\.a-z-]+)$",
        "url-template": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/$version/microsoft.aspnetcore.app.runtime.linux-x64.$version.nupkg"
    }
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/8.0.3/microsoft.netcore.app.runtime.linux-arm64.8.0.3.nupkg",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm64.8.0.3.nupkg",
    "x-checker-data": {
        "type": "html",
        "url": "https://dotnetcli.blob.core.windows.net/dotnet/aspnetcore/Runtime/8.0/latest.version",
        "version-pattern": "^([\\d\\.a-z-]+)$",
        "url-template": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/$version/microsoft.netcore.app.runtime.linux-arm64.$version.nupkg"
    }
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/8.0.3/microsoft.netcore.app.runtime.linux-x64.8.0.3.nupkg",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-x64.8.0.3.nupkg",
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
