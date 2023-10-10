# Скрипты

## Cоздание первой миграции и бд
```
Add-Migration -Verbose -Name "InitMigration" -Project "nameProject" -StartupProject "nameStartupProject"
Update-Database -Verbose -Context "nameContext" -Connection "Host=localhost;Port=port;Database=nameDB;Username=user;Password=pass" -Project "nameProject" -StartupProject "nameStartupProject"
```
[Entity Framework Core tools reference - Package Manager Console in Visual Studio](https://learn.microsoft.com/en-us/ef/core/cli/powershell)
