# SMAWA API Integration Notes

Last updated: 2025-12-02

## 1. Current Frontend State

- Authentication, user profile, tenant management, and device CRUD flows do **not** exist yet beyond mock UI state.

## 2. Swagger Endpoint Mapping

| App Area | Needed Capability | Swagger Endpoints to Use | Open Questions |
| --- | --- | --- | --- |
| **Authentication & User lifecycle** | Login, registration, profile update, password reset, MFA verification. | `POST /mobile/login`, `POST /mobile/registration/create-user`, `POST /mobile/update-user`, `POST /mobile/login/forgot-password`, `PATCH /mobile/user/password-update`, `POST /mobile/mfa/verify-email-code` | Could you share the full token flow: which header carries the initial mobile token when calling `/mobile/login`, whether the follow-up token comes from `user_mobile_api_token`, when we should start using it as the `mobile_token` header for `/mobile/user/...`, and if/when it expires or refreshes? |
| **Home & Alarms dashboards** | List alarms/events, resolve them, control valves, show per-location device data. | `GET /mobile/user/events`, `PATCH /mobile/user/resolve-event`, `PATCH /mobile/user/change-valve-position`, `GET /mobile/user/location-details` | Please confirm whether `LocationDetailsResponse` includes enough metadata (tenant/building names, addresses, etc.) to render the current UI. |
| **Building management & Device organizer** | Show building groups, buildings, apartments, tenants; CRUD for devices; manage “unassigned” devices. | `GET /mobile/user/locations`, `GET /mobile/user/locations-overview`, `GET /mobile/user/location-details`, `POST /mobile/user/create-device`, `PATCH /mobile/user/update-device`, `DELETE /mobile/user/delete-device/{device_id}` | Swagger only exposes “locations” and “devices”; there are **no endpoints for building groups, apartments, tenants, or “unassigned devices.”** Could you advise how you’d like us to map or migrate these concepts? |
| **Statistics / Consumption analytics** | Time-series consumption for apartment/building/group, export data. | **Missing in Swagger**. The only telemetry endpoint is `POST /device/add-flow-data` (device upload). | Are read endpoints for aggregated consumption on the roadmap? Without them, the statistics/export page must stay on mock data. |
| **Notifications & telemetry** | Store FCM token; device gateway uploads. | `POST /mobile/user/firebase-token`, `POST /device/add-flow-data` | Frontend only needs Firebase token call; telemetry upload remains a backend responsibility. |

## 3. Gaps To Confirm With Backend

1. **Data model coverage**
   - Could you let us know whether “building groups”, “apartments”, “tenants”, and “unassigned devices” will have dedicated endpoints, or if you’d prefer the frontend to collapse everything into the current `Location` model?
2. **Consumption analytics**
   - Is a `/mobile/user/statistics` (or similar) endpoint planned? That would let us replace `getConsumptionByApartment/Building/Group`.
3. **Token flow**
   - Which header should carry the initial mobile token when we call auth endpoints?
   - After login, should we take `user_mobile_api_token` (or another field) and start sending it as the `mobile_token` header for all `/mobile/user/...` requests?
   - Does the follow-up token expire, and would you like us to refresh it via another endpoint or by repeating login?
4. **Environment & credentials**
   - Which environment should the frontend target for now (staging vs prod), and are there sandbox accounts we can safely use?

Once we get confirmation on the open questions, we can prioritize which modules to hook up first (likely auth + alarms, then device management, finally analytics once endpoints exist).

