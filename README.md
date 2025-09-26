# TileViewer Managed Plugin

This repository hosts a .NET 9 AOT-ready managed plugin skeleton for [TileViewer](https://github.com/YuriSizuku/TileViewer).
The project replaces the native `tile_decoder_t` callbacks with safe managed abstractions while keeping method parity so it can
be used as a starting point for C# implementations.

## Projects

| Project | Description |
| --- | --- |
| TileViewer.Plugin.Dotnet | Class library targeting `net9.0` with AOT publishing enabled. |

## Getting started

1. Ensure the .NET 9 SDK is available (this repository uses `dotnet-sdk-9.0.305`).
2. Restore and build the solution:

   ```bash
   dotnet build
   ```

3. Reference `TileViewer.Plugin.Dotnet` from a TileViewer host application and instantiate `ManagedTileViewerPlugin` to access the managed API.

## Key features

- **Managed API surface** – `ITileViewerPlugin` exposes safe equivalents of the native callbacks (`DecodeOpen`, `DecodeClose`, `DecodeOne`, `DecodeAll`, `DecodePre`, `DecodePost`).
- **JSON configuration model** – `PluginConfiguration` and `PluginOption` map directly to the JSON expected by TileViewer. Serialization uses source generation (`System.Text.Json`) so it works in Native AOT scenarios.
- **No unsafe code** – All binary decoding uses `Span<T>`/`Memory<T>` abstractions.
- **Extensible decoder pipeline** – Implement `IManagedTileDecoder` to provide custom decoding logic. `PagedTwoBitTileDecoder` demonstrates how to expand packed pixels with `DecodeAll` while keeping the implementation entirely managed.
- **Configuration logging** – When the host updates plugin options, the managed bridge records the changes with `Log` so you can observe configuration churn without attaching a debugger.

## DecodeAll 2-bit demo (Ryu-Koku font.bin decode)

`ManagedTileViewerPlugin` ships with `MainTileDecoder`, a managed decoder that unpacks four logical pages encoded inside each byte. The plugin exposes a `page` option with the values `0`–`3`; TileViewer forwards the selected value back to the managed decoder via `DecodeAll` so only one page is expanded per run. Each packed 2-bit value is treated as an index into `cp4 = { 0x00, 0xFF, 0xFF, 0xFF }`, providing the pixel alpha while leaving RGB fixed at `0xFF` to emphasize the alpha masking effect.

## How to use 
Create a new class implementing IConfigurableTileDecoder. If you don’t know how to start, see MainTileDecoder (demo).
```c#
dll/so ==> TileDecoderNativeExports(unsafe) ==> ManagedTileViewerPlugin ==> IConfigurableTileDecoder
```

## Publishing for Native AOT

When publishing, enable AOT with:

```bash
dotnet publish src/TileViewer.ManagedPlugin/TileViewer.ManagedPlugin.csproj -c Release -r win-x64 -p:PublishAot=true
```

The project file already sets `PublishAot` to `true` so the property only needs to be reaffirmed when a RID is specified.
