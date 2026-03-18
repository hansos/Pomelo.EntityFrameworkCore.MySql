
## [2026-03-18 18:21] TASK-001: Verify prerequisites

Status: Complete

- **Verified**: .NET 10.0 SDK is installed (Compatible SDK found); global.json rollForward updated to latestMajor for .NET 10 compatibility
- **Files Modified**: global.json (rollForward changed to latestMajor by SDK validation tool)

### Outcome
Success - Prerequisites verified: .NET 10.0 SDK available and global.json compatible.


## [2026-03-18 18:42] TASK-002: Atomic framework and package upgrade with compilation fixes

Status: Paused

- **Files Modified**: 
  - `global.json` — SDK version updated to 10.0.100, rollForward to latestFeature
  - `Directory.Build.props` — All 6 framework properties updated to net10.0
  - `Directory.Packages.props` — EFCoreVersion to [10.0.0,10.0.999], 17 individual packages updated, plus xunit and Microsoft.CodeAnalysis packages updated for transitive dependency compatibility
  - `src/EFCore.MySql/Query/ExpressionVisitors/Internal/MySqlBoolOptimizingExpressionVisitor.cs` — Added VisitRightJoin method
  - `src/EFCore.MySql/Query/Internal/MySqlParameterBasedSqlProcessor.cs` — Migrated Optimize→Process, ProcessSqlNullability→ParametersCacheDecorator API
  - `src/EFCore.MySql/Query/Internal/SkipTakeCollapsingExpressionVisitor.cs` — Updated to ParametersCacheDecorator
  - `src/EFCore.MySql/Query/ExpressionVisitors/Internal/MySqlParameterInliningExpressionVisitor.cs` — Updated to ParametersCacheDecorator
  - `src/EFCore.MySql/Query/Internal/MySqlQueryableMethodTranslatingExpressionVisitor.cs` — Added [Obsolete] attribute
  - `src/EFCore.MySql/Query/Internal/MySqlQueryCompilationContext.cs` — Removed 5-param constructor
  - `src/EFCore.MySql/Query/Internal/MySqlQueryCompilationContextFactory.cs` — Updated CreatePrecompiled
  - `src/EFCore.MySql/Extensions/MySqlDbContextOptionsBuilderExtensions.cs` — TranslateParameterizedCollectionsToConstants→UseParameterizedCollectionMode
  - `src/EFCore.MySql/Query/ExpressionTranslators/Internal/MySqlStringComparisonMethodTranslator.cs` — Fixed QueryParameterPrefix removal and ParameterValues→Parameters rename
  - `src/Shared/Check.cs` — Fixed AbstractionsStrings method→property changes
- **Verified**: All 4 src projects (EFCore.MySql, Json.Microsoft, Json.Newtonsoft, NTS) and tools/QueryBaselineUpdater build successfully
- **Errors Fixed**: 18 EF Core 10 breaking changes in production code fixed
- **Remaining**: 451 errors in test projects (EF Core 10 test infrastructure API changes - renamed/removed/signature-changed methods in base test classes)

Paused - Production code complete. Test projects require extensive EF Core 10 test base class API migration.

