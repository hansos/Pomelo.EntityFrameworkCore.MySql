## About

_Pomelo.EntityFrameworkCore.MySql.Json.Microsoft_ adds JSON support for `System.Text.Json` (the Microsoft JSON stack) to [Pomelo.EntityFrameworkCore.MySql](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql).

`Pomelo.EntityFrameworkCore.MySql.Json.Microsoft` `10.x` is intended for use with `Pomelo.EntityFrameworkCore.MySql` `10.x` (`EF Core 10.0.x` / `.NET 10.0+`).

## How to Use

```csharp
optionsBuilder.UseMySql(
    connectionString,
    serverVersion,
    options => options.UseMicrosoftJson())
```

## Related Packages

* [Pomelo.EntityFrameworkCore.MySql](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MySql)
* [System.Text.Json](https://www.nuget.org/packages/System.Text.Json)

## License

_Pomelo.EntityFrameworkCore.MySql.Json.Microsoft_ is released as open source under the [MIT license](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql/blob/main/LICENSE).

## Feedback

Bug reports and contributions are welcome at our [GitHub repository](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql).