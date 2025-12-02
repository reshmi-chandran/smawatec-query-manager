# SMAWA API Integration Notes

Last updated: 2025-12-02

## 1. Current Frontend State

- All dashboards still pull data from `MockDataService` (see `src/app/services/mock-data.service.ts`), so there is **no real HTTP layer yet**.
- Components that depend on mock data:
  - `HomeComponent`, `AlarmsComponent`
  - `BuildingComponent`, `DeviceDetailsComponent`
  - `StatisticsComponent`
  - Supporting services (PDF/Excel export, translation, etc.) only wrap the mock data.
- Authentication, user profile, tenant management, and device CRUD flows do **not** exist yet beyond mock UI state.

## 2. Swagger Endpoint Mapping

| App Area | Needed Capability | Swagger Endpoints to Use | Open Questions |
| --- | --- | --- | --- |
| **Authentication & User lifecycle** | Login, registration, profile update, password reset, MFA verification. | `POST /mobile/login`, `POST /mobile/registration/create-user`, `POST /mobile/update-user`, `POST /mobile/login/forgot-password`, `PATCH /mobile/user/password-update`, `POST /mobile/mfa/verify-email-code` | Need exact flow for exchanging the “initial mobile token” for the long-lived token (`mobile_token` header). |
| **Home & Alarms dashboards** | List alarms/events, resolve them, control valves, show per-location device data. | `GET /mobile/user/events`, `PATCH /mobile/user/resolve-event`, `PATCH /mobile/user/change-valve-position`, `GET /mobile/user/location-details` | Need confirmation that `LocationDetailsResponse` includes enough metadata (tenant/building names, addresses, etc.) to render current UI. |
| **Building management & Device organizer** | Show building groups, buildings, apartments, tenants; CRUD for devices; manage “unassigned” devices. | `GET /mobile/user/locations`, `GET /mobile/user/locations-overview`, `GET /mobile/user/location-details`, `POST /mobile/user/create-device`, `PATCH /mobile/user/update-device`, `DELETE /mobile/user/delete-device/{device_id}` | Swagger only exposes “locations” and “devices”; there are **no endpoints for building groups, apartments, tenants, or “unassigned devices.”** Need to know how to map/migrate these concepts. |
| **Statistics / Consumption analytics** | Time-series consumption for apartment/building/group, export data. | **Missing in Swagger**. The only telemetry endpoint is `POST /device/add-flow-data` (device upload). | Need read endpoints that return aggregated consumption; otherwise statistics/export page cannot be wired up. |
| **Notifications & telemetry** | Store FCM token; device gateway uploads. | `POST /mobile/user/firebase-token`, `POST /device/add-flow-data` | Frontend only needs Firebase token call; telemetry upload remains a backend responsibility. |

## 3. Gaps To Confirm With Backend

1. **Data model coverage**
   - Are “building groups”, “apartments”, “tenants”, and “unassigned devices” going to be exposed via future endpoints, or should the frontend collapse everything into the current `Location` model?
2. **Consumption analytics**
   - Is there an upcoming `/mobile/user/statistics` (or similar) endpoint? Without it, we cannot replace `getConsumptionByApartment/Building/Group`.
3. **Token flow**
   - How long does the “initial mobile token” stay valid?
   - Do we refresh the follow-up token via a dedicated endpoint or repeated logins?
4. **Schema/change notifications**
   - Since endpoints are still moving, what’s the process (Slack/email/changelog) for alerting us to breaking changes?
5. **Environment & credentials**
   - Which environment should the frontend target for now (staging vs prod), and are there sandbox accounts we can safely use?

## 4. Suggested Next Steps

1. Backend to clarify/provide the missing endpoints/data listed above.
2. Frontend to replace `MockDataService` calls feature-by-feature once the required API contracts are finalized.
3. Establish a shared Postman collection (or similar) with concrete request/response samples so both teams keep schemas aligned.
4. Define error-handling expectations (status codes, validation errors, temporary outages) so the UI can surface meaningful states.

Once we get confirmation on the open questions, we can prioritize which modules to hook up first (likely auth + alarms, then device management, finally analytics once endpoints exist).

