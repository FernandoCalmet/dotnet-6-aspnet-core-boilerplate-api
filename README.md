# 🦄 DOTNET 6 ASPNET CORE BOILERPLATE API

[![Github][github-shield]][github-url]
[![Kofi][kofi-shield]][kofi-url]
[![LinkedIn][linkedin-shield]][linkedin-url]
[![Khanakat][khanakat-shield]][khanakat-url]

## TABLA DE CONTENIDO

* [Acerca del proyecto](#acerca-del-proyecto)
* [Características](#características)
* [Instalación](#instalación)
* [Descripción](#descripción)
* [Dependencias](#dependencias)
* [Licencia](#licencia)

## 🔥 ACERCA DEL PROYECTO

Este proyecto es una muestra de un boilerplate con CRUD + Login con Verificación + Autorización basado en roles  + JWT Autenticación + Recuperación de Contraseñas . Se utilizo ASP.NET Core 6 Web API.


## ✔️ CARACTERÍSTICAS

- [x] JWT authentication with refresh tokens
- [x] Email sign up and verification`
- [x] Forgot password and reset password functionality
- [x] Role based authorization with support for two roles (``User`` & ``Admin``)
- [x] Account management (CRUD) routes with role based access control
- [x] Swagger API documentation route

## ⚙️ INSTALACIÓN

Clonar el repositorio.

```bash
gh repo clone FernandoCalmet/DOTNET-6-ASPNET-Core-Boilerplate-API
```

Migrar base de datos

```bash
dotnet ef database update
```

Ejecutar aplicación.

```bash
dotnet run
```

### OTROS COMANDOS

Iniciar migración

```bash
dotnet ef migrations add InitialCreate --context DataContext --output-dir Migrations/SqlServerMigrations
```

## DESCRIPCIÓN

La API boilerplate le permite registrar una cuenta de usuario, iniciar sesión y realizar diferentes acciones según su función. El rol `Admin` tiene acceso completo para administrar (agregar/editar/eliminar) cualquier cuenta en el sistema, el rol `User` tiene acceso para actualizar/eliminar su propia cuenta. A la primera cuenta registrada se le asigna automáticamente el rol `Admin` y a los registros posteriores se les asigna el rol `User`.

Al registrarse, la API envía un correo electrónico de verificación con un token e instrucciones a la dirección de correo electrónico de la cuenta, las cuentas deben verificarse antes de que puedan autenticarse. La configuración de SMTP para el correo electrónico se configura en `appsettings.json` . Si no tiene un servicio SMTP, para una prueba rápida puede usar el servicio SMTP falso https://ethereal.email/ para crear una bandeja de entrada temporal, simplemente haga clic en Crear cuenta Ethereal y copie las opciones de configuración de SMTP.

### Descripción general de la implementación de la autenticación

La autenticación se implementa con tokens de acceso JWT y tokens de actualización. En una autenticación exitosa, la API devuelve un token de acceso JWT de corta duración que caduca después de 15 minutos y un token de actualización que caduca después de 7 días en una cookie HTTP Only. El JWT se usa para acceder a rutas seguras en la API y el token de actualización se usa para generar nuevos tokens de acceso JWT cuando (o justo antes) caducan.

Las cookies HTTP solo se utilizan para tokens de actualización para aumentar la seguridad porque no son accesibles para javascript del lado del cliente, lo que evita ataques XSS (secuencias de comandos entre sitios). Los tokens de actualización solo tienen acceso para generar nuevos tokens JWT (a través de la ruta `/accounts/refresh-token`), no pueden realizar ninguna otra acción segura que evite que se usen en ataques CSRF (falsificación de solicitud entre sitios).

### Puntos finales de la API

La API de .NET 6 de ejemplo tiene los siguientes puntos finales/rutas para demostrar el registro y la verificación de correo electrónico, la autenticación y la autorización basada en roles, la actualización y revocación de tokens, el olvido de la contraseña y el restablecimiento de la contraseña, y las rutas seguras de administración de cuentas:

- **POST** `/accounts/authenticate` : ruta pública que acepta solicitudes POST que contienen un correo electrónico y una contraseña en el cuerpo. En caso de éxito, se devuelve un token de acceso JWT con detalles básicos de la cuenta y una cookie HTTP Only que contiene un token de actualización.

- **POST** `/accounts/refresh-token` : ruta pública que acepta solicitudes POST que contienen una cookie con un token de actualización. En caso de éxito, se devuelve un nuevo token de acceso JWT con detalles básicos de la cuenta y una cookie HTTP Only que contiene un nuevo token de actualización (consulte la rotación del token de actualización justo debajo para obtener una explicación).

- **POST** `/accounts/revoke-token` : ruta segura que acepta solicitudes POST que contienen un token de actualización en el cuerpo de la solicitud o en una cookie, si ambos están presentes, se da prioridad al cuerpo de la solicitud. En caso de éxito, el token se revoca y ya no se puede usar para generar nuevos tokens de acceso JWT.

- **POST** `/accounts/register` : ruta pública que acepta solicitudes POST que contienen detalles de registro de cuenta. En caso de éxito, la cuenta se registra y se envía un correo electrónico de verificación a la dirección de correo electrónico de la cuenta, las cuentas deben verificarse antes de que puedan autenticarse.

- **POST** `/accounts/verify-email` : ruta pública que acepta solicitudes POST que contienen un token de verificación de cuenta. En caso de éxito, la cuenta se verifica y ahora puede iniciar sesión.

- **POST** `/accounts/forgot-password` : ruta pública que acepta solicitudes POST que contienen una dirección de correo electrónico de cuenta. En caso de éxito, se envía un correo electrónico de restablecimiento de contraseña a la dirección de correo electrónico de la cuenta. El correo electrónico contiene un token de reinicio de un solo uso que es válido por un día.

- **POST** `/accounts/validate-reset-token` : ruta pública que acepta solicitudes POST que contienen un token de restablecimiento de contraseña. Se devuelve un mensaje para indicar si el token es válido o no.

- **POST** `/accounts/reset-password` : ruta pública que acepta solicitudes POST que contienen un token de restablecimiento, una contraseña y una contraseña de confirmación. En caso de éxito, la contraseña de la cuenta se restablece.

- **GET** `/accounts` : ruta segura restringida al Adminrol que acepta solicitudes GET y devuelve una lista de todas las cuentas en la aplicación.

- **POST** `/accounts` : ruta segura restringida al Adminrol que acepta solicitudes POST que contienen nuevos detalles de cuenta. En caso de éxito, la cuenta se crea y se verifica automáticamente.

- **GET** `/accounts/{id}` : ruta segura que acepta solicitudes GET y devuelve los detalles de la cuenta con la identificación especificada. El Adminrol puede acceder a cualquier cuenta, el Userrol solo puede acceder a su propia cuenta.

- **PUT** `/accounts/{id}` : ruta segura que acepta solicitudes PUT para actualizar los detalles de la cuenta con la identificación especificada. El Adminrol puede actualizar cualquier cuenta, incluido su rol, el Userrol solo puede actualizar sus propios detalles de cuenta, excepto el rol.

- **DELETE** `/accounts/{id}` : ruta segura que acepta solicitudes de ELIMINACIÓN para eliminar la cuenta con la identificación especificada. El Adminrol puede eliminar cualquier cuenta, el Userrol solo puede eliminar su propia cuenta.

### Actualizar rotación de tokens

Cada vez que se usa un token de actualización para generar un nuevo token JWT (a través de la ruta `/accounts/refresh-token`), el token de actualización se revoca y se reemplaza por un nuevo token de actualización. Esta técnica se conoce como rotación de tokens de actualización y aumenta la seguridad al reducir la vida útil de los tokens de actualización, lo que hace que sea menos probable que un token comprometido sea válido (o válido por mucho tiempo). Cuando se rota un token de actualización, el nuevo token se guarda en el campo `ReplacedByToken` del token revocado para crear un registro de auditoría en la base de datos.

Los registros de tokens de actualización revocados y caducados se conservan en la base de datos durante el número de días establecido en la propiedad `RefreshTokenTTL` en el archivo `appsettings.json` . El valor predeterminado es de 2 días, después de los cuales el `servicio de cuenta` elimina los tokens inactivos antiguos en los métodos `Authenticate()` y `RefreshToken()`.

### Detección de reutilización de tokens revocados

Si se intenta generar un nuevo token JWT utilizando un token de actualización revocado, la API lo trata como un usuario potencialmente malicioso con un token de actualización robado (revocado), o un usuario válido que intenta acceder al sistema después de que su token haya sido revocado. por un usuario malintencionado con un token de actualización robado (activo). En cualquier caso, la API revoca todos los tokens descendientes porque es probable que el token y sus descendientes se hayan creado en el mismo dispositivo que puede haberse visto comprometido. El motivo de la revocación se registra en `"Attempted reuse of revoked ancestor token"` comparación con los tokens revocados en la base de datos.

### Instalación y configuración de base de datos SQL

Para tratar de simplificar las cosas, la API repetitiva utiliza una base de datos SQLite, SQLite es independiente y no requiere la instalación de un servidor de base de datos completo. La base de datos se crea automáticamente al iniciarse en el archivo Program.cs al activar la ejecución de las migraciones de EF Core en la carpeta `/Migrations`.

## 📥 DEPENDENCIAS

- [Swashbuckle.AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore/) : Herramientas Swagger para documentar API creadas en ASP.NET Core.
- [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/) : Entity Framework Core es un moderno mapeador de bases de datos de objetos para .NET. Admite consultas LINQ, seguimiento de cambios, actualizaciones y migraciones de esquemas. EF Core funciona con SQL Server, Azure SQL Database, SQLite, Azure Cosmos DB, MySQL, PostgreSQL y otras bases de datos a través de una API de complemento de proveedor.
- [Microsoft.EntityFrameworkCore.Design](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Design/) : Proveedor de base de datos de Microsoft SQL Server para Entity Framework Core.
- [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/) : Componentes de tiempo de diseño compartidos para las herramientas de Entity Framework Core.
- [Microsoft.EntityFrameworkCore.Sqlite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite/) : Proveedor de base de datos SQLite para Entity Framework Core.
- [Microsoft.AspNetCore.Authentication.JwtBearer](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer/) : Middleware ASP.NET Core que permite que una aplicación reciba un token de portador de OpenID Connect.
- [AutoMapper](https://www.nuget.org/packages/AutoMapper/) : AutoMapper es una pequeña biblioteca simple creada para resolver un problema aparentemente complejo: deshacerse del código que mapeó un objeto a otro. Este tipo de código es bastante triste y aburrido de escribir, entonces, ¿por qué no inventar una herramienta que lo haga por nosotros?
- [AutoMapper.Extensions.Microsoft.DependencyInjection](https://www.nuget.org/packages/AutoMapper.Extensions.Microsoft.DependencyInjection/) : Extensiónes para AutoMapper ASP.NET Core.
- [BCrypt.Net-Next](https://www.nuget.org/packages/BCrypt.Net-Next/) : Segmentación de NET 6.
- [System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt/) : Incluye tipos que brindan soporte para crear, serializar y validar tokens web JSON.
-[MailKit](https://www.nuget.org/packages/MailKit/) : MailKit es una biblioteca de cliente de correo .NET multiplataforma de código abierto que se basa en MimeKit y está optimizada para dispositivos móviles.

## 📄 LICENCIA

Este proyecto está bajo la Licencia (Licencia MIT) - mire el archivo [LICENSE](LICENSE) para más detalles.

## ⭐️ DAME UNA ESTRELLA

Si esta Implementación le resultó útil o la utilizó en sus Proyectos, déle una estrella. ¡Gracias! O, si te sientes realmente generoso, [¡Apoye el proyecto con una pequeña contribución!](https://ko-fi.com/fernandocalmet).

<!--- reference style links --->
[github-shield]: https://img.shields.io/badge/-@fernandocalmet-%23181717?style=flat-square&logo=github
[github-url]: https://github.com/fernandocalmet
[kofi-shield]: https://img.shields.io/badge/-@fernandocalmet-%231DA1F2?style=flat-square&logo=kofi&logoColor=ff5f5f
[kofi-url]: https://ko-fi.com/fernandocalmet
[linkedin-shield]: https://img.shields.io/badge/-fernandocalmet-blue?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/fernandocalmet
[linkedin-url]: https://www.linkedin.com/in/fernandocalmet
[khanakat-shield]: https://img.shields.io/badge/khanakat.com-brightgreen?style=flat-square
[khanakat-url]: https://khanakat.com