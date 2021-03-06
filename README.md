# 馃 DOTNET 6 ASPNET CORE BOILERPLATE API

[![Github][github-shield]][github-url]
[![Kofi][kofi-shield]][kofi-url]
[![LinkedIn][linkedin-shield]][linkedin-url]
[![Khanakat][khanakat-shield]][khanakat-url]

## TABLA DE CONTENIDO

* [Acerca del proyecto](#acerca-del-proyecto)
* [Caracter铆sticas](#caracter铆sticas)
* [Instalaci贸n](#instalaci贸n)
* [Descripci贸n](#descripci贸n)
* [Dependencias](#dependencias)
* [Licencia](#licencia)

## 馃敟 ACERCA DEL PROYECTO

Este proyecto es una muestra de un boilerplate con CRUD + Login con Verificaci贸n + Autorizaci贸n basado en roles  + JWT Autenticaci贸n + Recuperaci贸n de Contrase帽as . Se utilizo ASP.NET Core 6 Web API.


## 鉁旓笍 CARACTER脥STICAS

- [x] JWT authentication with refresh tokens
- [x] Email sign up and verification`
- [x] Forgot password and reset password functionality
- [x] Role based authorization with support for two roles (``User`` & ``Admin``)
- [x] Account management (CRUD) routes with role based access control
- [x] Swagger API documentation route

## 鈿欙笍 INSTALACI脫N

Clonar el repositorio.

```bash
gh repo clone FernandoCalmet/dotnet-6-aspnet-core-boilerplate-api
```

Migrar base de datos

```bash
dotnet ef database update
```

Ejecutar aplicaci贸n.

```bash
dotnet run
```

### OTROS COMANDOS

Iniciar migraci贸n

```bash
dotnet ef migrations add InitialCreate --context DataContext --output-dir Migrations/SqlServerMigrations
```

## DESCRIPCI脫N

La API boilerplate le permite registrar una cuenta de usuario, iniciar sesi贸n y realizar diferentes acciones seg煤n su funci贸n. El rol `Admin` tiene acceso completo para administrar (agregar/editar/eliminar) cualquier cuenta en el sistema, el rol `User` tiene acceso para actualizar/eliminar su propia cuenta. A la primera cuenta registrada se le asigna autom谩ticamente el rol `Admin` y a los registros posteriores se les asigna el rol `User`.

Al registrarse, la API env铆a un correo electr贸nico de verificaci贸n con un token e instrucciones a la direcci贸n de correo electr贸nico de la cuenta, las cuentas deben verificarse antes de que puedan autenticarse. La configuraci贸n de SMTP para el correo electr贸nico se configura en `appsettings.json` . Si no tiene un servicio SMTP, para una prueba r谩pida puede usar el servicio SMTP falso https://ethereal.email/ para crear una bandeja de entrada temporal, simplemente haga clic en Crear cuenta Ethereal y copie las opciones de configuraci贸n de SMTP.

### Descripci贸n general de la implementaci贸n de la autenticaci贸n

La autenticaci贸n se implementa con tokens de acceso JWT y tokens de actualizaci贸n. En una autenticaci贸n exitosa, la API devuelve un token de acceso JWT de corta duraci贸n que caduca despu茅s de 15 minutos y un token de actualizaci贸n que caduca despu茅s de 7 d铆as en una cookie HTTP Only. El JWT se usa para acceder a rutas seguras en la API y el token de actualizaci贸n se usa para generar nuevos tokens de acceso JWT cuando (o justo antes) caducan.

Las cookies HTTP solo se utilizan para tokens de actualizaci贸n para aumentar la seguridad porque no son accesibles para javascript del lado del cliente, lo que evita ataques XSS (secuencias de comandos entre sitios). Los tokens de actualizaci贸n solo tienen acceso para generar nuevos tokens JWT (a trav茅s de la ruta `/accounts/refresh-token`), no pueden realizar ninguna otra acci贸n segura que evite que se usen en ataques CSRF (falsificaci贸n de solicitud entre sitios).

### Puntos finales de la API

La API de .NET 6 de ejemplo tiene los siguientes puntos finales/rutas para demostrar el registro y la verificaci贸n de correo electr贸nico, la autenticaci贸n y la autorizaci贸n basada en roles, la actualizaci贸n y revocaci贸n de tokens, el olvido de la contrase帽a y el restablecimiento de la contrase帽a, y las rutas seguras de administraci贸n de cuentas:

- **POST** `/accounts/authenticate` : ruta p煤blica que acepta solicitudes POST que contienen un correo electr贸nico y una contrase帽a en el cuerpo. En caso de 茅xito, se devuelve un token de acceso JWT con detalles b谩sicos de la cuenta y una cookie HTTP Only que contiene un token de actualizaci贸n.

- **POST** `/accounts/refresh-token` : ruta p煤blica que acepta solicitudes POST que contienen una cookie con un token de actualizaci贸n. En caso de 茅xito, se devuelve un nuevo token de acceso JWT con detalles b谩sicos de la cuenta y una cookie HTTP Only que contiene un nuevo token de actualizaci贸n (consulte la rotaci贸n del token de actualizaci贸n justo debajo para obtener una explicaci贸n).

- **POST** `/accounts/revoke-token` : ruta segura que acepta solicitudes POST que contienen un token de actualizaci贸n en el cuerpo de la solicitud o en una cookie, si ambos est谩n presentes, se da prioridad al cuerpo de la solicitud. En caso de 茅xito, el token se revoca y ya no se puede usar para generar nuevos tokens de acceso JWT.

- **POST** `/accounts/register` : ruta p煤blica que acepta solicitudes POST que contienen detalles de registro de cuenta. En caso de 茅xito, la cuenta se registra y se env铆a un correo electr贸nico de verificaci贸n a la direcci贸n de correo electr贸nico de la cuenta, las cuentas deben verificarse antes de que puedan autenticarse.

- **POST** `/accounts/verify-email` : ruta p煤blica que acepta solicitudes POST que contienen un token de verificaci贸n de cuenta. En caso de 茅xito, la cuenta se verifica y ahora puede iniciar sesi贸n.

- **POST** `/accounts/forgot-password` : ruta p煤blica que acepta solicitudes POST que contienen una direcci贸n de correo electr贸nico de cuenta. En caso de 茅xito, se env铆a un correo electr贸nico de restablecimiento de contrase帽a a la direcci贸n de correo electr贸nico de la cuenta. El correo electr贸nico contiene un token de reinicio de un solo uso que es v谩lido por un d铆a.

- **POST** `/accounts/validate-reset-token` : ruta p煤blica que acepta solicitudes POST que contienen un token de restablecimiento de contrase帽a. Se devuelve un mensaje para indicar si el token es v谩lido o no.

- **POST** `/accounts/reset-password` : ruta p煤blica que acepta solicitudes POST que contienen un token de restablecimiento, una contrase帽a y una contrase帽a de confirmaci贸n. En caso de 茅xito, la contrase帽a de la cuenta se restablece.

- **GET** `/accounts` : ruta segura restringida al Adminrol que acepta solicitudes GET y devuelve una lista de todas las cuentas en la aplicaci贸n.

- **POST** `/accounts` : ruta segura restringida al Adminrol que acepta solicitudes POST que contienen nuevos detalles de cuenta. En caso de 茅xito, la cuenta se crea y se verifica autom谩ticamente.

- **GET** `/accounts/{id}` : ruta segura que acepta solicitudes GET y devuelve los detalles de la cuenta con la identificaci贸n especificada. El Adminrol puede acceder a cualquier cuenta, el Userrol solo puede acceder a su propia cuenta.

- **PUT** `/accounts/{id}` : ruta segura que acepta solicitudes PUT para actualizar los detalles de la cuenta con la identificaci贸n especificada. El Adminrol puede actualizar cualquier cuenta, incluido su rol, el Userrol solo puede actualizar sus propios detalles de cuenta, excepto el rol.

- **DELETE** `/accounts/{id}` : ruta segura que acepta solicitudes de ELIMINACI脫N para eliminar la cuenta con la identificaci贸n especificada. El Adminrol puede eliminar cualquier cuenta, el Userrol solo puede eliminar su propia cuenta.

### Actualizar rotaci贸n de tokens

Cada vez que se usa un token de actualizaci贸n para generar un nuevo token JWT (a trav茅s de la ruta `/accounts/refresh-token`), el token de actualizaci贸n se revoca y se reemplaza por un nuevo token de actualizaci贸n. Esta t茅cnica se conoce como rotaci贸n de tokens de actualizaci贸n y aumenta la seguridad al reducir la vida 煤til de los tokens de actualizaci贸n, lo que hace que sea menos probable que un token comprometido sea v谩lido (o v谩lido por mucho tiempo). Cuando se rota un token de actualizaci贸n, el nuevo token se guarda en el campo `ReplacedByToken` del token revocado para crear un registro de auditor铆a en la base de datos.

Los registros de tokens de actualizaci贸n revocados y caducados se conservan en la base de datos durante el n煤mero de d铆as establecido en la propiedad `RefreshTokenTTL` en el archivo `appsettings.json` . El valor predeterminado es de 2 d铆as, despu茅s de los cuales el `servicio de cuenta` elimina los tokens inactivos antiguos en los m茅todos `Authenticate()` y `RefreshToken()`.

### Detecci贸n de reutilizaci贸n de tokens revocados

Si se intenta generar un nuevo token JWT utilizando un token de actualizaci贸n revocado, la API lo trata como un usuario potencialmente malicioso con un token de actualizaci贸n robado (revocado), o un usuario v谩lido que intenta acceder al sistema despu茅s de que su token haya sido revocado. por un usuario malintencionado con un token de actualizaci贸n robado (activo). En cualquier caso, la API revoca todos los tokens descendientes porque es probable que el token y sus descendientes se hayan creado en el mismo dispositivo que puede haberse visto comprometido. El motivo de la revocaci贸n se registra en `"Attempted reuse of revoked ancestor token"` comparaci贸n con los tokens revocados en la base de datos.

### Instalaci贸n y configuraci贸n de base de datos SQL

Para tratar de simplificar las cosas, la API repetitiva utiliza una base de datos SQLite, SQLite es independiente y no requiere la instalaci贸n de un servidor de base de datos completo. La base de datos se crea autom谩ticamente al iniciarse en el archivo Program.cs al activar la ejecuci贸n de las migraciones de EF Core en la carpeta `/Migrations`.

## 馃摜 DEPENDENCIAS

- [Swashbuckle.AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore/) : Herramientas Swagger para documentar API creadas en ASP.NET Core.
- [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/) : Entity Framework Core es un moderno mapeador de bases de datos de objetos para .NET. Admite consultas LINQ, seguimiento de cambios, actualizaciones y migraciones de esquemas. EF Core funciona con SQL Server, Azure SQL Database, SQLite, Azure Cosmos DB, MySQL, PostgreSQL y otras bases de datos a trav茅s de una API de complemento de proveedor.
- [Microsoft.EntityFrameworkCore.Design](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Design/) : Proveedor de base de datos de Microsoft SQL Server para Entity Framework Core.
- [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/) : Componentes de tiempo de dise帽o compartidos para las herramientas de Entity Framework Core.
- [Microsoft.EntityFrameworkCore.Sqlite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite/) : Proveedor de base de datos SQLite para Entity Framework Core.
- [Microsoft.AspNetCore.Authentication.JwtBearer](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer/) : Middleware ASP.NET Core que permite que una aplicaci贸n reciba un token de portador de OpenID Connect.
- [AutoMapper](https://www.nuget.org/packages/AutoMapper/) : AutoMapper es una peque帽a biblioteca simple creada para resolver un problema aparentemente complejo: deshacerse del c贸digo que mape贸 un objeto a otro. Este tipo de c贸digo es bastante triste y aburrido de escribir, entonces, 驴por qu茅 no inventar una herramienta que lo haga por nosotros?
- [AutoMapper.Extensions.Microsoft.DependencyInjection](https://www.nuget.org/packages/AutoMapper.Extensions.Microsoft.DependencyInjection/) : Extensi贸nes para AutoMapper ASP.NET Core.
- [BCrypt.Net-Next](https://www.nuget.org/packages/BCrypt.Net-Next/) : Segmentaci贸n de NET 6.
- [System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt/) : Incluye tipos que brindan soporte para crear, serializar y validar tokens web JSON.
-[MailKit](https://www.nuget.org/packages/MailKit/) : MailKit es una biblioteca de cliente de correo .NET multiplataforma de c贸digo abierto que se basa en MimeKit y est谩 optimizada para dispositivos m贸viles.

## 馃搫 LICENCIA

Este proyecto est谩 bajo la Licencia (Licencia MIT) - mire el archivo [LICENSE](LICENSE) para m谩s detalles.

## 猸愶笍 DAME UNA ESTRELLA

Si esta Implementaci贸n le result贸 煤til o la utiliz贸 en sus Proyectos, d茅le una estrella. 隆Gracias! O, si te sientes realmente generoso, [隆Apoye el proyecto con una peque帽a contribuci贸n!](https://ko-fi.com/fernandocalmet).

<!--- reference style links --->
[github-shield]: https://img.shields.io/badge/-@fernandocalmet-%23181717?style=flat-square&logo=github
[github-url]: https://github.com/fernandocalmet
[kofi-shield]: https://img.shields.io/badge/-@fernandocalmet-%231DA1F2?style=flat-square&logo=kofi&logoColor=ff5f5f
[kofi-url]: https://ko-fi.com/fernandocalmet
[linkedin-shield]: https://img.shields.io/badge/-fernandocalmet-blue?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/fernandocalmet
[linkedin-url]: https://www.linkedin.com/in/fernandocalmet
[khanakat-shield]: https://img.shields.io/badge/khanakat.com-brightgreen?style=flat-square
[khanakat-url]: https://khanakat.com
