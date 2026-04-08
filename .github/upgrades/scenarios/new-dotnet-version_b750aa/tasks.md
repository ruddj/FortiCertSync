# FortiCertSync .NET 10.0 Upgrade Tasks

## Overview

This document tracks the execution of the FortiCertSync upgrade from .NET 8.0 to .NET 10.0. The single project will be upgraded in one atomic operation.

**Progress**: 0/1 tasks complete (0%) ![0%](https://progress-bar.xyz/0)

---

## Tasks

### [▶] TASK-001: Atomic framework and package upgrade
**References**: Plan §Migration Strategy, Plan §Project-by-Project Plans, Plan §Package Update Reference, Plan §Breaking Changes Catalog

- [✓] (1) Update TargetFramework to net10.0 in FortiCertSync\FortiCertSync.csproj
- [✓] (2) Update System.Security.Cryptography.ProtectedData from 9.0.8 to 10.0.5 in FortiCertSync\FortiCertSync.csproj
- [✓] (3) Restore dependencies
- [✓] (4) Build solution and fix all compilation errors per Plan §Breaking Changes Catalog (key areas: DataProtectionScope/ProtectedData usage in FortiClient.cs, X509Certificate2(byte[]) constructor in WindowsCertService.cs, TimeSpan.FromSeconds ambiguity in FortiClient.cs)
- [✓] (5) Solution builds with 0 errors (**Verify**)
- [▶] (6) Commit changes with message: "chore: upgrade FortiCertSync from net8.0 to net10.0"

---





