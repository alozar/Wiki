# Swagger

## Подмена пути "api/v1/" на "api/" в сваггер-документе v1
```csharp
public class ReplaceV1WithEmptyInPath : IDocumentFilter
{
    /// <inheritdoc/>
    public void Apply(OpenApiDocument swaggerDoc, DocumentFilterContext context)
    {
        OpenApiPaths paths = [];

        foreach (KeyValuePair<string, OpenApiPathItem> pair in swaggerDoc.Paths)
        {
            paths.Add(pair.Key.Replace("v1/", string.Empty), pair.Value);
        }

        swaggerDoc.Paths = paths;
    }
}
```

## Распределяем все маршруты по версиям сваггер-документов.
```csharp
private static bool SwaggerDocumentPredicate(string version, ApiDescription apiDescription)
{
    // Выводим только контроллеры из нашей сборки 
    if (!apiDescription.ActionDescriptor.DisplayName
            .StartsWith("my namespace", StringComparison.OrdinalIgnoreCase))
    {
        return false;
    }

    // Если один и тот же метод помечен для вывода в v1 и v2, то выводим с нужным RelativePath
    var matched = $"v{apiDescription.GetApiVersion()?.MajorVersion}" == version;

    return matched;
}
```

## Настройка AddSwaggerGen
```csharp
services.AddSwaggerGen(swaggerGenOptions =>
{
    swaggerGenOptions.SwaggerDoc("v1", new OpenApiInfo {...});

    swaggerGenOptions.SwaggerDoc("v2", new OpenApiInfo {...});

    OpenApiSecurityScheme openApiSecurityScheme = new()
    {
        Name = HeaderNames.Authorization,
        Description = "Авторизация по токену. Введите значение в формате: `Bearer <token>`",
        Type = SecuritySchemeType.ApiKey,
        In = ParameterLocation.Header,
        Scheme = "Bearer",
        BearerFormat = "JWT",
        Reference = new OpenApiReference
        {
            Id = "Bearer",
            Type = ReferenceType.SecurityScheme
        }
    };
    swaggerGenOptions.AddSecurityDefinition("Bearer", openApiSecurityScheme);
    swaggerGenOptions.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            openApiSecurityScheme,
            new List<string>()
        }
    });

    // Нужно чтобы сваггер хавал модели с одинаковыми названиями, но в разных namespace
    swaggerGenOptions.CustomSchemaIds(x => x.FullName);

    swaggerGenOptions.DocumentFilter<ReplaceV1WithEmptyInPath>();

    swaggerGenOptions.DocInclusionPredicate(SwaggerDocumentPredicate);
});
```

## Настройка UseSwagger
```csharp
app.UseSwagger();
app.UseSwaggerUI(options =>
{
    options.SwaggerEndpoint("/swagger/v1/swagger.json", "...");
    options.SwaggerEndpoint("/swagger/v2/swagger.json", "...");
    options.RoutePrefix = "swagger";
});
```