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
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/8.0.5/microsoft.aspnetcore.app.runtime.linux-arm64.8.0.5.nupkg",
    "sha512": "287fa013fc4d560ca28a67e68baec62fe60f95edcac7eba9be6d51e6b81152bae96555ebc8f2570cad1d2696e34ec2dfa72ba0fab1fdc559bb7cf2f74dd9a5cd",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm64.8.0.5.nupkg",
    "x-checker-data": {
        "type": "html",
        "url": "https://dotnetcli.blob.core.windows.net/dotnet/aspnetcore/Runtime/8.0/latest.version",
        "version-pattern": "^([\\d\\.a-z-]+)$",
        "url-template": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/$version/microsoft.aspnetcore.app.runtime.linux-arm64.$version.nupkg"
    }
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/8.0.5/microsoft.aspnetcore.app.runtime.linux-x64.8.0.5.nupkg",
    "sha512": "e53840b17723dd593f0daffb9fe549a1d924df1206973c8b3e26e48ab0261822813f745d9704303b69f78e2ad319d924347a35de6ee0c196317bce82df2e91f0",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-x64.8.0.5.nupkg",
    "x-checker-data": {
        "type": "html",
        "url": "https://dotnetcli.blob.core.windows.net/dotnet/aspnetcore/Runtime/8.0/latest.version",
        "version-pattern": "^([\\d\\.a-z-]+)$",
        "url-template": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/$version/microsoft.aspnetcore.app.runtime.linux-x64.$version.nupkg"
    }
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/8.0.5/microsoft.netcore.app.runtime.linux-arm64.8.0.5.nupkg",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm64.8.0.5.nupkg",
    "x-checker-data": {
        "type": "html",
        "url": "https://dotnetcli.blob.core.windows.net/dotnet/aspnetcore/Runtime/8.0/latest.version",
        "version-pattern": "^([\\d\\.a-z-]+)$",
        "url-template": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/$version/microsoft.netcore.app.runtime.linux-arm64.$version.nupkg"
    }
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/8.0.5/microsoft.netcore.app.runtime.linux-x64.8.0.5.nupkg",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-x64.8.0.5.nupkg",
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
