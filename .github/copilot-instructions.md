# Copilot Instructions

Hard fork of the OPC Foundation OPC COM Wrapper: bridges classic OPC (COM/DCOM) and OPC UA in both directions. Windows-only, .NET Framework 4.8 (old-style csproj) plus native C++ pieces.

## Build / test

- Open `COMtoUANx.sln` in Visual Studio (or `msbuild COMtoUANx.sln` from a VS developer prompt). NuGet restore uses `Nuget.Config`; UA stack comes from `OPCFoundation.NetStandard.Opc.Ua.*` packages (Core, Client, Server, Configuration).
- There are no test projects and no lint configuration; no single-test command exists.
- Regenerating the UA information model: run `ComIOP\Common\Common\BuildDesign.bat` (needs `Opc.Ua.ModelCompiler.exe` on PATH). It compiles `ModelDesign.xml` / `ModelDesign.csv` into the generated `Opc.Ua.Com.*.cs`, `*.NodeSet.xml` and `*.NodeIds.csv`. Edit the design files, never the generated outputs.

## Architecture

Projects:
- `ComIOP\Common\UA COM Interop Library.csproj` – the core library, namespaced `Opc.Ua.Com...`.
- `ComIOP\Wrapper\ServerWrapper\COMtoUANx.csproj` – WinExe host (`Opc.Ua.ComServerWrapper`) referencing the library and `Samples\ServerControls.Net4`.
- `ComIOP\Wrapper\Common` and `ComIOP\Wrapper\Include` – native C++ COM helper code (`COpc*` classes) and headers.

The library has two directions, split by folder under `ComIOP\Common`:
- **`Client\`** – UA server that wraps a *COM server* (COM -> UA). Per-spec subfolders: `Da` (Data Access), `Hda` (Historical -> UA HA), `Ae` (Alarms & Events -> UA A&C). Each has a `Com<X>Client`, `Com<X>ClientManager`, `Com<X>ClientNodeManager` plus node state classes and `ParsedNodeId` parsers for mapping UA NodeIds to COM item IDs. `ComWrapperServer` / `ComClientManager` tie them together.
- **`Server\`** – COM server that wraps a *UA server* (UA -> COM proxy), again `Da`, `Hda`, `Ae`, plus `Ae2`, an alternate A&E implementation. `ComProxy` / `ComProxyConfiguration` / `ComNamespaceMapper` are shared; `Ae` also contains C++ (`COpcUaAe*`) sources.
- **`Rcw\`** – hand-maintained COM interop (runtime callable wrapper) definitions for OPC DA/HDA/AE/Security interfaces.
- **`Common\`** – shared utils (`ComUtils`, `Namespaces`) and the generated model (`Opc.Ua.Com.*`).
- **`Config\`, `Proxy\`** – configuration classes and older proxy helpers.

## Conventions

- Files and classes follow the OPC Foundation style (`Com`/`Da`/`Hda`/`Ae` prefixes); mirror the existing Da/Hda/Ae structure when adding features so each spec stays parallel.
- Many classes exist in similarly named copies across `Client`, `Server` and `Proxy` (e.g. `ComDaGroup`, `DaElement`, `DaValue`); verify which namespace/folder you are editing.
- `.gitattributes` governs line endings; keep existing file endings.
- Contributions need agreement to the OPC Foundation CLA (see `CONTRIBUTING.md`).
