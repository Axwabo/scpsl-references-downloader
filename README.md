# SCP:SL References Downloader

A simple GitHub Action to download reference DLLs for SCP: Secret Laboratory plugin development.

> [!NOTE]
> This action does **not** publicize assemblies.
> It's recommended to use the [MSBuild publicizer](https://github.com/BepInEx/BepInEx.AssemblyPublicizer)

# Usage

To download assemblies into a directory in the runner's temporary directory, simply add the following step:

```yaml
- name: Download SCP:SL References
  uses: Axwabo/scpsl-references-downloader@v1
```

This will also set the `ReferencePath` environment variable to the directory.

> [!TIP]
> See also: [outputs](#outputs) and [full usage](#full-workflow-steps)

To locate downloaded assemblies automatically, add the following line in a `PropertyGroup`
of your `.csproj` or `Directory.Build.targets` file:
`<AssemblySearchPaths>$(ReferencePath);$(AssemblySearchPaths)</AssemblySearchPaths>`

> [!TIP]
> Create a user-specific props file that specifies the ReferencePath on your local machine.
> [Example](https://github.com/Axwabo/SCPSL-Helpers/blob/main/Directory.Build.targets)

> [!IMPORTANT]
> Don't commit user-specific files to your repository.
> To create the recommended .gitignore file, run `dotnet new gitignore`

# Inputs

## branch

Default value: `public`

The steam branch to download references from, e.g. `experimental`

## directory

Default value: `${{ runner.temp }}/SCPSL-References`

The directory to place the DLLs into.
By default, it creates a directory in the runner's temporary directory.

## specify-reference-path

Default value: `true`

Whether to set the `ReferencePath` environment variable to the value of `directory`
If true, the environment variable will be available during the entire job.

> [!CAUTION]
> The ReferencePath has to be absolute (relative to the built project) for MSBuild to recognize it.
> Hence why `runner.temp` is included in the `directory` input by default.

# Outputs

The `target` output of the workflow specifies the directory where references were placed.

# Full Workflow Steps

> [!NOTE]
> GitHub runners have the .NET 8 and .NET 9 SDKs installed by default, you can skip the first step.
> [Linux info](https://github.com/actions/runner-images/blob/main/images/ubuntu/Ubuntu2404-Readme.md#net-tools)
> [Windows info](https://github.com/actions/runner-images/blob/main/images/windows/Windows2025-Readme.md#net-core-tools)

```yaml
- uses: actions/checkout@v5

- uses: actions/setup-dotnet@v5
  with:
    dotnet-version: 9.0.x

- name: Download SCP:SL References
  uses: Axwabo/scpsl-references-downloader@v1
  id: refs
  with:
    directory: ${{ github.workspace }}/MyReferencesDirectory
    branch: public
    specify-reference-path: false

- name: Build
  run: dotnet build
  env:
    ReferencePath: ${{ steps.refs.outputs.target }}

# Upload artifacts and/or create release
```
