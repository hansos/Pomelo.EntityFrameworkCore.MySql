# Fork Changes (.NET 10 / EF Core 10)

This page summarizes the key changes introduced in this fork.

## Purpose

This fork exists to provide a temporary `.NET 10` and `EF Core 10` compatible version while waiting for an official upstream release.

## Technical Changes

### Runtime and SDK

- Solution target framework updated to `.NET 10` (`net10.0`).
- SDK pinning updated to `.NET SDK 10.0.100` in `global.json`.

### EF Core and Related Dependencies

- Central package versioning updated to `EF Core 10.0.5`.
- ASP.NET Core / `Microsoft.Extensions.*` dependencies aligned to `10.0.5` where applicable.
- `System.Text.Json` aligned to `10.0.5`.

### Provider and Supporting Packages

- `MySqlConnector` remains at `2.4.0`.
- Main package line documented as `Pomelo.EntityFrameworkCore.MySql 10.x`.
- Companion package documentation aligned:
  - `Pomelo.EntityFrameworkCore.MySql.Json.Microsoft`
  - `Pomelo.EntityFrameworkCore.MySql.Json.Newtonsoft`
  - `Pomelo.EntityFrameworkCore.MySql.NetTopologySuite`

## Documentation Updates

- Root `README.md` updated with:
  - `.NET 10` / `EF Core 10` compatibility info
  - temporary fork status
  - updated package reference examples (`10.0.0`)
- Package-level docs updated with compatibility notes for `10.x`.

## Validation Status

- Library compiles successfully in this fork.
- A prerelease NuGet package has been created.
- The package line is already used in an application under development and tested in that context.

## Notes

- This fork is intended as a bridge until official upstream support is available.
- Follow upstream releases to migrate back when an official `.NET 10` version is published.
