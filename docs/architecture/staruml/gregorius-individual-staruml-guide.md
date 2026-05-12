# StarUML Reconstruction Guide - Gregorius Individual Architecture

StarUML was not available in this environment, so
`bidmart-individual-architecture.mdj` was not generated. Do not claim the PNGs
in this folder are StarUML exports. They are PlantUML preview images generated
from the same architecture content.

Before final submission, manually recreate these diagrams in StarUML and export
them to the listed PNG paths.

## Boundary Rules

- This individual work expands only `auth-fe -> auth-be -> auth-db`.
- Do not show Core DB.
- Do not show Admin DB.
- Do not show Admin Backend direct database access.
- Auth Backend repositories connect only to Auth PostgreSQL.
- Resend Email Provider is used only for email-related flows, especially MFA
  email.

## Required StarUML Project

Create this file manually in StarUML:

`docs/architecture/staruml/bidmart-individual-architecture.mdj`

It should contain these diagrams:

1. `Individual Component Diagram - Auth Settings Security`
2. `Code Diagram - Auth Frontend Settings`
3. `Code Diagram - Auth Backend Settings`
4. `Bonus Component Diagram - MFA Settings`
5. `Bonus Code Diagram - MFA Settings`

## Diagram 1: Individual Component Diagram - Auth Settings Security

Export to:

`docs/architecture/individual-component-auth-settings-security.png`

Diagram type: UML Component Diagram.

Containers:

- `Auth Frontend Container` `<<Frontend>>`
- `Auth Backend Container` `<<Backend>>`

Auth Frontend components:

- `Settings Pages`: `profile-page`, `security-page`, `sessions-page`,
  `notifications-page`, `mfa-page`
- `Settings Hooks`: `useGetMfaStatusQuery`, `useChangePasswordMutation`,
  `useGetSessionsQuery`, `useUpdateNotificationPreferencesMutation`
- `Settings Use Cases`: `GetMfaStatusUseCase`, `ChangePasswordUseCase`,
  `GetSessionsUseCase`, `UpdateNotificationPreferencesUseCase`
- `SettingsApiRepository`
- `apiClient`
- `Zod settings schemas`

Auth Backend components:

- `Settings/Auth Controllers`: `get_profile`, `update_profile`,
  `change_password`, `get_sessions`, `revoke_session`,
  `revoke_all_sessions`, `get_notification_preferences`,
  `update_notification_preferences`, `get_mfa_settings`
- `AuthenticatedUser / JWT extractor`
- `AuthMfaUseCase`
- `PostgresUserRepository`
- `PostgresSessionRepository`
- `PostgresNotificationPreferencesRepository`
- `PostgresEmailMfaCodeRepository`
- `PostgresTotpSetupRepository`
- `ScryptPasswordHasher`
- `TotpRsService`
- `ResendVerificationEmailSender`

External components:

- `Auth PostgreSQL` `<<Database>>`
- `Resend Email Provider` `<<External System>>`

Relationships:

- Settings pages call settings hooks.
- Settings hooks execute settings use cases.
- Settings use cases use `SettingsApiRepository`.
- `SettingsApiRepository` validates with Zod schemas and calls `apiClient`.
- `apiClient` calls settings/auth controllers with a bearer token.
- Controllers require `AuthenticatedUser` and call `AuthMfaUseCase`.
- `AuthMfaUseCase` calls repositories and services.
- PostgreSQL repositories connect only to `Auth PostgreSQL`.
- `ResendVerificationEmailSender` connects only to `Resend Email Provider`.

## Diagram 2: Code Diagram - Auth Frontend Settings

Export to:

`docs/architecture/individual-code-frontend-settings.png`

Diagram type: UML Class Diagram.

Include only frontend settings code:

- Pages: `ProfilePage`, `SecurityPage`, `SessionsPage`,
  `NotificationsPage`, `MfaPage`
- Hooks: `useGetProfileQuery`, `useUpdateProfileMutation`,
  `useChangePasswordMutation`, `useGetSessionsQuery`,
  `useRevokeSessionMutation`, `useGetNotificationPreferencesQuery`,
  `useUpdateNotificationPreferencesMutation`, `useGetMfaStatusQuery`
- Use cases: `GetProfileUseCase`, `UpdateProfileUseCase`,
  `ChangePasswordUseCase`, `GetSessionsUseCase`, `RevokeSessionUseCase`,
  `GetNotificationPreferencesUseCase`, `UpdateNotificationPreferencesUseCase`,
  `GetMfaStatusUseCase`
- Domain: `ISettingsRepository`, `UserProfile`, `Session`,
  `NotificationPreferences`, `MfaStatus`
- Infrastructure: `SettingsApiRepository`, `SettingsApiMapper`,
  `settings-api/schemas.ts`, `apiClient`, `accessTokenStore`

Do not include backend classes in this diagram.

## Diagram 3: Code Diagram - Auth Backend Settings

Export to:

`docs/architecture/individual-code-backend-settings.png`

Diagram type: UML Class Diagram.

Include only backend settings/security code:

- `controllers`
- `AuthenticatedUser`
- `AuthMfaUseCase`
- Command DTOs: `UpdateProfileCommand`, `ChangePasswordCommand`,
  `UpdateNotificationPreferencesCommand`
- Response DTOs: `MfaSettingsDto`, `SessionDto`, `NotificationPreferencesDto`
- Context DTO: `AuthenticatedUserContext`
- Entities: `User`, `AuthSession`, `NotificationPreferences`,
  `EmailMfaCode`, `TotpSetup`
- Repository traits: `UserRepository`, `SessionRepository`,
  `NotificationPreferencesRepository`, `EmailMfaCodeRepository`,
  `TotpSetupRepository`
- Infrastructure repositories: `PostgresUserRepository`,
  `PostgresSessionRepository`, `PostgresNotificationPreferencesRepository`,
  `PostgresEmailMfaCodeRepository`, `PostgresTotpSetupRepository`
- Services: `ScryptPasswordHasher`, `TotpRsService`,
  `ResendVerificationEmailSender`, `SystemClock`
- External storage/provider: `Auth PostgreSQL`, `Resend Email Provider`

Do not include frontend classes in this diagram.

## Diagram 4: Bonus Component Diagram - MFA Settings

Export to:

`docs/architecture/bonus-component-mfa-settings.png`

Diagram type: UML Component Diagram.

Show:

- TOTP setup/verify path to `TotpRsService` and
  `PostgresTotpSetupRepository`.
- Email MFA setup/verify path to `ResendVerificationEmailSender` and
  `PostgresEmailMfaCodeRepository`.
- Both paths persist only to `Auth PostgreSQL`.
- No Core DB, Admin DB, or Admin Backend direct database access.

## Diagram 5: Bonus Code Diagram - MFA Settings

Export to:

`docs/architecture/bonus-code-mfa-settings.png`

Diagram type: UML Class Diagram.

Show:

- `AuthMfaUseCase`
- MFA command DTOs
- `MfaSettingsDto`
- `User`, `TotpSetup`, `EmailMfaCode`, `EmailMfaCodePurpose`
- MFA ports: `UserRepository`, `TotpSetupRepository`,
  `EmailMfaCodeRepository`, `TotpService`, `MfaEmailSender`,
  `PasswordVerifier`, `VerificationTokenGenerator`,
  `VerificationTokenHasher`, `Clock`
- MFA infrastructure: `PostgresUserRepository`,
  `PostgresTotpSetupRepository`, `PostgresEmailMfaCodeRepository`,
  `TotpRsService`, `ResendVerificationEmailSender`,
  `ScryptPasswordHasher`, `RandomVerificationTokenGenerator`,
  `Sha256VerificationTokenHasher`, `SystemClock`
- `Auth PostgreSQL`
- `Resend Email Provider`

