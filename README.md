# Auth Infrastructure

This project focuses on the infrastructure side of authentication, designing a standalone system that preserves availability, traceability, and resource stability during brute force attacks, credential stuffing, and sudden traffic spikes.

Rather than implementing a full identity management platform, the system is intentionally scoped around authentication, session handling, and defensive mechanisms, with architecture decisions driven by failure modes observed in real-world attack scenarios.

## Run App

### Start RabbitMQ (Docker recommended)
> Make sure it's running on port 15672

```bash

docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4-management
```
### Run The API
```bash
# Navigate to the API folder
cd AuthServiceSolution/Api

# Restore dependencies
dotnet restore

# Run the application
dotnet run
```

## Run Tests
```bash
# Navigate to the solution root
cd AuthServiceSolution

# Run all tests
dotnet test
```

## Project Structure & Architecture
```
AuthServiceSolution/
│
├─ Core/
│  ├─ Constants/
│  ├─ Dto/                     
│  ├─ Entities/
│  ├─ Interfaces/
│
├─ Infrastructure/           
│  ├─ Data/
│  ├─ EventBus/ 
│  ├─ Repositories/ 
│
├─ Application/
│  ├─ Helpers/            
│  ├─ Services/
│  └─ UseCases/               
│
└─ Api/                    
│  ├─ Controllers/
│  ├─ Extensions/
│  ├─ Program.cs
│  ├─ appsettingExample.json
│
└─ Tests/    
```

## Security Implementation

- JWT Authentication with refresh tokens
- IP Rate Limiting - Global request throttling to prevent DDoS and abuse
- Password hashing with BCrypt (work factor 12)
- Login attempt logging (successful and failed)
- Temporary account lockout after multiple failed attempts
- Refresh token revocation on logout
- Secure token validation without sensitive information exposure
- Queue-based persistence (RabbitMQ) for failed login attempts to prevent database connection pool saturation during brute force attacks

## Audit & Monitoring

- UserLoginHistory: Successful login tracking for analytics and reporting
- SecurityLoginAttempt: Failed attempt monitoring and block tracking
- EmailAttemptsService: Real-time brute force prevention

## Auth Use Case

- Credential validation with account lockout handling
- Automatic invalidation of previous tokens when generating new ones
- Transactional persistence for data consistency
- IP address and device information capture for audit trails

## Testing Strategy

- Multi-layer test suite (Entities, Services, UseCases)
- Critical component coverage: UserService, RefreshToken, EmailAttempts
- Comprehensive Auth Use Case tests (full login flow)
- Entity validation tests for domain logic

## Data Layer

- Entity Framework Core with configured DbContext
- SQL Server as primary database
- Repository pattern for data access abstraction
- Environment-aware configuration via connection strings

## Future Improvements

- Add proxy configuration middleware to accurately capture real client IP addresses, enhancing security, audit trails, and IP-based blocking in production environments
- Secure password recovery endpoint with time-limited tokens
-  Separation of Responsibilities in AuthUseCase


> In this project, the authentication use case (`AuthUseCase`) is designed with a clear separation of responsibilities in mind. By using coordinators, each aspect of the authentication workflow—such as user validation, token management, and login auditing can be encapsulated separately, making the code more organized, maintainable, and easier to understand. 


| Coordinator       | Responsibility                                      | Internal Services                                       |
|------------------|----------------------------------------------------|--------------------------------------------------------|
| UserCoordinator   | Handle user validation, credentials, and account blocking | `_userServices`, `_emailAttemptsService`             |
| TokenCoordinator  | Generate, revoke, and manage tokens               | `_jwtService`, `_refreshTokenService`                |
| AuditCoordinator  | Record login audits and failed login attempts     | `IEventProducer`, `_logger` |


