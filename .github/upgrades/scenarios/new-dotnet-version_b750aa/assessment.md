# Projects and dependencies analysis

This document provides a comprehensive overview of the projects and their dependencies in the context of upgrading to .NETCoreApp,Version=v10.0.

## Table of Contents

- [Executive Summary](#executive-Summary)
  - [Highlevel Metrics](#highlevel-metrics)
  - [Projects Compatibility](#projects-compatibility)
  - [Package Compatibility](#package-compatibility)
  - [API Compatibility](#api-compatibility)
- [Aggregate NuGet packages details](#aggregate-nuget-packages-details)
- [Top API Migration Challenges](#top-api-migration-challenges)
  - [Technologies and Features](#technologies-and-features)
  - [Most Frequent API Issues](#most-frequent-api-issues)
- [Projects Relationship Graph](#projects-relationship-graph)
- [Project Details](#project-details)

  - [FortiCertSync\FortiCertSync.csproj](#forticertsyncforticertsynccsproj)


## Executive Summary

### Highlevel Metrics

| Metric | Count | Status |
| :--- | :---: | :--- |
| Total Projects | 1 | All require upgrade |
| Total NuGet Packages | 2 | 1 need upgrade |
| Total Code Files | 5 |  |
| Total Code Files with Incidents | 2 |  |
| Total Lines of Code | 764 |  |
| Total Number of Issues | 39 |  |
| Estimated LOC to modify | 37+ | at least 4.8% of codebase |

### Projects Compatibility

| Project | Target Framework | Difficulty | Package Issues | API Issues | Est. LOC Impact | Description |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| [FortiCertSync\FortiCertSync.csproj](#forticertsyncforticertsynccsproj) | net8.0 | 🟢 Low | 1 | 37 | 37+ | DotNetCoreApp, Sdk Style = True |

### Package Compatibility

| Status | Count | Percentage |
| :--- | :---: | :---: |
| ✅ Compatible | 1 | 50.0% |
| ⚠️ Incompatible | 0 | 0.0% |
| 🔄 Upgrade Recommended | 1 | 50.0% |
| ***Total NuGet Packages*** | ***2*** | ***100%*** |

### API Compatibility

| Category | Count | Impact |
| :--- | :---: | :--- |
| 🔴 Binary Incompatible | 0 | High - Require code changes |
| 🟡 Source Incompatible | 12 | Medium - Needs re-compilation and potential conflicting API error fixing |
| 🔵 Behavioral change | 25 | Low - Behavioral changes that may require testing at runtime |
| ✅ Compatible | 1234 |  |
| ***Total APIs Analyzed*** | ***1271*** |  |

## Aggregate NuGet packages details

| Package | Current Version | Suggested Version | Projects | Description |
| :--- | :---: | :---: | :--- | :--- |
| Microsoft.SourceLink.GitHub | 8.0.0 |  | [FortiCertSync.csproj](#forticertsyncforticertsynccsproj) | ✅Compatible |
| System.Security.Cryptography.ProtectedData | 9.0.8 | 10.0.5 | [FortiCertSync.csproj](#forticertsyncforticertsynccsproj) | NuGet package upgrade is recommended |

## Top API Migration Challenges

### Technologies and Features

| Technology | Issues | Percentage | Migration Path |
| :--- | :---: | :---: | :--- |

### Most Frequent API Issues

| API | Count | Percentage | Category |
| :--- | :---: | :---: | :--- |
| T:System.Net.Http.HttpContent | 10 | 27.0% | Behavioral Change |
| T:System.Text.Json.JsonDocument | 8 | 21.6% | Behavioral Change |
| T:System.Uri | 5 | 13.5% | Behavioral Change |
| T:System.Security.Cryptography.DataProtectionScope | 4 | 10.8% | Source Incompatible |
| F:System.Security.Cryptography.DataProtectionScope.CurrentUser | 2 | 5.4% | Source Incompatible |
| T:System.Security.Cryptography.ProtectedData | 2 | 5.4% | Source Incompatible |
| M:System.Security.Cryptography.X509Certificates.X509Certificate2Collection.Import(System.Byte[],System.String,System.Security.Cryptography.X509Certificates.X509KeyStorageFlags) | 1 | 2.7% | Behavioral Change |
| M:System.Security.Cryptography.X509Certificates.X500DistinguishedName.#ctor(System.String) | 1 | 2.7% | Behavioral Change |
| M:System.Security.Cryptography.X509Certificates.X509Certificate2.#ctor(System.Byte[]) | 1 | 2.7% | Source Incompatible |
| M:System.Security.Cryptography.ProtectedData.Protect(System.Byte[],System.Byte[],System.Security.Cryptography.DataProtectionScope) | 1 | 2.7% | Source Incompatible |
| M:System.TimeSpan.FromSeconds(System.Double) | 1 | 2.7% | Source Incompatible |
| M:System.Security.Cryptography.ProtectedData.Unprotect(System.Byte[],System.Byte[],System.Security.Cryptography.DataProtectionScope) | 1 | 2.7% | Source Incompatible |

## Projects Relationship Graph

Legend:
📦 SDK-style project
⚙️ Classic project

```mermaid
flowchart LR
    P1["<b>📦&nbsp;FortiCertSync.csproj</b><br/><small>net8.0</small>"]
    click P1 "#forticertsyncforticertsynccsproj"

```

## Project Details

<a id="forticertsyncforticertsynccsproj"></a>
### FortiCertSync\FortiCertSync.csproj

#### Project Info

- **Current Target Framework:** net8.0
- **Proposed Target Framework:** net10.0
- **SDK-style**: True
- **Project Kind:** DotNetCoreApp
- **Dependencies**: 0
- **Dependants**: 0
- **Number of Files**: 6
- **Number of Files with Incidents**: 2
- **Lines of Code**: 764
- **Estimated LOC to modify**: 37+ (at least 4.8% of the project)

#### Dependency Graph

Legend:
📦 SDK-style project
⚙️ Classic project

```mermaid
flowchart TB
    subgraph current["FortiCertSync.csproj"]
        MAIN["<b>📦&nbsp;FortiCertSync.csproj</b><br/><small>net8.0</small>"]
        click MAIN "#forticertsyncforticertsynccsproj"
    end

```

### API Compatibility

| Category | Count | Impact |
| :--- | :---: | :--- |
| 🔴 Binary Incompatible | 0 | High - Require code changes |
| 🟡 Source Incompatible | 12 | Medium - Needs re-compilation and potential conflicting API error fixing |
| 🔵 Behavioral change | 25 | Low - Behavioral changes that may require testing at runtime |
| ✅ Compatible | 1234 |  |
| ***Total APIs Analyzed*** | ***1271*** |  |

