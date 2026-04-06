Student Management System
A multi-project ASP.NET Core solution for managing student records. Includes a RESTful Web API backend and a server-rendered MVC frontend, built on .NET 8 with Entity Framework Core, JWT authentication, and Serilog logging.
---
Solution Structure
```
Student/
├── Student/            # ASP.NET Core Web API
├── Student.Data/       # Data access layer (EF Core, Repository, UnitOfWork)
├── Student.Model/      # Shared entity models
├── StudentMvc/         # ASP.NET Core MVC client
└── tools/Encryptor/    # CLI utility for encrypting connection strings
```
Request flow:
`Controller → IStudentService → IStudentRepository → StudentDbContext → Database`
---
Prerequisites
.NET 8 SDK
EF Core CLI tools: `dotnet tool install --global dotnet-ef`
SQL Server (optional — defaults to SQLite for local development)
---
Getting Started
1. Build the solution
```bash
dotnet build Student.sln
```
2. Run the API
```bash
dotnet run --project Student/Student.csproj
```
The API starts on `https://localhost:7001` by default. Swagger UI is available at `/swagger`.
3. Run the MVC client
Update `StudentMvc/appsettings.json` to match the API port:
```json
{
  "StudentApi": {
    "BaseUrl": "https://localhost:7001/"
  }
}
```
Then run:
```bash
dotnet run --project StudentMvc/StudentMvc.csproj
```
---
Database
The API auto-migrates on startup. SQLite (`students.db`) is used for local development. SQL Server is used when a `Default` connection string is configured.
Manual migration commands
```bash
# Add a new migration
dotnet ef migrations add <MigrationName> \
  --project Student.Data/Student.Data.csproj \
  --startup-project Student/Student.csproj

# Apply migrations
dotnet ef database update \
  --project Student.Data/Student.Data.csproj \
  --startup-project Student/Student.csproj
```
---
Authentication
All write endpoints require a JWT bearer token.
Login endpoint: `POST /api/auth/login`
```json
{ "username": "admin", "password": "password" }
```
Returns a JWT valid for 1 hour. Include it as `Authorization: Bearer <token>` on protected requests.
> **Note:** The hardcoded credentials and JWT signing key (`ThisIsADevelopmentKeyDontUseInProduction`) are for development only and must not be used in production.
---
API Endpoints
Method	Endpoint	Auth Required	Description
POST	`/api/auth/login`	No	Obtain JWT token
GET	`/api/students`	No	List all students
GET	`/api/students/{id}`	Yes	Get student by ID
POST	`/api/students`	Yes	Create a student
PUT	`/api/students/{id}`	Yes	Update a student
DELETE	`/api/students/{id}`	Yes	Delete a student
---
Student Model
Field	Type	Constraints
Id	int	Primary key, auto-increment
Name	string	Required, max 200 chars
Email	string	Required, max 200 chars, unique
Age	int	Required
Course	string	Required, max 200 chars
CreatedDate	DateTime	Auto-set to UTC now
---
Encrypting Connection Strings
The API can decrypt an AES-encrypted connection string from `EncryptedConnection` in `appsettings.json`. To generate one:
```bash
dotnet run --project tools/Encryptor/Encryptor.csproj -- "<plaintext-connection-string>"
```
Paste the output into `Student/appsettings.json`:
```json
{
  "EncryptedConnection": "<encrypted-value>"
}
```
Set the `ENCRYPTION_KEY` environment variable to the key used for encryption.
---
Project Details
Student.Model
Pure POCO class library with no dependencies. Defines the `Student` entity with validation attributes.
Student.Data
Data access layer implementing:
Repository pattern — `IStudentRepository` / `EfStudentRepository`
Unit of Work pattern — `IUnitOfWork` / `EfUnitOfWork` (Begin / Commit / Rollback)
EF Core DbContext — `StudentDbContext` with `StudentConfiguration` (fluent API constraints)
Student (Web API)
JWT authentication (HS256, 1-hour expiry)
Global exception handling middleware (stack trace in Development only)
Serilog structured logging (console + daily rolling file under `Logs/`)
Swagger UI with JWT authorization support
Auto-migration on startup
StudentMvc (MVC Client)
Server-rendered views for listing, creating, editing, and deleting students
Retrieves JWT from the API and stores it in a cookie via `ApiTokenController`
Uses `IHttpClientFactory` for API calls
---
Configuration
API — `Student/appsettings.json`
```json
{
  "Jwt": {
    "Key": "ThisIsADevelopmentKeyDontUseInProduction",
    "Issuer": "MyApi",
    "Audience": "MyApiUsers"
  },
  "ConnectionStrings": {
    "Default": ""
  },
  "EncryptedConnection": ""
}
```
MVC — `StudentMvc/appsettings.json`
```json
{
  "StudentApi": {
    "BaseUrl": "https://localhost:7001/"
  }
}
```
---
Production Considerations
Replace the JWT signing key with a strong, secret value stored in a secure vault.
Replace hardcoded credentials with a proper identity provider.
Restrict `AllowedHosts` from `*` to your actual domain.
Store the MVC token in a secure, HTTP-only cookie.
Use SQL Server (or another production-grade database) instead of SQLite.
Set the `ENCRYPTION_KEY` environment variable for connection string decryption.
