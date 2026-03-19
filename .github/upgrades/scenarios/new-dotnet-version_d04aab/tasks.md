# Pomelo.EntityFrameworkCore.MySql .NET 10.0 Upgrade Tasks

## Overview

This document tracks the execution of the Pomelo.EntityFrameworkCore.MySql solution upgrade from .NET 8.0/9.0 to .NET 10.0. All 8 projects will be upgraded simultaneously in a single atomic operation, followed by comprehensive testing and validation.

**Progress**: 1/4 tasks complete (25%) ![0%](https://progress-bar.xyz/25)

---

## Tasks

### [✓] TASK-001: Verify prerequisites *(Completed: 2026-03-18 17:21)*

**References**: Plan §4 Phase 0

- [✓] (1) Verify .NET 10.0 SDK is installed per Plan §Prerequisites
- [✓] (2) SDK version meets minimum requirements (**Verify**)

---

### [▶] TASK-002: Atomic framework and package upgrade with compilation fixes

**References**: Plan §5 Steps 1-4, Plan §7 Package Update Reference, Plan §8 Breaking Changes Catalog

- [▶] (1) Update `global.json` SDK version to `10.0.100` per Plan §5 Step 1

- [ ] (2) `global.json` updated successfully (**Verify**)

- [ ] (3) Update `Directory.Build.props` framework properties per Plan §5 Step 2 (all 6 properties: `PomeloTargetFramework`, `PomeloTestTargetFramework`, `EfCoreTargetFramework`, `EfCoreTestTargetFramework`, `MySqlConnectorTargetFramework`, `MySqlConnectorDependencyInjectionTargetFramework` from `net8.0`/`net9.0` → `net10.0`)

- [ ] (4) All framework properties updated to `net10.0` (**Verify**)

- [ ] (5) Update `Directory.Packages.props` per Plan §7 Package Update Reference (update `EFCoreVersion` property to `[10.0.0,10.0.999]` and all 16 individual package versions listed)

- [ ] (6) All package versions updated per Plan §7 (**Verify**)

- [ ] (7) Restore all dependencies

- [ ] (8) All dependencies restored successfully (**Verify**)

- [ ] (9) Build solution and fix all compilation errors per Plan §8 Breaking Changes Catalog (focus: TimeSpan overload ambiguities in 56 locations across FunctionalTests and IntegrationTests; WebHost migration in IntegrationTests\Program.cs; DependencyModel API in Tests\BuildReference.cs; string.Join ambiguity in IntegrationTests\GeneratedTypesTest.cs)

- [ ] (10) Solution builds with 0 errors (**Verify**)

---

### [ ] TASK-003: Run full test suite and validate upgrade

**References**: Plan §10 Testing & Validation Strategy

- [ ] (1) Run tests in all 3 test projects: `EFCore.MySql.FunctionalTests`, `EFCore.MySql.IntegrationTests`, `EFCore.MySql.Tests`
- [ ] (2) Fix any test failures (reference Plan §8 Breaking Changes Catalog for behavioral changes - JsonDocument behavioral differences may affect JSON tests)
- [ ] (3) Re-run tests after fixes
- [ ] (4) All tests pass with 0 failures (**Verify**)

---

### [ ] TASK-004: Final commit

**References**: Plan §12 Source Control Strategy

- [ ] (1) Commit all changes with message: "Upgrade to .NET 10.0 (LTS)"

---
