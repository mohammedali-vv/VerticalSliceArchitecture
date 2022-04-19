# Vertical Slice Architecture

This project is an experiment trying to create a API solution template that uses Vertical Slice architecture style.

The Vertical Slice architecture style is about organizing code by features and vertical slices instead of organizing by technical concerns. It's about an idea of grouping code according to the business functionality and putting all the relevant code close together.
Vertical Slice architecture can be a starting point and can be evolved later when an application become more sophisticated:

> We can start simple (Transaction Script) and simply refactor to the patterns that emerges from code smells we see in the business logic. [Vertical slice architecture by Jimmy Bogard](https://jimmybogard.com/vertical-slice-architecture/).

## Technologies and patterns

This project repository is created based on [Clean Architecture solution template by Jason Taylor](https://github.com/jasontaylordev/CleanArchitecture), and it uses technology choices and application business logic from this template.

- [ASP.NET API with .NET 6](https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-6.0)
- CQRS with [MediatR](https://github.com/jbogard/MediatR)
- [FluentValidation](https://fluentvalidation.net/)
- [AutoMapper](https://automapper.org/)
- [Entity Framework Core 6](https://docs.microsoft.com/en-us/ef/core/)
- [NUnit](https://nunit.org/), [FluentAssertions](https://fluentassertions.com/), [Moq](https://github.com/moq)

Afterwards, the projects and architecture is refactored towards the Vertical slice architecture style.

## Purpose of this repository

Most applications start simple but they tend to change and evolve over time. Because of this, I wanted to create a simpler API solution template that focuses on the vertical slices architecture style.

Typically if I need to change a feature in an application, I end up touching different layers of the application and navigating through piles of projects, folders and files.

Goal is to stop thinking about horizontal layers and start thinking about vertical slices and organize code by **Features**. When the code is organized by feature you get the benefits of not having to jump around projects, folders and files. Things related to given features are placed close together.

When moving towards the vertical slices we stop thinking about layers and abstractions. The reason is the vertical slice doesn't necessarily need shared layer abstractions like repositories, services, controllers. We are more focused on concrete behavior implementation and what is the best solution to implements.

## Projects breakdown

The solution template is broken into 2 projects:

### Api

ASP.NET Web API project is an entry point to the application, but it doesn't have any controllers, all the controller actions are moved to the **Application** project features.

### Application

This projects contains contains all applications logic and shared concerns like Domain Entities, Infrastructure and other common concerns. All the business logic is placed in a `Feature` folders. Instead of having a file for basically controller, command/query, validator, handlers, models, I placed everything usually in one file and have all the relevant things close together.

## Getting started

1. Install the latest [.NET 6 SDK](https://dotnet.microsoft.com/download/dotnet/6.0)
2. Navigate to `src/Api` and run `dotnet run` to launch the back end (ASP.NET Core Web API) or via `dotnet run --project src/Api/Api.csproj`

### Build, test and publish application

CLI commands executed from the root folder.

```bash
# build
dotnet build

# run
dotnet run --project src/Api/Api.csproj

# run unit tests
dotnet test tests/Application.UnitTests/Application.UnitTests.csproj 

# run integrations tests (required database up and running)
dotnet test tests/Application.IntegrationTests/Application.IntegrationTests.csproj 

# publish
dotnet publish src/Api/Api.csproj --configuration Release 
```

### Database Configuration

The project is configured to use an in-memory database by default. So you can run the project without additional infrastructure (SQL Server)

If you would like to use SQL Server, you will need to update **Api/appsettings.json** as follows:

```json
  "UseInMemoryDatabase": false,
```

Verify that the **DefaultConnection** connection string within **appsettings.json** points to a valid SQL Server instance.

When you run the application the database will be automatically created (if necessary) and the latest migrations will be applied.

The solution I work with is running Azure SQL Edge in Docker as a database for local development and testing. In the following section, I'll describe the steps needed to setup SQL Edge:

#### Use Azure SQL Edge

Docker is a great way to play with SQL Server without the administrative overhead of managing an instance.

Azure SQL Edge is a way to test and develop using SQL Server locally, and have a consistent experience between machines, whether they're PCs running Windows, Intel-based Macs, or the new Apple silicon M1 (there are no Docker images for SQL Server that support ARM64 just yet).

We'll need Docker first to get started – easiest is to install [Docker Desktop](https://www.docker.com/products/docker-desktop/).

With Docker up and running, next we need to pull the most recent Azure SQL Edge container image:

```bash
sudo docker pull mcr.microsoft.com/azure-sql-edge:latest
```

We can create and start the appropriate container with the following command:

```bash
sudo docker run --cap-add SYS_PTRACE -e 'ACCEPT_EULA=1' -e 'MSSQL_SA_PASSWORD=yourStrong(!)Password' -p 1433:1433 --name azuresqledge -d mcr.microsoft.com/azure-sql-edge
```

I decided to use Azure SQL Edge as flavor of SQL Server in Docker, and enable local development and testing no matter which OS you're running.

#### Database Migrations

To use `dotnet-ef` for your migrations please add the following flags to your command (values assume you are executing from repository root)

- `--project src/Application` (optional if in this folder)
- `--startup-project src/Api`
- `--output-dir Infrastructure/Persistence/Migrations`

For example, to add a new migration from the root folder:

 ```shell
dotnet ef migrations add "InitialMigration" --project src/Application --startup-project src/Api --output-dir Infrastructure/Persistence/Migrations
```

and apply migration and update the database:

```shell
dotnet ef database update --project src/Application --startup-project src/Api   
```

## Inspired by

- [Clean Architecture solution template by Jason Taylor](https://github.com/jasontaylordev/CleanArchitecture)
- [Vertical slice architecture by Jimmy Bogard](https://jimmybogard.com/vertical-slice-architecture/)
- [Organize code by Feature using Vertical Slices by Derek Comartin](https://codeopinion.com/organizing-code-by-feature-using-vertical-slices/)

## License

This project is licensed with the [MIT license](./LICENSE).
