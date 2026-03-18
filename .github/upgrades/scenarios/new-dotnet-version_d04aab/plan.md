# .NET 10.0 Upgrade Plan — Pomelo.EntityFrameworkCore.MySql

## Table of Contents

- [1. Executive Summary](#1-executive-summary)
- [2. Migration Strategy](#2-migration-strategy)
- [3. Detailed Dependency Analysis](#3-detailed-dependency-analysis)
- [4. Implementation Timeline](#4-implementation-timeline)
- [5. Detailed Execution Steps](#5-detailed-execution-steps)
- [6. Project-by-Project Migration Plans](#6-project-by-project-migration-plans)
- [7. Package Update Reference](#7-package-update-reference)
- [8. Breaking Changes Catalog](#8-breaking-changes-catalog)
- [9. Risk Management](#9-risk-management)
- [10. Testing & Validation Strategy](#10-testing--validation-strategy)
- [11. Complexity & Effort Assessment](#11-complexity--effort-assessment)
- [12. Source Control Strategy](#12-source-control-strategy)
- [13. Success Criteria](#13-success-criteria)

---

## 1. Executive Summary

### Scenario

Upgrade all projects in the **Pomelo.EntityFrameworkCore.MySql** solution from their current target frameworks (.NET 8 for `src` libraries, .NET 9 for `test` and `tools` projects) to **.NET 10.0 (LTS)**.

### Scope

| Metric | Value |
|:---|:---|
| Total Projects | 8 (4 src, 3 test, 1 tool) |
| Total NuGet Packages | 28 (13 need upgrade, 15 compatible) |
| Total Code Files | 622 |
| Total Lines of Code | 127,534 |
| Total API Issues | 113 (56 source-incompatible, 57 behavioral changes) |
| Estimated LOC to Modify | 113+ (< 0.1% of codebase) |
| Security Vulnerabilities | 0 |

### Selected Strategy

**All-At-Once Strategy** — All 8 projects upgraded simultaneously in a single atomic operation.

**Rationale:**
- Small solution (8 projects, well below 30-project threshold)
- All projects already on .NET 8.0+ (modern SDK-style)
- Homogeneous codebase with consistent patterns
- All NuGet packages have known target versions available
- No security vulnerabilities to address
- Clear dependency structure (max depth 3, no circular dependencies)
- All projects rated 🟢 Low difficulty

### Complexity Assessment

- **Classification**: **Simple**
- Max dependency depth: 3
- High-risk projects: 0
- Circular dependencies: None
- All API breaking changes are confined to test projects (no production code changes needed beyond framework/package updates)

### Critical Notes

- **Centralized version management**: This solution uses `Directory.Build.props` and `Directory.Packages.props` for centralized framework targeting and package version management. Target framework properties (`PomeloTargetFramework`, `PomeloTestTargetFramework`, `EfCoreTargetFramework`, `EfCoreTestTargetFramework`) are defined in root `Directory.Build.props`.
- **Central Package Management (CPM)**: All package versions are defined in `Directory.Packages.props` with `<ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>`.
- **EF Core version range**: The `EFCoreVersion` property currently uses `[9.0.0,9.0.999]`; this must be updated to the 10.x range.
- **`global.json`**: SDK version `9.0.100` with `rollForward: latestFeature` — must be updated to .NET 10.0 SDK.

## 2. Migration Strategy

### Approach: All-At-Once

All 8 projects are upgraded simultaneously in a single coordinated operation. No intermediate states or partial upgrades.

### Justification

| Criterion | Assessment | Supports All-At-Once? |
|:---|:---|:---|
| Project count | 8 projects (small solution) | ✅ Yes |
| Current framework | All on .NET 8.0+ (modern) | ✅ Yes |
| Dependency complexity | Max depth 3, no cycles | ✅ Yes |
| Package compatibility | All 13 packages have known target versions | ✅ Yes |
| Security vulnerabilities | None | ✅ Yes |
| Risk profile | All projects rated Low difficulty | ✅ Yes |
| Codebase size | 127K LOC but <0.1% estimated changes | ✅ Yes |
| Centralized config | Single `Directory.Build.props` + `Directory.Packages.props` | ✅ Yes (minimal file changes) |

### Key Advantages for This Solution

1. **Centralized framework properties**: Changing `PomeloTargetFramework` and `PomeloTestTargetFramework` in one file (`Directory.Build.props`) updates all 8 projects at once.
2. **Central Package Management**: All package versions in `Directory.Packages.props` — a single file update handles all 13 package upgrades.
3. **No production code changes**: All source-incompatible API issues are confined to test projects; the 4 `src` libraries have no source-incompatible issues.
4. **Clean dependency resolution**: Single atomic update avoids version mismatches between EF Core packages across projects.

### Execution Order (within the atomic operation)

1. Update `global.json` — SDK version to 10.x
2. Update `Directory.Build.props` — framework properties to `net10.0`
3. Update `Directory.Packages.props` — all package versions
4. Restore dependencies
5. Build solution and fix all compilation errors
6. Verify solution builds with 0 errors

## 3. Detailed Dependency Analysis

### Dependency Graph Summary

```
Level 0 (Foundation — no project dependencies):
  ├── EFCore.MySql.csproj              [src, net8.0]  ← core library, depended on by 6 projects
  └── QueryBaselineUpdater.csproj      [tools, net9.0] ← standalone tool, no dependants

Level 1 (depends on Level 0 only):
  ├── EFCore.MySql.Json.Microsoft.csproj  [src, net8.0]  → depends on: EFCore.MySql
  ├── EFCore.MySql.Json.Newtonsoft.csproj [src, net8.0]  → depends on: EFCore.MySql
  └── EFCore.MySql.NTS.csproj            [src, net8.0]  → depends on: EFCore.MySql

Level 2 (depends on Levels 0–1):
  ├── EFCore.MySql.FunctionalTests.csproj [test, net9.0] → depends on: EFCore.MySql, Json.Microsoft, Json.Newtonsoft, NTS
  └── EFCore.MySql.IntegrationTests.csproj [test, net9.0] → depends on: EFCore.MySql, Json.Newtonsoft

Level 3 (depends on Levels 0–2):
  └── EFCore.MySql.Tests.csproj          [test, net9.0] → depends on: EFCore.MySql, FunctionalTests
```

### Project Groupings

Since this is an **All-At-Once** upgrade, all projects are upgraded simultaneously. The dependency levels above are documented for context only — there are no intermediate migration phases.

| Group | Projects | Description |
|:---|:---|:---|
| Source Libraries | `EFCore.MySql`, `EFCore.MySql.Json.Microsoft`, `EFCore.MySql.Json.Newtonsoft`, `EFCore.MySql.NTS` | Core packages distributed via NuGet |
| Test Projects | `EFCore.MySql.FunctionalTests`, `EFCore.MySql.IntegrationTests`, `EFCore.MySql.Tests` | Validation suite |
| Tools | `QueryBaselineUpdater` | Standalone utility |

### Critical Path

`EFCore.MySql` → `Json.Microsoft/Json.Newtonsoft/NTS` → `FunctionalTests` → `Tests`

The core `EFCore.MySql.csproj` is the foundation — all other projects depend on it directly or transitively. However, since all updates happen atomically, this ordering affects compilation resolution, not migration sequencing.

### Circular Dependencies

None detected.

## 4. Implementation Timeline

### Phase 0: Preparation
- Verify .NET 10.0 SDK is installed (✅ confirmed)
- Update `global.json` to target .NET 10.0 SDK

### Phase 1: Atomic Upgrade
**Operations** (performed as single coordinated batch):
- Update `Directory.Build.props` framework properties (`net8.0` → `net10.0`, `net9.0` → `net10.0`)
- Update `Directory.Packages.props` with all 13 package version changes
- Restore dependencies
- Build solution and fix all compilation errors from framework/package upgrades

**Deliverables**: Solution builds with 0 errors

### Phase 2: Test Validation
**Operations**:
- Execute all 3 test projects
- Address any test failures

**Deliverables**: All tests pass

## 5. Detailed Execution Steps

### Step 1: Update `global.json`

Update the SDK version to target .NET 10.0:

| Property | Current Value | Target Value |
|:---|:---|:---|
| `sdk.version` | `9.0.100` | `10.0.100` |

The `rollForward: latestFeature` and `allowPrerelease: false` settings should be preserved.

### Step 2: Update `Directory.Build.props` Framework Properties

Update the centralized framework target properties (lines 29–34 of `Directory.Build.props`):

| Property | Current Value | Target Value | Affects |
|:---|:---|:---|:---|
| `PomeloTargetFramework` | `net8.0` | `net10.0` | All 4 `src` projects |
| `PomeloTestTargetFramework` | `net9.0` | `net10.0` | All 3 `test` projects + `QueryBaselineUpdater` |
| `EfCoreTargetFramework` | `net8.0` | `net10.0` | Local EF Core reference paths |
| `EfCoreTestTargetFramework` | `net9.0` | `net10.0` | Local EF Core test reference paths |
| `MySqlConnectorTargetFramework` | `net8.0` | `net10.0` | Local MySqlConnector reference paths |
| `MySqlConnectorDependencyInjectionTargetFramework` | `net8.0` | `net10.0` | Local MySqlConnector DI reference paths |

### Step 3: Update `Directory.Packages.props` Package Versions

Update the `EFCoreVersion` property and all individual package versions:

| Property / Package | Current Version | Target Version |
|:---|:---|:---|
| `EFCoreVersion` (property) | `[9.0.0,9.0.999]` | `[10.0.0,10.0.999]` |
| `Microsoft.AspNetCore.Identity.EntityFrameworkCore` | `9.0.0` | `10.0.5` |
| `Microsoft.AspNetCore.Mvc.NewtonsoftJson` | `9.0.0` | `10.0.5` |
| `Microsoft.Bcl.AsyncInterfaces` | `9.0.0` | `10.0.5` |
| `Microsoft.Extensions.Caching.Memory` | `9.0.0` | `10.0.5` |
| `Microsoft.Extensions.Configuration.Binder` | `9.0.0` | `10.0.5` |
| `Microsoft.Extensions.Configuration.EnvironmentVariables` | `9.0.0` | `10.0.5` |
| `Microsoft.Extensions.Configuration.FileExtensions` | `9.0.0` | `10.0.5` |
| `Microsoft.Extensions.Configuration.Json` | `9.0.0` | `10.0.5` |
| `Microsoft.Extensions.Configuration` | `9.0.0` | `10.0.5` |
| `Microsoft.Extensions.DependencyInjection` | `9.0.0` | `10.0.5` |
| `Microsoft.Extensions.DependencyModel` | `9.0.0` | `10.0.5` |
| `Microsoft.Extensions.Logging` | `9.0.0` | `10.0.5` |
| `Newtonsoft.Json` | `13.0.3` | `13.0.4` |
| `System.Collections.Immutable` | `9.0.0` | `10.0.5` |
| `System.Diagnostics.DiagnosticSource` | `9.0.0` | `10.0.5` |
| `System.Text.Json` | `9.0.0` | `10.0.5` |

> **Note**: The `EFCoreVersion` property controls 4 packages: `Microsoft.EntityFrameworkCore`, `Microsoft.EntityFrameworkCore.Design`, `Microsoft.EntityFrameworkCore.Relational`, and `Microsoft.EntityFrameworkCore.Relational.Specification.Tests`. Updating the property handles all 4 at once.

### Step 4: Build Solution and Fix All Compilation Errors

After updating all configuration files, restore dependencies and build the entire solution. Fix all compilation errors caused by the framework and package upgrades. Expected breaking changes are cataloged in §[8. Breaking Changes Catalog](#8-breaking-changes-catalog).

Key areas requiring code changes:
- **`TimeSpan.FromHours/FromSeconds/FromMilliseconds`** overload resolution changes (26+ occurrences across test projects)
- **`IWebHost`/`WebHost`** source incompatibility in `IntegrationTests\Program.cs`
- **`IdentityEntityFrameworkBuilderExtensions.AddEntityFrameworkStores`** signature change in `IntegrationTests\Startup.cs`
- **`DependencyContext`/`CompilationLibrary`** API changes in `Tests\TestUtilities\BuildReference.cs`
- **`string.Join` with `ReadOnlySpan<string>`** ambiguity in `IntegrationTests\Tests\Models\GeneratedTypesTest.cs`

### Step 5: Execute Tests

Run all 3 test projects:
- `test\EFCore.MySql.FunctionalTests`
- `test\EFCore.MySql.IntegrationTests`
- `test\EFCore.MySql.Tests`

Address any test failures, paying particular attention to:
- `System.Text.Json.JsonDocument` behavioral changes (57 occurrences — may affect JSON serialization/deserialization tests)
- `ConsoleLoggerExtensions.AddConsole` behavioral changes (3 occurrences in IntegrationTests)

## 6. Project-by-Project Migration Plans

### Project: src\EFCore.MySql\EFCore.MySql.csproj

**Current State:**
- Target Framework: `net8.0` (via `$(PomeloTargetFramework)`)
- Project Kind: ClassLibrary
- Dependencies: 0 project dependencies
- Dependants: 6 projects
- Files: 204 | LOC: 31,151
- API Issues: 0 source-incompatible, 0 behavioral
- Risk: 🟢 Low

**Target State:**
- Target Framework: `net10.0`
- Updated packages: 1 (`Microsoft.EntityFrameworkCore.Relational` via `EFCoreVersion`)

**Migration Steps:**
1. Framework update handled by `PomeloTargetFramework` property change in `Directory.Build.props`
2. Package update handled by `EFCoreVersion` property change in `Directory.Packages.props`
3. No code changes expected
4. Validation: Builds without errors or warnings

---

### Project: src\EFCore.MySql.Json.Microsoft\EFCore.MySql.Json.Microsoft.csproj

**Current State:**
- Target Framework: `net8.0` (via `$(PomeloTargetFramework)`)
- Project Kind: ClassLibrary
- Dependencies: `EFCore.MySql`
- Dependants: `FunctionalTests`
- Files: 28 | LOC: 2,092
- API Issues: 0 source-incompatible, 10 behavioral (`System.Text.Json.JsonDocument`)
- Risk: 🟢 Low

**Target State:**
- Target Framework: `net10.0`
- Updated packages: 1 (`Microsoft.EntityFrameworkCore.Relational` via `EFCoreVersion`)

**Migration Steps:**
1. Framework update handled by `PomeloTargetFramework` property change
2. Package update handled by `EFCoreVersion` property change
3. No code changes expected (behavioral changes only — `JsonDocument` may behave differently at runtime but no compilation errors expected)
4. Validation: Builds without errors

**Behavioral Changes to Monitor:**
- `System.Text.Json.JsonDocument` — 10 occurrences in value converter files:
  - `MySqlJsonMicrosoftJsonDocumentValueConverter.cs` (lines 16, 20, 29, 30)
  - `MySqlJsonMicrosoftStringValueConverter.cs` (line 30)
  - `MySqlJsonMicrosoftJsonElementValueConverter.cs` (line 27)

---

### Project: src\EFCore.MySql.Json.Newtonsoft\EFCore.MySql.Json.Newtonsoft.csproj

**Current State:**
- Target Framework: `net8.0` (via `$(PomeloTargetFramework)`)
- Project Kind: ClassLibrary
- Dependencies: `EFCore.MySql`
- Dependants: `FunctionalTests`, `IntegrationTests`
- Files: 27 | LOC: 1,870
- API Issues: 0
- Risk: 🟢 Low

**Target State:**
- Target Framework: `net10.0`
- Updated packages: 3 (`Microsoft.EntityFrameworkCore.Relational` via `EFCoreVersion`, `Microsoft.Extensions.DependencyInjection`, `Newtonsoft.Json`)

**Migration Steps:**
1. Framework update handled by `PomeloTargetFramework` property change
2. Package updates handled centrally in `Directory.Packages.props`
3. No code changes expected
4. Validation: Builds without errors

---

### Project: src\EFCore.MySql.NTS\EFCore.MySql.NTS.csproj

**Current State:**
- Target Framework: `net8.0` (via `$(PomeloTargetFramework)`)
- Project Kind: ClassLibrary
- Dependencies: `EFCore.MySql`
- Dependants: `FunctionalTests`
- Files: 33 | LOC: 3,024
- API Issues: 0
- Risk: 🟢 Low

**Target State:**
- Target Framework: `net10.0`
- Updated packages: 2 (`Microsoft.EntityFrameworkCore.Relational` via `EFCoreVersion`, `Microsoft.Extensions.DependencyInjection`)

**Migration Steps:**
1. Framework update handled by `PomeloTargetFramework` property change
2. Package updates handled centrally in `Directory.Packages.props`
3. No code changes expected
4. Validation: Builds without errors

---

### Project: test\EFCore.MySql.FunctionalTests\EFCore.MySql.FunctionalTests.csproj

**Current State:**
- Target Framework: `net9.0` (via `$(PomeloTestTargetFramework)`)
- Project Kind: DotNetCoreApp (test project)
- Dependencies: `EFCore.MySql`, `Json.Microsoft`, `Json.Newtonsoft`, `NTS`
- Dependants: `EFCore.MySql.Tests`
- Files: 270 | LOC: 83,092
- API Issues: 32 source-incompatible, 44 behavioral
- Risk: 🟢 Low

**Target State:**
- Target Framework: `net10.0`
- Updated packages: 8 (EF Core packages via `EFCoreVersion`, `Microsoft.Extensions.Configuration.*`)

**Migration Steps:**
1. Framework update handled by `PomeloTestTargetFramework` property change
2. Package updates handled centrally in `Directory.Packages.props`
3. **Code changes required:**
   - `TimeSpan.FromHours(double)` → cast argument explicitly: `TimeSpan.FromHours((double)-8.0)` or use the integer overload if appropriate: `TimeSpan.FromHours(-8)`.
   - `TimeSpan.FromHours(int)` → cast argument: `TimeSpan.FromHours((int)6)` or ensure the argument type is unambiguous.
   - `TimeSpan.FromSeconds(long)` → cast argument explicitly: `TimeSpan.FromSeconds((long)10)` or `TimeSpan.FromSeconds((double)10)`.
   - `TimeSpan.FromMilliseconds(double)` → cast argument explicitly: `TimeSpan.FromMilliseconds((double)1000.0)`.
   - `TimeSpan.FromMilliseconds(long, long)` → cast argument or verify method selection is correct with new overloads (lines 85, 88).
4. Validation: Builds without errors, tests pass

**Behavioral Changes to Monitor:**
- `System.Text.Json.JsonDocument` — 44 occurrences across test files (`JsonMicrosoftDomQueryTest.cs`, `MySqlJsonMicrosoftTypeMappingTest.cs`, etc.)

---

### Project: test\EFCore.MySql.IntegrationTests\EFCore.MySql.IntegrationTests.csproj

**Current State:**
- Target Framework: `net9.0` (via `$(PomeloTestTargetFramework)`)
- Project Kind: AspNetCore (test project, Sdk=`Microsoft.NET.Sdk.Web`)
- Dependencies: `EFCore.MySql`, `Json.Newtonsoft`
- Dependants: None (top-level)
- Files: 42 | LOC: 3,117
- API Issues: 18 source-incompatible, 3 behavioral
- Risk: 🟢 Low

**Target State:**
- Target Framework: `net10.0`
- Updated packages: 5 (`Microsoft.AspNetCore.Identity.EntityFrameworkCore`, `Microsoft.AspNetCore.Mvc.NewtonsoftJson`, `Newtonsoft.Json`, EF Core packages via `EFCoreVersion`)

**Migration Steps:**
1. Framework update handled by `PomeloTestTargetFramework` property change
2. Package updates handled centrally in `Directory.Packages.props`
3. **Code changes required:**
   - **`Program.cs`** (lines 43–48): `IWebHost`/`WebHost.CreateDefaultBuilder` is source-incompatible in .NET 10. Update `BuildWebHost` to use `WebApplication.CreateBuilder` or `Host.CreateDefaultBuilder` pattern.
   - **`Startup.cs`** (line 37): `IdentityEntityFrameworkBuilderExtensions.AddEntityFrameworkStores` signature change — verify compatibility after package update; may need cast or method adjustment.
   - **`Tests\Models\GeneratedTypesTest.cs`** (line 24): `string.Join(string, ReadOnlySpan<string>)` ambiguity — explicitly convert arguments to `string[]`.
   - **`Tests\Models\GeneratedTypesTest.cs`**: `TimeSpan.FromSeconds(long)` (6 occurrences, lines 38, 91) and `TimeSpan.FromMilliseconds` (4 occurrences, lines 85–89) — cast arguments.
   - **`Tests\Models\ExpressionTest.cs`**: `TimeSpan.FromSeconds(long)` (2 occurrences, lines 167–168) — cast arguments.
   - **`Tests\Models\DataTypesTest.cs`**: `TimeSpan.FromMilliseconds(double)` (1 occurrence, line 85) — cast argument.
4. Validation: Builds without errors, tests pass

**Behavioral Changes to Monitor:**
- `ConsoleLoggerExtensions.AddConsole` — 3 occurrences (`Program.cs`, `AppDbScope.cs`, `Startup.cs`); behavior may change but no compilation error expected.

---

### Project: test\EFCore.MySql.Tests\EFCore.MySql.Tests.csproj

**Current State:**
- Target Framework: `net9.0` (via `$(PomeloTestTargetFramework)`)
- Project Kind: DotNetCoreApp (test project)
- Dependencies: `EFCore.MySql`, `FunctionalTests`
- Dependants: None (top-level)
- Files: 30 | LOC: 3,039
- API Issues: 6 source-incompatible, 0 behavioral
- Risk: 🟢 Low

**Target State:**
- Target Framework: `net10.0`
- Updated packages: 5 (EF Core packages via `EFCoreVersion`, `Microsoft.Extensions.DependencyModel`, `Newtonsoft.Json`)

**Migration Steps:**
1. Framework update handled by `PomeloTestTargetFramework` property change
2. Package updates handled centrally in `Directory.Packages.props`
3. **Code changes required:**
   - **`TestUtilities\BuildReference.cs`** (line 35): `DependencyContext.Default.CompileLibraries` and `CompilationLibrary.ResolveReferencePaths()` — these are source-incompatible in .NET 10. Verify after package update; may require using the updated `Microsoft.Extensions.DependencyModel` 10.0.5 API.
4. Validation: Builds without errors, tests pass

---

### Project: tools\QueryBaselineUpdater\QueryBaselineUpdater.csproj

**Current State:**
- Target Framework: `net9.0` (via `$(PomeloTestTargetFramework)`)
- Project Kind: DotNetCoreApp (console tool)
- Dependencies: None
- Dependants: None (standalone)
- Files: 1 | LOC: 149
- API Issues: 0
- Risk: 🟢 Low

**Target State:**
- Target Framework: `net10.0`
- Updated packages: 0

**Migration Steps:**
1. Framework update handled by `PomeloTestTargetFramework` property change
2. No package or code changes expected
3. Validation: Builds without errors

---

## 7. Package Update Reference

### EF Core Packages (controlled by `EFCoreVersion` property — affects 6 projects)

| Package | Current Range | Target Range | Projects Affected |
|:---|:---|:---|:---|
| `Microsoft.EntityFrameworkCore` | `[9.0.0,9.0.999]` | `[10.0.0,10.0.999]` | FunctionalTests |
| `Microsoft.EntityFrameworkCore.Design` | `[9.0.0,9.0.999]` | `[10.0.0,10.0.999]` | FunctionalTests, IntegrationTests, Tests |
| `Microsoft.EntityFrameworkCore.Relational` | `[9.0.0,9.0.999]` | `[10.0.0,10.0.999]` | EFCore.MySql, Json.Microsoft, Json.Newtonsoft, NTS, FunctionalTests, Tests |
| `Microsoft.EntityFrameworkCore.Relational.Specification.Tests` | `[9.0.0,9.0.999]` | `[10.0.0,10.0.999]` | FunctionalTests, IntegrationTests, Tests |

### Microsoft.Extensions.* Packages (individually versioned — affects multiple projects)

| Package | Current | Target | Projects Affected |
|:---|:---|:---|:---|
| `Microsoft.Extensions.Configuration.Binder` | `9.0.0` | `10.0.5` | FunctionalTests |
| `Microsoft.Extensions.Configuration.EnvironmentVariables` | `9.0.0` | `10.0.5` | FunctionalTests |
| `Microsoft.Extensions.Configuration.FileExtensions` | `9.0.0` | `10.0.5` | FunctionalTests |
| `Microsoft.Extensions.Configuration.Json` | `9.0.0` | `10.0.5` | FunctionalTests |
| `Microsoft.Extensions.Configuration` | `9.0.0` | `10.0.5` | (used in conditional build) |
| `Microsoft.Extensions.Caching.Memory` | `9.0.0` | `10.0.5` | (used in conditional build) |
| `Microsoft.Extensions.DependencyInjection` | `9.0.0` | `10.0.5` | Json.Newtonsoft, NTS |
| `Microsoft.Extensions.DependencyModel` | `9.0.0` | `10.0.5` | Tests |
| `Microsoft.Extensions.Logging` | `9.0.0` | `10.0.5` | (used in conditional build) |

### ASP.NET Core Packages (affects IntegrationTests)

| Package | Current | Target | Projects Affected |
|:---|:---|:---|:---|
| `Microsoft.AspNetCore.Identity.EntityFrameworkCore` | `9.0.0` | `10.0.5` | IntegrationTests |
| `Microsoft.AspNetCore.Mvc.NewtonsoftJson` | `9.0.0` | `10.0.5` | IntegrationTests |

### System.* Packages (individually versioned)

| Package | Current | Target | Projects Affected |
|:---|:---|:---|:---|
| `System.Collections.Immutable` | `9.0.0` | `10.0.5` | (used in conditional build) |
| `System.Diagnostics.DiagnosticSource` | `9.0.0` | `10.0.5` | (used in conditional build) |
| `System.Text.Json` | `9.0.0` | `10.0.5` | (used in conditional build) |

### Third-Party Packages

| Package | Current | Target | Projects Affected |
|:---|:---|:---|:---|
| `Newtonsoft.Json` | `13.0.3` | `13.0.4` | Json.Newtonsoft, IntegrationTests, Tests |

### Packages Requiring No Update (compatible as-is)

`DotNetAnalyzers.DocumentationAnalyzers`, `GitHubActionsTestLogger`, `Microsoft.CodeAnalysis` (4.10.0), `Microsoft.NET.Test.Sdk` (17.12.0), `Microsoft.SourceLink.GitHub` (8.0.0), `Moq` (4.20.72), `MySqlConnector` (2.4.0), `MySqlConnector.DependencyInjection` (2.4.0), `NetTopologySuite` (2.5.0), `StyleCop.Analyzers` (1.1.118), `xunit.assert` (2.9.2), `xunit.core` (2.9.2), `xunit.runner.console` (2.9.2), `xunit.runner.visualstudio` (2.8.2), `Xunit.SkippableFact` (1.4.13)

## 8. Breaking Changes Catalog

### Source-Incompatible Changes (56 occurrences — require code changes to compile)

#### 1. `TimeSpan.FromHours(double)` overload ambiguity (26 occurrences)

**Affected Projects:** FunctionalTests
**Affected Files:**
- `BuiltInDataTypesMySqlTest.cs` (lines 564, 614, 730, 780, 882, 931, 1528, 1574, 1602, 1648, 1676, 1721, 1749, 1798)
- `CustomConvertersMySqlTest.cs` (lines 113, 159, 187, 232, 260, 309)

**Issue:** In .NET 10, `TimeSpan.FromHours`, `FromMinutes`, `FromSeconds`, and `FromMilliseconds` now have overloads accepting `int`/`long` in addition to `double`. Calls like `TimeSpan.FromHours(-8.0)` become ambiguous.

**Fix:** Cast the argument explicitly: `TimeSpan.FromHours((double)-8.0)` or use the integer overload if appropriate: `TimeSpan.FromHours(-8)`.

#### 2. `TimeSpan.FromHours(int)` overload ambiguity (4 occurrences)

**Affected Projects:** FunctionalTests
**Affected Files:** `BuiltInDataTypesMySqlTest.cs` (lines 564, 614, 730, 780)

**Fix:** Explicit cast: `TimeSpan.FromHours((int)6)` or simply ensure the argument type is unambiguous.

#### 3. `TimeSpan.FromSeconds(long)` overload ambiguity (6 occurrences)

**Affected Projects:** IntegrationTests
**Affected Files:**
- `Tests\Models\ExpressionTest.cs` (lines 167, 168)
- `Tests\Models\GeneratedTypesTest.cs` (lines 38, 91)

**Fix:** Cast argument explicitly: `TimeSpan.FromSeconds((long)10)` or `TimeSpan.FromSeconds((double)10)`.

#### 4. `TimeSpan.FromMilliseconds(double)` overload ambiguity (5 occurrences)

**Affected Projects:** FunctionalTests, IntegrationTests
**Affected Files:**
- `StoredProcedureUpdateMySqlTest.cs` (line 637)
- `TestUtilities\MySqlTestStore.cs` (line 232)
- `Tests\Models\DataTypesTest.cs` (line 85)
- `Tests\Models\GeneratedTypesTest.cs` (lines 86, 89)

**Fix:** Cast argument explicitly: `TimeSpan.FromMilliseconds((double)1000.0)`.

#### 5. `TimeSpan.FromMilliseconds(long, long)` overload (2 occurrences)

**Affected Projects:** IntegrationTests
**Affected Files:** `Tests\Models\GeneratedTypesTest.cs` (lines 85, 88)

**Fix:** Cast argument or verify method selection is correct with new overloads.

#### 6. `IWebHost` / `WebHost.CreateDefaultBuilder` source incompatibility (3 occurrences)

**Affected Projects:** IntegrationTests
**Affected Files:** `Program.cs` (lines 43, 45)

**Issue:** `Microsoft.AspNetCore.Hosting.IWebHost` and `Microsoft.AspNetCore.WebHost` are source-incompatible in .NET 10. The `WebHost.CreateDefaultBuilder` hosting model is obsolete.

**Fix:** Migrate to `WebApplication.CreateBuilder` / `WebApplicationBuilder` pattern, or add appropriate compatibility references if the old pattern is still supported.

#### 7. `IdentityEntityFrameworkBuilderExtensions.AddEntityFrameworkStores` (2 occurrences)

**Affected Projects:** IntegrationTests
**Affected Files:** `Startup.cs` (line 37)

**Issue:** The `AddEntityFrameworkStores<TContext>` extension method signature may have changed in `Microsoft.AspNetCore.Identity.EntityFrameworkCore` 10.0.5.

**Fix:** Verify compilation after package update. Adjust method call if parameter types changed.

#### 8. `DependencyContext` / `CompilationLibrary` API (6 occurrences)

**Affected Projects:** Tests
**Affected Files:** `TestUtilities\BuildReference.cs` (line 35)

**Issue:** `DependencyContext.Default`, `DependencyContext.CompileLibraries`, and `CompilationLibrary.ResolveReferencePaths()` from `Microsoft.Extensions.DependencyModel` are source-incompatible in .NET 10.

**Fix:** Update to new API after upgrading to `Microsoft.Extensions.DependencyModel` 10.0.5. Verify property and method names still resolve.

#### 9. `string.Join` overload ambiguity (1 occurrence)

**Affected Projects:** IntegrationTests
**Affected Files:** `Tests\Models\GeneratedTypesTest.cs` (line 24)

**Issue:** `string.Join(string, ...)` with `params string[]` is now ambiguous with the new `ReadOnlySpan<string>` overload in .NET 10.

**Fix:** Explicitly create a `string[]`: `string.Join(", ", new[] { address, city, state, zip })`.

---

### Behavioral Changes (57 occurrences — may affect runtime behavior, no compilation errors)

#### 1. `System.Text.Json.JsonDocument` (53 occurrences)

**Affected Projects:** Json.Microsoft (10), FunctionalTests (44)
**Affected Files:**
- `src\EFCore.MySql.Json.Microsoft\Storage\ValueConversion\Internal\MySqlJsonMicrosoftJsonDocumentValueConverter.cs`
- `src\EFCore.MySql.Json.Microsoft\Storage\ValueConversion\Internal\MySqlJsonMicrosoftStringValueConverter.cs`
- `src\EFCore.MySql.Json.Microsoft\Storage\ValueConversion\Internal\MySqlJsonMicrosoftJsonElementValueConverter.cs`
- `test\EFCore.MySql.FunctionalTests\Storage\MySqlJsonMicrosoftTypeMappingTest.cs`
- `test\EFCore.MySql.FunctionalTests\Query\JsonMicrosoftDomQueryTest.cs`

**Impact:** `JsonDocument` behavior changes in .NET 10 may affect JSON parsing, serialization, or comparison. Monitor test results for unexpected failures in JSON-related tests.

#### 2. `ConsoleLoggerExtensions.AddConsole` (3 occurrences)

**Affected Projects:** IntegrationTests
**Affected Files:** `Program.cs` (line 21), `AppDbScope.cs` (line 13), `Startup.cs` (line 37)

**Impact:** Console logging behavior may change. No compilation error, but log output format or filtering behavior may differ.

#### 3. `ValueType.Equals` (1 occurrence)

**Impact:** Behavioral change in `ValueType.Equals` — monitor for equality comparison differences in tests.

## 9. Risk Management

### Risk Assessment Summary

| Project | Risk Level | Description | Mitigation |
|:---|:---|:---|:---|
| `EFCore.MySql` | 🟢 Low | No code changes, package update only | Automated build verification |
| `EFCore.MySql.Json.Microsoft` | 🟢 Low | Behavioral changes only (JsonDocument) | Runtime test validation |
| `EFCore.MySql.Json.Newtonsoft` | 🟢 Low | No API issues | Automated build verification |
| `EFCore.MySql.NTS` | 🟢 Low | No API issues | Automated build verification |
| `EFCore.MySql.FunctionalTests` | 🟢 Low | 32 source-incompatible (TimeSpan overloads — mechanical fixes) | Pattern-based search-and-replace |
| `EFCore.MySql.IntegrationTests` | 🟢 Low | 18 source-incompatible (WebHost, TimeSpan, string.Join) | Targeted code updates per catalog |
| `EFCore.MySql.Tests` | 🟢 Low | 6 source-incompatible (DependencyModel API) | Verify with updated package |
| `QueryBaselineUpdater` | 🟢 Low | No changes needed | Automated build verification |

### Key Risks

1. **EF Core 10.0 API compatibility**: The `Microsoft.EntityFrameworkCore.Relational.Specification.Tests` package is used by test projects. If EF Core 10.0 introduces breaking changes to test base classes, test projects may require additional adaptation beyond what the assessment identified.
   - **Mitigation**: Build and test iteratively; consult [EF Core 10.0 breaking changes documentation](https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-10.0/breaking-changes).

2. **`JsonDocument` behavioral changes**: 53 occurrences of behavioral changes may cause subtle test failures in JSON parsing/comparison.
   - **Mitigation**: Run JSON-related functional tests thoroughly; compare outputs before and after upgrade.

3. **`WebHost` → `WebApplication` migration**: The IntegrationTests `Program.cs` uses the legacy `WebHost.CreateDefaultBuilder` pattern which is source-incompatible in .NET 10.
   - **Mitigation**: Migrate to `WebApplication.CreateBuilder` pattern. Since this is a test project, the migration scope is limited.

### Contingency Plans

- **If EF Core 10.0 packages are not yet stable**: Pin to latest stable version available within the `[10.0.0,10.0.999]` range.
- **If `WebHost` migration is complex**: Consider suppressing the warning temporarily and using the compatibility shim if available, then address in a follow-up PR.
- **If tests fail due to behavioral changes**: Compare test output with .NET 9 baseline; update test expectations where the new behavior is correct.

---

## 10. Testing & Validation Strategy

### Test Projects

| Test Project | Type | Test Count (approx) | Dependencies |
|:---|:---|:---|:---|
| `EFCore.MySql.FunctionalTests` | xUnit | Large suite (270 files) | All src projects |
| `EFCore.MySql.IntegrationTests` | xUnit | Medium suite (42 files) | EFCore.MySql, Json.Newtonsoft |
| `EFCore.MySql.Tests` | xUnit | Medium suite (30 files) | EFCore.MySql, FunctionalTests |

### Validation Approach

**After Atomic Upgrade (build verification):**
- [ ] Solution restores without dependency conflicts
- [ ] Solution builds with 0 errors
- [ ] Solution builds with 0 new warnings (existing suppressed warnings excluded)

**Test Execution:**
- [ ] `EFCore.MySql.FunctionalTests` — all tests pass
- [ ] `EFCore.MySql.IntegrationTests` — all tests pass
- [ ] `EFCore.MySql.Tests` — all tests pass

**Behavioral Change Monitoring:**
- Pay attention to JSON-related test failures (JsonDocument changes)
- Pay attention to console logging behavior changes
- Pay attention to TimeSpan precision differences

> **Note**: Integration tests require a running MySQL/MariaDB instance. Ensure test database is available before running the IntegrationTests project.

---

## 11. Complexity & Effort Assessment

| Project | Complexity | Rationale |
|:---|:---|:---|
| `EFCore.MySql` | Low | Config-only change, no code modifications |
| `EFCore.MySql.Json.Microsoft` | Low | Config-only change, behavioral monitoring only |
| `EFCore.MySql.Json.Newtonsoft` | Low | Config-only change, no API issues |
| `EFCore.MySql.NTS` | Low | Config-only change, no API issues |
| `EFCore.MySql.FunctionalTests` | Low | Mechanical TimeSpan overload fixes (pattern-based) |
| `EFCore.MySql.IntegrationTests` | Low-Medium | WebHost migration + TimeSpan + string.Join fixes |
| `EFCore.MySql.Tests` | Low | DependencyModel API update (1 file) |
| `QueryBaselineUpdater` | Low | Config-only change |

### Overall Assessment
- The upgrade is straightforward due to centralized configuration
- Most code changes are mechanical (TimeSpan overload disambiguation)
- The only non-trivial change is the `WebHost` → `WebApplication` migration in IntegrationTests `Program.cs`
- No production (`src`) code changes are needed beyond framework/package version bumps

---

## 12. Source Control Strategy

### Branch Strategy
- **Source branch**: `main`
- **Upgrade branch**: `upgrade-to-NET10`
- All changes committed to the `upgrade-to-NET10` branch

### Commit Strategy

Single commit for the entire atomic upgrade:

**Commit message**: `Upgrade to .NET 10.0 (LTS)`

**Contents**:
- `global.json` — SDK version update
- `Directory.Build.props` — framework property updates
- `Directory.Packages.props` — package version updates
- All code files with breaking change fixes

**Rationale**: All-At-Once strategy — a single commit captures the entire upgrade as one atomic unit, making it easy to review, revert, or cherry-pick.

### Review & Merge Process
- Create pull request from `upgrade-to-NET10` → `main`
- Ensure CI pipeline passes (build + tests)
- Review all code changes, focusing on breaking change fixes
- Squash merge to maintain clean history

---

## 13. Success Criteria

### Technical Criteria
- [ ] All 8 projects target `net10.0`
- [ ] All 13 package updates applied per §[7. Package Update Reference](#7-package-update-reference)
- [ ] `global.json` updated to .NET 10.0 SDK
- [ ] Solution builds with 0 errors
- [ ] All 3 test projects pass
- [ ] No package dependency conflicts
- [ ] No security vulnerabilities

### Quality Criteria
- [ ] No new compiler warnings introduced
- [ ] All breaking changes resolved per §[8. Breaking Changes Catalog](#8-breaking-changes-catalog)
- [ ] Behavioral changes monitored through test execution

### Process Criteria
- [ ] All-At-Once strategy followed (single atomic upgrade)
- [ ] Single commit on `upgrade-to-NET10` branch
- [ ] Plan.md followed as specification
