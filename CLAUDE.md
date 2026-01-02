# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **Responsible Travel Adviser (RTA)** platform - a sustainable travel assessment and management system built with ASP.NET Core 2.2. The repository currently contains documentation and action plans but **no actual source code**.

**Important**: This appears to be a documentation-only repository for a .NET Core application. The actual source code would be located elsewhere (likely `/Users/alexjeffries/Downloads/SustainableTravel-master/` based on the documentation).

## Technology Stack

- **Framework**: ASP.NET Core 2.2 (⚠️ End-of-Life - needs upgrade to .NET 6/8)
- **Database**: SQL Server with Entity Framework Core
- **Frontend**: Razor Views, jQuery, Bootstrap
- **Authentication**: JWT Bearer tokens, IdentityServer4, Facebook OAuth
- **API**: REST with Swagger/OpenAPI documentation
- **Caching**: Redis + In-Memory (EasyCaching)

## Common Commands

Based on the documentation, these commands should work when the actual source code is available:

### Build Commands
```bash
# Navigate to the solution root
cd /path/to/SustainableTravel-master

# Restore packages
dotnet restore

# Build entire solution
dotnet build SustainableTravel.sln

# Clean and rebuild
dotnet clean && dotnet restore && dotnet build
```

### Database Commands
```bash
# Install EF Core tools (one-time setup)
dotnet tool install --global dotnet-ef --version 2.2.0

# Navigate to web project
cd Presentation/Orca.Web

# Run database migrations
dotnet ef database update --project ../../Libraries/Orca.Data

# Create new migration (when making schema changes)
dotnet ef migrations add <MigrationName> --project ../../Libraries/Orca.Data
```

### Run Commands
```bash
# Run the main web application
cd Presentation/Orca.Web
dotnet run

# Run the API project
cd Presentation/Orca.Api
dotnet run
```

### Test Commands
**Note**: The documentation indicates no tests currently exist in the project.
```bash
# When tests are added, use:
dotnet test

# Run tests with coverage
dotnet test --collect:"XPlat Code Coverage"
```

## Architecture Overview

### Solution Structure
```
SustainableTravel/
├── Libraries/                    # Core Business Logic
│   ├── Orca.Core                # Domain entities, infrastructure
│   ├── Orca.Data                # EF Core, repositories, migrations
│   ├── Orca.Services            # Business services layer
│   └── Orca.Web.Framework       # MVC framework extensions
│
├── Presentation/                # User-Facing Applications
│   ├── Orca.Web                 # Main MVC web app + Admin area
│   └── Orca.Api                 # REST API project
│
└── Plugins/                     # Extensibility
    ├── Orca.Plugin.Api          # REST API plugin
    └── Orca.Plugin.ExternalAuth.Facebook
```

### Key Components

- **Multi-tenant Platform**: Supports multiple websites and languages
- **Plugin Architecture**: Extensible via plugins for additional functionality
- **Questionnaire System**: Core feature for sustainability assessments
- **Admin Dashboard**: Management interface for operators
- **Travel Operator Portal**: Dedicated interface for tour companies

## Environment Setup

### Prerequisites
1. .NET Core 2.2 SDK (recommended to upgrade to .NET 6/8)
2. SQL Server (SQL Server Express acceptable for development)
3. Visual Studio 2019+ or VS Code with C# extension

### Configuration Files
- `Presentation/Orca.Web/appsettings.Development.json` - Development database connection
- `Presentation/Orca.Web/App_Data/dataSettings.json` - Runtime database settings
- `Presentation/Orca.Api/appsettings.json` - API configuration including JWT settings

### Security Requirements
⚠️ **CRITICAL**: Before any development work:
1. Change all exposed passwords in configuration files
2. Generate new JWT secret (replace "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb")
3. Set up User Secrets for development environment
4. Remove hardcoded production server references

## Development Workflow

### Getting Started
1. Follow the [QUICK_START_CHECKLIST.md](doc/QUICK_START_CHECKLIST.md) for 2-3 hour setup
2. Review [SECURITY_CHECKLIST.md](doc/SECURITY_CHECKLIST.md) for critical security fixes
3. Use [ACTION_PLAN.md](ACTION_PLAN.md) as the comprehensive development roadmap

### Current Status
- **Platform Completion**: ~70%
- **Missing**: Documentation, Tests, DevOps setup, Production configuration
- **Current Priority**: Security fixes and local environment setup

### Development Tasks Priority
1. **Week 1-2**: Security fixes and local setup (CRITICAL)
2. **Week 3-4**: Documentation and bug fixes
3. **Week 5-6**: Testing infrastructure
4. **Week 7-8**: API completion
5. **Week 9-10**: DevOps and deployment
6. **Week 11-12**: Framework upgrade to .NET 6/8

## Key Features

### Sustainability Assessment
- 16 sustainability qualifiers system
- Multiple questionnaire types for different user segments
- Performance analytics with IPA (Importance-Performance Analysis) matrix
- Pre/post trip sustainability surveys

### User Management
- Role-based access (Admin, Tour Operator, Traveler)
- Company registration and approval workflow
- GDPR compliance features

### Content Management
- Multi-language support
- News/blog system
- Product/tour management
- Media handling with Azure Blob Storage support

## Documentation References

- **Main Action Plan**: [ACTION_PLAN.md](ACTION_PLAN.md) - Your comprehensive 90-day roadmap
- **Technical Analysis**: [doc/CODEBASE_ASSESSMENT.md](doc/CODEBASE_ASSESSMENT.md)
- **Security Checklist**: [doc/SECURITY_CHECKLIST.md](doc/SECURITY_CHECKLIST.md)
- **Quick Setup**: [doc/QUICK_START_CHECKLIST.md](doc/QUICK_START_CHECKLIST.md)
- **Bug Reports**: [doc/CALCULATION_BUGS_FOUND.md](doc/CALCULATION_BUGS_FOUND.md)

## Important Notes

1. **No Source Code**: This repository contains only documentation. Actual development work requires the source code from `SustainableTravel-master`.

2. **Security First**: The platform has exposed credentials and must not be deployed without addressing security issues documented in SECURITY_CHECKLIST.md.

3. **Framework Upgrade Needed**: .NET Core 2.2 is End-of-Life. Plan upgrade to .NET 6 or .NET 8 for continued security support.

4. **Follow ACTION_PLAN.md**: Use this as the single source of truth for all development activities. Work through it step-by-step, checking off completed items.

## Getting Help

When working on this project:
1. Always check ACTION_PLAN.md first for the current task and context
2. Reference the appropriate documentation in the `/doc` folder
3. Follow the 90-day plan structure for systematic progress
4. Prioritize security fixes before any feature development