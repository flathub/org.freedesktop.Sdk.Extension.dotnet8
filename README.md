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
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/8.0.0-rc.1.23421.29/microsoft.aspnetcore.app.runtime.linux-arm64.8.0.0-rc.1.23421.29.nupkg",
    "sha512": "d802e547b4e495f7a6a720857b744e7b78d7d2626e3331935ba90e6c94aba86b8ed122c18306762ff8a4c5bf6739bfbdd332e0dffc58a9efa97336a88e51f2c3",
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
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/8.0.0-rc.1.23421.29/microsoft.aspnetcore.app.runtime.linux-x64.8.0.0-rc.1.23421.29.nupkg",
    "sha512": "d7c5380b6d1d1ff1d0370a9975cbba6ec08e2ec601d1d360b7d5aee9e5dfcfc7ed0a480a6c56ed76a3f4f83f2cac22eed6f9d897d3ac332e3dc4c96285ff21c9",
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
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/8.0.0-rc.1.23421.29/microsoft.netcore.app.runtime.linux-arm64.8.0.0-rc.1.23421.29.nupkg",
    "sha512": "31e228e5ae550a6f3e3a0fa92bfda7acc794ee3b1a6a49cf85864e1ce4a17276eb7cdee7c42cda717e74afa5836fabb5a1568503e3896cbf239555b5bd6bb166",
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
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/8.0.0-rc.1.23421.29/microsoft.netcore.app.runtime.linux-x64.8.0.0-rc.1.23421.29.nupkg",
    "sha512": "924e73fa83f5f8c418a789fdedd0ef0877637587ee1aff3571e318c3e9fa910c722d45df90d6cbceac5d29ffc10c35d2ffd805731df907328ef4688137462854",
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
