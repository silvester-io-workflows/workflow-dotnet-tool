Reusable workflow for packaging a .NET CLI tool as a NuGet package and publishing a container image that includes the published binaries.

## Usage

Add a workflow in the consuming repository that triggers when you want to publish and calls this reusable workflow. The reusable workflow restores/tests on every run, packs/publishes NuGet packages only on tag refs, and always publishes a container image with the standard tags.

```yaml
name: Publish CLI Tool

on:
  push:
    branches: ["main"]
    tags: ["v*"]
  workflow_dispatch: {}

permissions:
  contents: read
  packages: write

jobs:
  publish:
    uses: silvester-io-workflows/workflow-dotnet-tool/.github/workflows/publish-dotnet-tool.yaml@v1
    with:
      tool_project: ./source/Assemblies/Youtubarr.Hosts.Cli/Youtubarr.Hosts.Cli.csproj
      solution_path: ./source/Youtubarr.slnx
      runsettings_path: ./source/Tests/Tests.Integration.Hosts.Cli/.runsettings.ci
      dockerfile: ./source/Assemblies/Youtubarr.Hosts.Cli/containerfile
      context: .
      platforms: linux/amd64
    secrets: inherit
```

### Notes
- The container build context must include `publish_output_dir` (default `./artifacts/publish`) because the containerfile copies the published binaries from that location.
- If `dockerfile` is omitted, it defaults to `containerfile` in the tool project directory.

### Inputs
- `tool_project` (required): Path to the .NET tool `.csproj`.
- `solution_path` (optional): Path to the solution to restore/test.
- `runsettings_path` (required): Path to the `.runsettings` file used during tests.
- `configuration` (optional): Build configuration. Defaults to `Release`.
- `dotnet_version` (optional): .NET SDK version. Defaults to `10.0.x`.
- `package_source` (optional): NuGet source to push to. Defaults to the caller's GitHub Packages feed.
- `output_dir` (optional): Directory for packed tool packages. Defaults to `./artifacts/packages`.
- `publish_output_dir` (optional): Directory for published binaries. Defaults to `./artifacts/publish`.
- `restore_source` (optional): NuGet source used for internal packages. Defaults to `https://nuget.pkg.github.com/silvester-io-libraries/index.json`.
- `publish_runtime` (optional): Runtime identifier for publish output. Defaults to `linux-x64`.
- `self_contained` (optional): Whether to publish as self-contained. Defaults to `false`.
- `use_app_host` (optional): Whether to generate an app host for publish output. Defaults to `false`.
- `registry` (optional): Registry hostname. Defaults to `ghcr.io`.
- `image_name` (optional): Image name to publish. Defaults to the caller repo.
- `context` (optional): Container build context. Defaults to the repository root.
- `dockerfile` (optional): Path to the containerfile. Defaults to `containerfile` next to the tool project.
- `platforms` (optional): Target platforms. Defaults to `linux/amd64`.
