# User Auth Identity Portal

> A security-conscious, full-stack identity platform featuring JWT authentication, email-verified registration, and a secure password reset flow.

---

## Overview

Modern applications demand more than a basic login form. **User Auth Identity Portal** is a production-minded authentication system built to demonstrate the kind of identity infrastructure that real-world, security-sensitive applications require.

The platform enforces a complete user lifecycle: registration with email verification, credential-protected login, JWT-based session management, self-service profile editing, and a cryptographically secure password reset flow — all backed by a cleanly layered ASP.NET Core API and a reactive React frontend.

**Who it's for:** Developers, hiring teams, and security reviewers looking for a reference implementation of identity best practices in a .NET + React stack.

**What makes it notable:** Authentication logic is isolated into a dedicated service layer (`AuthService` / `IAuthService`), email tokens carry a server-enforced 24-hour expiry, JWT claims are validated on every protected endpoint, and a global exception middleware prevents internal error details from leaking to clients.

---

## Live Demo

🚀 Demo coming soon

---

## Tech Stack

| Category | Technology |
|---|---|
| Frontend Framework | React 18 + TypeScript |
| Build Tool | Vite |
| UI Component Library | Material UI (MUI v5) |
| State Management | Redux Toolkit |
| API Layer | RTK Query |
| Form Handling | React Hook Form |
| Client Routing | React Router v6 |
| Token Parsing | jwt-decode |
| Backend Framework | ASP.NET Core Web API (.NET) |
| Authentication | ASP.NET Core Identity + JWT Bearer (HMAC-SHA256) |
| ORM | Entity Framework Core |
| Database | Microsoft SQL Server |
| Email Delivery | MailKit / MimeKit (SMTP) |
| API Documentation | Swagger / OpenAPI |

---

## Key Features

- 🔐 **JWT Authentication** — Stateless Bearer tokens signed with HMAC-SHA256, with configurable expiry and full issuer/audience validation on every protected route.
- ✉️ **Email Confirmation Tokens** — New accounts are locked until the user clicks a confirmation link; tokens are issued by ASP.NET Core Identity and expire after 24 hours.
- 🔑 **Secure Password Reset Flow** — Time-limited reset tokens are generated server-side and delivered by email; the token is consumed once and cannot be reused.
- 🛡️ **Server-Enforced Password Policy** — Passwords must include uppercase, lowercase, digit, and special character with a minimum length of 8, enforced by ASP.NET Core Identity and mirrored in client-side validation.
- 🏗️ **Dedicated Authentication Service Layer** — All identity logic lives in `AuthService` behind the `IAuthService` interface, keeping controllers thin and business logic independently testable.
- 👤 **Protected Profile Management** — Users can view and edit their name and change their password; every profile endpoint requires a valid JWT.
- 🚧 **Global Exception Middleware** — A custom `ExceptionMiddleware` in `LS.Core` intercepts unhandled exceptions and returns a structured error response, preventing stack-trace leakage.
- 📋 **Swagger UI with Bearer Auth** — The API is fully documented and testable via Swagger, with JWT Bearer authentication pre-configured in the UI.

---

## Architecture Overview

```
┌─────────────────────────────────────┐
│         React SPA (port 5173)       │
│  Redux Toolkit / RTK Query          │
│  React Router  │  React Hook Form   │
└──────────────┬──────────────────────┘
               │  HTTP / JSON (Bearer JWT)
               ▼
┌─────────────────────────────────────┐
│  ASP.NET Core Web API (port 5041)   │
│  AuthController                     │
│  ┌──────────────────────────────┐   │
│  │  LS.BLL  (Business Logic)    │   │
│  │  AuthService  EmailService   │   │
│  └──────────┬───────────────────┘   │
│  ┌──────────┴───────────────────┐   │
│  │  LS.DAL  (Data Access)       │   │
│  │  EF Core  LoginDbContext     │   │
│  │  ApplicationUser  Migrations │   │
│  └──────────┬───────────────────┘   │
│  ┌──────────┴───────────────────┐   │
│  │  LS.Core (Cross-cutting)     │   │
│  │  ExceptionMiddleware         │   │
│  └──────────────────────────────┘   │
└──────────────┬──────────────────────┘
               │
               ▼
       SQL Server Database
```

**Authentication flow:**
1. User registers → Identity creates account (`EmailConfirmed = false`) → confirmation email sent.
2. User clicks email link → `ConfirmEmail` endpoint validates token → account activated.
3. User logs in → password checked, `EmailConfirmed` verified → JWT issued.
4. All subsequent API calls carry the JWT in the `Authorization: Bearer` header.

---

## Getting Started

### Prerequisites

- [.NET SDK 8+](https://dotnet.microsoft.com/download)
- [SQL Server](https://www.microsoft.com/en-us/sql-server) (local or remote)
- [Node.js 18+](https://nodejs.org/) and [pnpm](https://pnpm.io/)
- A Gmail account with an [App Password](https://support.google.com/accounts/answer/185833) enabled for SMTP

### Installation

```bash
# Clone the repository
git clone https://github.com/yashvi-khunt/user-auth-identity-portal.git
cd user-auth-identity-portal
```

### Environment Variables

**Backend** — edit `Backend/LoginSystem/LoginSystem/appsettings.json`:

```json
{
  "ConnectionStrings": {
    "Default": "Data Source=<server>; Database=login-system; TrustServerCertificate=True; Trusted_Connection=True;"
  },
  "JWT": {
    "ValidAudience": "http://localhost:5041",
    "ValidIssuer": "http://localhost:61925",
    "Secret": "<your-32-char-secret>",
    "TokenValidityInMinutes": 1440
  },
  "EmailSettings": {
    "Host": "smtp.gmail.com",
    "Port": 587,
    "DisplayName": "<Your Name>",
    "SenderEmail": "<your-gmail>@gmail.com",
    "Password": "<gmail-app-password>"
  }
}
```

> ⚠️ Never commit real secrets to source control. Use [.NET user-secrets](https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets) or environment variables in production.

### Run Locally

**Backend:**

```bash
cd Backend/LoginSystem

# Apply database migrations
dotnet ef database update --project LS.DAL --startup-project LoginSystem

# Start the API
dotnet run --project LoginSystem
# API available at http://localhost:5041
# Swagger UI at http://localhost:5041/swagger
```

**Frontend:**

```bash
cd Frontend/LoginSystem

pnpm install
pnpm dev
# App available at http://localhost:5173
```

---

## Project Structure

```
user-auth-identity-portal/
├── Backend/
│   └── LoginSystem/
│       ├── LoginSystem/                  # ASP.NET Core entry point
│       │   ├── Controllers/
│       │   │   └── AuthController.cs     # API endpoints (login, register, confirm, reset, profile)
│       │   ├── Program.cs                # DI configuration, JWT & Identity setup, CORS
│       │   └── appsettings.json          # App configuration (JWT, DB, Email)
│       ├── LS.BLL/                       # Business Logic Layer
│       │   ├── Repositories/             # Service interfaces (IAuthService, IEmailService)
│       │   └── Services/
│       │       ├── AuthService.cs        # Core identity logic (register, login, confirm email)
│       │       └── EmailService.cs       # SMTP email delivery via MailKit
│       ├── LS.DAL/                       # Data Access Layer
│       │   ├── Helper/
│       │   │   ├── ApplicationUser.cs    # Extended IdentityUser (FirstName, LastName)
│       │   │   ├── LoginDbContext.cs     # EF Core DbContext
│       │   │   ├── EmailSettings.cs      # SMTP configuration model
│       │   │   └── MailRequest.cs        # Email DTO
│       │   ├── Migrations/               # EF Core database migrations
│       │   └── ViewModels/               # Request/response DTOs (VMLogin, VMRegister, etc.)
│       └── LS.Core/                      # Cross-cutting concerns
│           └── Middlewares/
│               └── ExceptionMiddleware.cs # Global error handling middleware
└── Frontend/
    └── LoginSystem/
        ├── src/
        │   ├── components/               # Feature components
        │   │   ├── Login.tsx
        │   │   ├── Register.tsx
        │   │   ├── ForgotPassword.tsx
        │   │   ├── ResetPassword.tsx
        │   │   ├── ChangePassword.tsx
        │   │   ├── ProfilePage.tsx
        │   │   ├── ProfileEdit.tsx
        │   │   ├── EmailConfirmSuccess.tsx
        │   │   └── form/                 # Reusable form field components
        │   ├── pages/
        │   │   ├── Profile.tsx           # Profile layout page
        │   │   └── Header.tsx            # App header
        │   ├── redux/
        │   │   ├── authApi.ts            # RTK Query API slice (all auth endpoints)
        │   │   ├── authSlice.ts          # Auth state (JWT storage, decode, logout)
        │   │   ├── snackbarSlice.ts      # Global notification state
        │   │   ├── store.ts              # Redux store configuration
        │   │   └── hooks.ts              # Typed Redux hooks
        │   ├── routes/
        │   │   └── router.tsx            # React Router config with protected routes
        │   ├── types/                    # TypeScript type declarations
        │   └── App.tsx
        ├── index.html
        ├── vite.config.ts
        └── package.json
```

---

## Screenshots

📸 Screenshots coming soon

---

## What I Learned

- **Layering security at every tier** — Implementing the same password-strength rules on both the client (React Hook Form regex) and the server (ASP.NET Core Identity options) taught me that client-side validation is a UX convenience, but the real enforcement always has to live on the backend.
- **Token lifecycle management** — Generating, delivering, and consuming time-limited tokens for both email confirmation and password reset gave me hands-on experience with how ASP.NET Core Identity's `DataProtectionTokenProvider` works under the hood, including configuring the 24-hour expiry window.
- **Keeping controllers thin** — Extracting all identity logic into `AuthService` behind `IAuthService` made the controller easier to read and would make it straightforward to unit-test the business rules in isolation — a pattern I'll carry into every future API project.
- **State management for auth in React** — Wiring JWT decoding, localStorage persistence, and protected routes together through Redux Toolkit and RTK Query showed me how to build a cohesive, type-safe auth flow in a React SPA without relying on a third-party auth library.

---

## Author

**Yashvi Khunt**  
MS Computer Science (Cybersecurity) — Stevens Institute of Technology

- 🐙 GitHub: [github.com/yashvi-khunt](https://github.com/yashvi-khunt)
- 💼 LinkedIn: [linkedin.com/in/yashvi-khunt](https://linkedin.com/in/yashvi-khunt)
