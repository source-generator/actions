# Actions for use in the source-generator organisation

Reusable GitHub Actions and workflows for building, testing, and publishing .NET NuGet packages.

## Composite Actions

### `build-dotnet`

Sets up .NET SDK and builds a .NET project or solution.

```yaml
- uses: source-generator/actions/build-dotnet@main
  with:
    dotnet-version: "10.0.x"   # Optional, default: 10.0.x
    project-path: "src/MyApp"  # Optional, default: solution root
    configuration: "Release"   # Optional, default: Release
    extra-args: ""             # Optional extra dotnet build args
```

### `test-dotnet`

Runs tests and uploads results as artifacts.

```yaml
- uses: source-generator/actions/test-dotnet@main
  with:
    dotnet-version: "10.0.x"      # Optional, default: 10.0.x
    project-path: "tests/MyApp"  # Optional, default: solution root
    configuration: "Release"     # Optional, default: Release
    collect-coverage: "true"     # Optional, default: true
    results-directory: "TestResults"  # Optional
    extra-args: ""               # Optional extra dotnet test args
```

### `publish-nuget`

Packs and publishes a .NET project as a NuGet package.

```yaml
- uses: source-generator/actions/publish-nuget@main
  with:
    project-path: "src/MyLib"         # Required
    nuget-api-key: ${{ secrets.NUGET_API_KEY }}  # Required
    dotnet-version: "10.0.x"           # Optional, default: 10.0.x
    configuration: "Release"          # Optional, default: Release
    package-version: "1.0.0"          # Optional, uses project version if empty
    nuget-source: "https://api.nuget.org/v3/index.json"  # Optional
    output-directory: "nupkg"         # Optional, default: nupkg
    skip-duplicate: "true"            # Optional, default: true
    extra-pack-args: ""               # Optional extra dotnet pack args
```

## Reusable Workflows

### Build and Test

Builds and tests a .NET project in separate jobs.

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    uses: source-generator/actions/.github/workflows/dotnet-build-test.yml@main
    with:
      dotnet-version: "10.0.x"
      project-path: "src/MyApp.sln"
      configuration: "Release"
      collect-coverage: true
```

### Publish NuGet Package

Packs and publishes a NuGet package.

```yaml
# .github/workflows/publish.yml
name: Publish

on:
  release:
    types: [published]

jobs:
  publish:
    uses: source-generator/actions/.github/workflows/dotnet-publish-nuget.yml@main
    with:
      project-path: "src/MyLib/MyLib.csproj"
      package-version: ${{ github.event.release.tag_name }}
    secrets:
      nuget-api-key: ${{ secrets.NUGET_API_KEY }}
```

### Full CI/CD Pipeline

Combined workflow that builds, tests, and optionally publishes.

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  release:
    types: [published]

jobs:
  ci-cd:
    uses: source-generator/actions/.github/workflows/dotnet-ci-cd.yml@main
    with:
      dotnet-version: "10.0.x"
      project-path: "src/MyApp.sln"
      publish-project-path: "src/MyLib/MyLib.csproj"
      publish: ${{ github.event_name == 'release' }}
      package-version: ${{ github.event.release.tag_name || '' }}
    secrets:
      nuget-api-key: ${{ secrets.NUGET_API_KEY }}
```

## Inputs Reference

| Input | Build | Test | Publish | CI/CD | Description |
|-------|-------|------|---------|-------|-------------|
| `dotnet-version` | ✅ | ✅ | ✅ | ✅ | .NET SDK version |
| `project-path` | ✅ | ✅ | ✅ | ✅ | Project/solution path |
| `configuration` | ✅ | ✅ | ✅ | ✅ | Build configuration |
| `runs-on` | — | — | ✅ | ✅ | Runner to use |
| `collect-coverage` | — | ✅ | — | ✅ | Collect code coverage |
| `package-version` | — | — | ✅ | ✅ | NuGet package version |
| `nuget-source` | — | — | ✅ | ✅ | NuGet feed URL |
| `nuget-api-key` | — | — | ✅ | ✅ | NuGet API key (secret) |
| `publish` | — | — | — | ✅ | Enable publishing |
| `skip-duplicate` | — | — | ✅ | ✅ | Skip existing versions |
| `publish-project-path` | — | — | — | ✅ | Project path for publishing |