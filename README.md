# dotnet-how-to-debug-source-generator-vs2022
Step by step guide on how to debug a C# SourceGenerator in Visual Studio 2022

> Valid as of 1st March 2022 with Visual Studio 2022 Version 17.1.0.

Source: [Github | Roslyn SDK | Support source generator debugging in the new Launch Profiles UI
#850](https://github.com/dotnet/roslyn-sdk/issues/850)

## Prerequisites
The [.NET Compiler Platform SDK](https://docs.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/) must be installed.

## Solution structure
1. Add a .NET Standard 2.0 class library for your SourceGenerator.
2. Set the SourceGenerator projet as Startup project.
3. Edit the SourceGenerator project and replace all with:
```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <Nullable>enable</Nullable>
    <EmitCompilerGeneratedFiles>true</EmitCompilerGeneratedFiles>
    <CompilerGeneratedFilesOutputPath>Generated</CompilerGeneratedFilesOutputPath>
    <IsRoslynComponent>true</IsRoslynComponent>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.2.0" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.3">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>

  <ItemGroup>
    <None Include="$(OutputPath)\$(AssemblyName).dll" Pack="true" PackagePath="analyzers/dotnet/cs" Visible="false" />
  </ItemGroup>

</Project>
```

4. Save it.
5. Add a Console Application project with some examples of code you would like to test the SourceGenerator with.
6. Edit the Console Application porject and add a reference to the SourceGenerator this way:
```xml
  <ItemGroup>
    <ProjectReference
      Include="..\MySourceGenerator\MySourceGenerator.csproj"
      OutputItemType="Analyzer"
      ReferenceOutputAssembly="false"/>
  </ItemGroup>
```

7. Save it.

## Debug Configuration
1. Right click on the SourceGenerator project
2. Click `Properties`.
2. Click `Debug`.
3. Click `Open debug launch profiles UI`.
4. Click on `Delete` to delete the profile shown.
5. Click on `Add`
6. Select `Roslyn component`.
7. In `Target project` select the Console Application project.
8. Close the UI.
9. Restart Visual Studio 2022.
10. In the debug profiles dropdown next to the `Play` button, select your SourceGenerator project.
11. Put a break point in your SourceGenerator to make sure the debugger stops.
12. Click `Play`.

## Notes
Every time you change your source generator code, you will need to restart Visual Studio, otherwise Rebuilding the target project will not use the new version. This has something to do with Visual Studio caching.

Sources:
* [MIcrosoft Docs | Source Generators](https://docs.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview)
* [Steve Talks Code | Debugging C# Source Generators with Visual Studio 2019 16.10](http://stevetalkscode.co.uk/debug-source-generators-with-vs2019-1610)
