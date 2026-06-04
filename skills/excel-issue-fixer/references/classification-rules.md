# Classification Rules

## backend signals (any match → backend)
- C#, .NET, ASP.NET, controller, endpoint, route, HTTP verb (GET/POST/PUT/DELETE/PATCH)
- service, repository, interface, dependency injection, DI
- DbContext, EF Core, migration, SQL, query, stored procedure, Dapper
- auth, JWT, OAuth, claim, role, permission, policy, middleware
- MediatR, CQRS, command, handler, pipeline behavior, validator
- background service, hosted service, SignalR hub (server-side)
- Azure Function, Service Bus, queue, event, message

## frontend signals (any match → frontend)
- Angular, component, module, template, HTML, CSS, SCSS, style
- routing, route guard, lazy load, resolver
- reactive form, FormGroup, FormControl, ngModel, validator (client-side)
- HttpClient, interceptor, service (Angular service)
- pipe, directive, decorator (@Component, @Injectable, @NgModule)
- Angular Material, CDK, overlay, dialog, snackbar, table, paginator
- UI, UX, layout, responsive, mobile, accessibility, a11y
- state management, NgRx, signal, BehaviorSubject, observable
- i18n, translation, locale

## fullstack signals
- "full stack", "end to end", "from API to UI"
- issue mentions both API response AND display/rendering
- performance: backend slow + frontend shows loading
- auth: JWT generation (backend) + token storage/refresh (frontend)

## priority mapping
| Excel value | Internal |
|---|---|
| Critical / P0 / Urgent | critical |
| High / P1 | high |
| Medium / P2 / Normal | medium |
| Low / P3 / Minor | low |
| (empty) | medium |

## ambiguous → ask
If issue description < 10 words and no clear signal → ask user before classifying.
