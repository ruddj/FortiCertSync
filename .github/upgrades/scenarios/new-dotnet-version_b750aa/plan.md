# .NET 10 Upgrade Plan — FortiCertSync

## Table of Contents

- [Executive Summary](#executive-summary)
- [Migration Strategy](#migration-strategy)
- [Detailed Dependency Analysis](#detailed-dependency-analysis)
- [Project-by-Project Plans](#project-by-project-plans)
- [Package Update Reference](#package-update-reference)
- [Breaking Changes Catalog](#breaking-changes-catalog)
- [Testing & Validation Strategy](#testing--validation-strategy)
- [Risk Management](#risk-management)
- [Complexity & Effort Assessment](#complexity--effort-assessment)
- [Source Control Strategy](#source-control-strategy)
- [Success Criteria](#success-criteria)

---

## Executive Summary

### Scenario

Upgrade `FortiCertSync` from `.NET 8.0` to `.NET 10.0`. FortiCertSync is a Windows service that synchronises TLS certificates issued by Windows (ACME/Let's Encrypt) into a FortiGate appliance over REST API.

### Scope

| Item | Detail |
|------|--------|
| Projects | 1 (`FortiCertSync\FortiCertSync.csproj`) |
| Current Framework | `net8.0` |
| Target Framework | `net10.0` |
| Total LOC | 764 |
| Files with Issues | 2 (`FortiClient.cs`, `WindowsCertService.cs`) |
| NuGet Packages | 2 (1 update required) |
| Source-Incompatible APIs | 12 (require code fixes) |
| Behavioral-Change APIs | 25 (require test validation) |
| Estimated LOC Impact | 37+ (~4.8%) |

### Selected Strategy

**All-At-Once Strategy** — single project, single coordinated upgrade operation.

**Rationale:**
- Only 1 project — no dependency sequencing needed
- Small codebase (764 LOC) — low blast radius
- All packages have target versions available
- Source-incompatible changes are localised to 2 files
- Low overall difficulty (🟢 per assessment)

### Complexity Classification

**Simple** — 1 project, depth 0, no security vulnerabilities, no circular dependencies.

### Critical Issues

No security vulnerabilities detected. All issues are API compatibility or behavioral changes addressable during compilation and targeted code fixes.

---

## Migration Strategy

### Approach: All-At-Once

All changes are applied in a single coordinated operation:

1. Update `TargetFramework` in the project file
2. Update the one NuGet package requiring an upgrade
3. Restore dependencies
4. Build the solution and fix all compilation errors (source-incompatible APIs)
5. Verify the solution builds with 0 errors

### Ordering Principles

Single project — no dependency ordering required. Within the upgrade task:

- Project file change comes first (framework drives package compatibility)
- Package update follows immediately
- Compilation errors surface after restore + build
- Behavioral-change items noted for runtime validation only (no compile-time fix needed)

### Parallel vs Sequential

Not applicable — single project. All operations are strictly sequential within one atomic task.

---

## Detailed Dependency Analysis

### Dependency Graph

```
FortiCertSync.csproj   (net8.0 → net10.0)
  └── [no project dependencies]
```

### Project Groupings

**Phase 1 (only phase) — Atomic Upgrade:**
- `FortiCertSync\FortiCertSync.csproj`

### Critical Path

`FortiCertSync.csproj` is both leaf and root — it is the entire critical path.

### Circular Dependencies

None.

---

## Project-by-Project Plans

### Project: `FortiCertSync\FortiCertSync.csproj`

#### Current State

| Property | Value |
|----------|-------|
| Target Framework | `net8.0` |
| Project Kind | DotNetCoreApp (SDK-style, AOT-enabled) |
| Output Type | Executable (`win-x64`) |
| Lines of Code | 764 |
| Project Dependencies | 0 |
| NuGet Packages | 2 |
| Risk Level | 🟢 Low |

#### Target State

| Property | Value |
|----------|-------|
| Target Framework | `net10.0` |
| Updated Packages | 1 (`System.Security.Cryptography.ProtectedData` → `10.0.5`) |
| Source-Incompatible APIs to Fix | 12 occurrences across 2 files |

#### Migration Steps

**Step 1 — Update Target Framework**

In `FortiCertSync\FortiCertSync.csproj`, change:

```xml
<TargetFramework>net8.0</TargetFramework>
```
to:
```xml
<TargetFramework>net10.0</TargetFramework>
```

**Step 2 — Update NuGet Package Reference**

In `FortiCertSync\FortiCertSync.csproj`, change:

```xml
<PackageReference Include="System.Security.Cryptography.ProtectedData" Version="9.0.8" />
```
to:
```xml
<PackageReference Include="System.Security.Cryptography.ProtectedData" Version="10.0.5" />
```

`Microsoft.SourceLink.GitHub` 8.0.0 is compatible — no update required.

**Step 3 — Restore and Build**

Run `dotnet restore` then `dotnet build` to surface all compilation errors from source-incompatible APIs.

**Step 4 — Fix Source-Incompatible APIs**

See [Breaking Changes Catalog](#breaking-changes-catalog) for the complete list of required code changes. Key areas:

- `DataProtectionScope` + `ProtectedData` usage in `FortiClient.cs` (lines 29, 40)
- `X509Certificate2(byte[])` constructor usage in `WindowsCertService.cs` (line 21)
- `TimeSpan.FromSeconds(double)` in `FortiClient.cs` (line 32)
- `X509Certificate2Collection.Import(byte[], string, X509KeyStorageFlags)` in `FortiClient.cs` (line 211)

**Step 5 — Rebuild and Verify**

Run `dotnet build` again. Expected outcome: **0 errors, 0 warnings** (excluding pre-existing `#pragma warning disable` suppressions).

#### Validation Checklist

- [ ] `TargetFramework` set to `net10.0` in `.csproj`
- [ ] `System.Security.Cryptography.ProtectedData` updated to `10.0.5`
- [ ] Solution builds with 0 errors
- [ ] No new compiler warnings introduced
- [ ] App starts and connects to FortiGate (smoke test)
- [ ] Certificate import/delete/rebind operations verified at runtime

---

## Package Update Reference

### Common Package Updates

| Package | Current Version | Target Version | Projects Affected | Reason |
|---------|----------------|----------------|-------------------|--------|
| `System.Security.Cryptography.ProtectedData` | `9.0.8` | `10.0.5` | `FortiCertSync.csproj` | Framework alignment — upgrade recommended |

### Compatible Packages (no update required)

| Package | Version | Projects Affected | Status |
|---------|---------|-------------------|--------|
| `Microsoft.SourceLink.GitHub` | `8.0.0` | `FortiCertSync.csproj` | ✅ Compatible with `net10.0` |

---

## Breaking Changes Catalog

### Source-Incompatible Changes (require code fixes)

#### 1. `DataProtectionScope` + `ProtectedData` (Windows DPAPI)

| Property | Detail |
|----------|--------|
| **API** | `System.Security.Cryptography.DataProtectionScope`, `ProtectedData.Protect`, `ProtectedData.Unprotect` |
| **File** | `FortiCertSync\FortiClient.cs` |
| **Lines** | 29, 40 |
| **Issue** | Source-incompatible in .NET 10; the type may require explicit `using` disambiguation or the API surface changed |
| **Fix** | Ensure `using System.Security.Cryptography;` is present. If compiler reports ambiguity or removal, switch to the fully-qualified type name `System.Security.Cryptography.ProtectedData.Protect(...)` / `...Unprotect(...)`. These are Windows-only APIs — the existing `#pragma warning disable CA1416` suppression remains valid. Verify the package reference `System.Security.Cryptography.ProtectedData 10.0.5` is in place. |

**Affected code:**
```csharp
// Line 29
ProtectedData.Unprotect(Convert.FromBase64String(tokenRaw[4..]), null, DataProtectionScope.CurrentUser)
// Line 40
ProtectedData.Protect(Encoding.UTF8.GetBytes(token), null, DataProtectionScope.CurrentUser)
```

#### 2. `X509Certificate2(byte[])` Constructor

| Property | Detail |
|----------|--------|
| **API** | `M:System.Security.Cryptography.X509Certificates.X509Certificate2.#ctor(System.Byte[])` |
| **File** | `FortiCertSync\WindowsCertService.cs` |
| **Line** | 21 |
| **Issue** | This constructor overload is obsoleted/source-incompatible in .NET 10. |
| **Fix** | Replace with `X509CertificateLoader.LoadCertificate(bytes)` (available in .NET 9+). If the original usage wraps an existing `X509Certificate2` for cloning, use `new X509Certificate2(cert.Export(X509ContentType.Cert))` or the copy constructor pattern, checking what overload resolves cleanly. |

**Affected code:**
```csharp
// WindowsCertService.cs line 21
return cert is null ? null : new X509Certificate2(cert); // detached clone
```

**Recommended replacement:**
```csharp
return cert is null ? null : X509CertificateLoader.LoadCertificate(cert.RawData);
```

#### 3. `TimeSpan.FromSeconds(double)`

| Property | Detail |
|----------|--------|
| **API** | `M:System.TimeSpan.FromSeconds(System.Double)` |
| **File** | `FortiCertSync\FortiClient.cs` |
| **Line** | 32 |
| **Issue** | Source-incompatible — .NET 10 added new `TimeSpan.FromSeconds(int)` / `TimeSpan.FromSeconds(long)` overloads; passing a literal integer constant may now be ambiguous. |
| **Fix** | Use an explicit `double` literal to disambiguate: `TimeSpan.FromSeconds(30.0)` instead of `TimeSpan.FromSeconds(30)`. |

**Affected code:**
```csharp
// FortiClient.cs line 32
var h = new HttpClient { Timeout = TimeSpan.FromSeconds(30) };
```

**Recommended replacement:**
```csharp
var h = new HttpClient { Timeout = TimeSpan.FromSeconds(30.0) };
```

#### 4. `X509Certificate2Collection.Import(byte[], string, X509KeyStorageFlags)`

| Property | Detail |
|----------|--------|
| **API** | `M:System.Security.Cryptography.X509Certificates.X509Certificate2Collection.Import(System.Byte[],System.String,System.Security.Cryptography.X509Certificates.X509KeyStorageFlags)` |
| **File** | `FortiCertSync\FortiClient.cs` |
| **Line** | 211 |
| **Issue** | Behavioral change in .NET 10 — import behaviour or exception types may differ. Classified as behavioral (🔵), not source-incompatible, but included here for completeness since runtime testing is critical. |
| **Fix** | No source change required. Validate at runtime that `collection.Import(pfxBytes, pfxPass, X509KeyStorageFlags.Exportable)` still populates the collection as expected after the upgrade. |

### Behavioral Changes (no code fix required — validate at runtime)

These APIs have behavioral changes in .NET 10 that are flagged but do not break compilation. Validate via runtime/smoke testing.

| API | Occurrences | File | Lines | Nature |
|-----|-------------|------|-------|--------|
| `HttpContent.ReadAsStringAsync()` | 10 | `FortiClient.cs` | 167, 168, 174, 175, 249, 276, 351, 360, 362 | Timeout/cancellation behavior may change |
| `JsonDocument.Parse()` | 8 | `FortiClient.cs` | 280, 362 | Parsing behavior or exception types may differ |
| `Uri.EscapeDataString()` | 5 | `FortiClient.cs` | 172, 188, 265, 270 | May now throw on certain inputs rather than silently encoding |
| `X509DistinguishedName.ctor(string)` | 1 | `FortiClient.cs` | — | Parsing behavior change |
| `X509Certificate2Collection.Import(...)` | 1 | `FortiClient.cs` | 211 | See above |

**Key runtime validations:**
- All HTTP responses are read correctly via `ReadAsStringAsync()`
- JSON payloads parsed by `JsonDocument.Parse()` produce expected results
- URLs built with `Uri.EscapeDataString()` are accepted by FortiGate API without encoding regressions

---

## Testing & Validation Strategy

### Phase 1: Build Validation

After the atomic upgrade is complete:

- [ ] `dotnet restore` completes without errors
- [ ] `dotnet build` reports **0 errors**
- [ ] No new warnings beyond pre-existing suppressions

### Phase 2: Runtime / Smoke Validation

There are no automated test projects in this solution. Validation must be performed manually:

| Test | Expected Result |
|------|----------------|
| App starts without exception | ✅ Process exits with code 0 (or continues running) |
| FortiGate connection (ping check) | ✅ Auth succeeds — `200 OK` from `/api/v2/monitor/system/status` |
| API key DPAPI encrypt/decrypt cycle | ✅ Key is encrypted to `enc:` prefix on first run; decrypted correctly on subsequent run |
| `ListLocalCertsAsync` | ✅ Returns expected certificates from FortiGate |
| Certificate import (`ImportCaFromPfxAsync`) | ✅ PFX imported successfully; no exception |
| Certificate deletion (`DeleteLocalCertAsync`) | ✅ Named cert removed from FortiGate |
| `RebindAutoAsync` | ✅ Usage references updated correctly |
| URL construction (`Uri.EscapeDataString`) | ✅ Special characters in cert names encoded correctly |
| JSON parsing (`JsonDocument.Parse`) | ✅ Parsed fields match expected values |

### No Automated Test Projects

⚠️ The solution has no test projects. All validation above must be performed via manual execution against a FortiGate test environment or a staging appliance.

---

## Risk Management

### Risk Table

| Area | Risk Level | Description | Mitigation |
|------|-----------|-------------|------------|
| `ProtectedData` / DPAPI | 🟡 Medium | Windows-only API; source-incompatible flag in .NET 10. Could break API key encryption/decryption on first run after upgrade. | Validate immediately after upgrade — first run re-encrypts the key; confirm decryption round-trip works. |
| `X509Certificate2(byte[])` | 🟡 Medium | Constructor obsoleted/removed in .NET 10; if not replaced, app will fail to clone certificates. | Replace with `X509CertificateLoader.LoadCertificate()` before build. |
| `TimeSpan.FromSeconds(30)` | 🟢 Low | Ambiguous overload; will surface as a compile error. | Easy fix: change to `30.0`. |
| Behavioral changes (HttpContent, JsonDocument, Uri) | 🟢 Low | 25 behavioral-change flags — unlikely to break in practice for typical REST payloads, but require runtime confirmation. | Smoke test against live or test FortiGate after upgrade. |
| AOT compilation (`PublishAot=true`) | 🟡 Medium | AOT trimming is enabled. .NET 10 may trim differently. Any reflection-dependent code not annotated correctly could fail at publish time. | Run `dotnet publish -r win-x64 -c Release` after build succeeds and verify no AOT warnings are promoted to errors. |
| No automated tests | 🟡 Medium | Without a test suite, regressions can only be caught manually. | Prioritise manual smoke testing against a FortiGate test environment immediately post-upgrade. |

### Contingency Plans

- **If DPAPI breaks**: Temporarily disable encryption (remove `enc:` prefix handling), validate plain token works, then re-investigate `ProtectedData` usage.
- **If AOT publish fails**: Add `[DynamicDependency]` attributes or `rd.xml` suppressions for any reflection paths identified in AOT warnings.
- **If behavioral changes cause unexpected errors**: Inspect HTTP responses, compare with .NET 8 behaviour using a side-by-side test run.

---

## Complexity & Effort Assessment

### Per-Project Complexity

| Project | Complexity | LOC | Package Updates | Source-Incompatible APIs | Risk |
|---------|-----------|-----|-----------------|--------------------------|------|
| `FortiCertSync.csproj` | **Low** | 764 | 1 | 12 (in 2 files) | 🟢 Low |

### Phase Complexity

| Phase | Description | Complexity |
|-------|-------------|-----------|
| Phase 1: Atomic Upgrade | Framework + package + code fixes | **Low** |
| Validation | Manual smoke testing | **Low–Medium** (depends on FortiGate availability) |

### Resource Requirements

- **Skill Level**: Mid-level .NET developer familiar with X.509 and Windows DPAPI
- **Environment**: Windows machine with access to a FortiGate test appliance
- **Parallel Capacity**: Single developer, single operation

---

## Source Control Strategy

### Branching

Upgrade is performed on the current working branch (`master`) as requested — no separate upgrade branch.

### Commit Strategy

**Single commit** for the entire upgrade, once the solution builds with 0 errors:

```
chore: upgrade FortiCertSync from net8.0 to net10.0

- TargetFramework: net8.0 → net10.0
- System.Security.Cryptography.ProtectedData: 9.0.8 → 10.0.5
- Fixed source-incompatible API: X509Certificate2(byte[]) → X509CertificateLoader.LoadCertificate()
- Fixed source-incompatible API: TimeSpan.FromSeconds(30) → TimeSpan.FromSeconds(30.0)
- Verified DataProtectionScope / ProtectedData usage compatible with net10.0
```

### Review Process

Direct commit to `master` is acceptable given the small scope. A self-review of the diff focusing on the 4 source-incompatible API fixes is recommended before committing.

---

## Success Criteria

### Technical Criteria

| Criterion | Target |
|-----------|--------|
| `TargetFramework` | `net10.0` in `.csproj` |
| `System.Security.Cryptography.ProtectedData` | `10.0.5` |
| Build result | 0 errors |
| Build warnings | No new warnings beyond pre-existing suppressions |
| Source-incompatible APIs resolved | All 12 occurrences fixed |
| `dotnet publish -r win-x64 -c Release` (AOT) | Succeeds with 0 AOT errors |

### Quality Criteria

| Criterion | Target |
|-----------|--------|
| DPAPI encryption round-trip | API key encrypted and decrypted correctly |
| FortiGate connectivity | Auth succeeds post-upgrade |
| Certificate operations | Import, delete, rebind all function correctly |
| URL encoding | No regressions from `Uri.EscapeDataString` behavior change |
| JSON parsing | All FortiGate API responses parsed correctly |

### Process Criteria

| Criterion | Target |
|-----------|--------|
| Strategy followed | All-At-Once — single atomic operation |
| Single commit | One clean commit on `master` once build passes |
| No intermediate broken states | Build passes before commit |
