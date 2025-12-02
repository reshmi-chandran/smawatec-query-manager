# SMAWA API Integration Notes

Last updated: 2025-12-02

## 1. Current Frontend State

- Authentication, user profile, tenant management, and device CRUD flows do **not** exist yet beyond mock UI state.

## 2. Swagger Endpoint Mapping

| App Area | Needed Capability | Swagger Endpoints to Use | Open Questions |
| --- | --- | --- | --- |
| **Authentication & User lifecycle** | Login, registration, profile update, password reset, MFA verification. | `POST /mobile/login`, `POST /mobile/registration/create-user`, `POST /mobile/update-user`, `POST /mobile/login/forgot-password`, `PATCH /mobile/user/password-update`, `POST /mobile/mfa/verify-email-code` | Could you share the full token flow: which header carries the initial mobile token when calling `/mobile/login`, whether the follow-up token comes from `user_mobile_api_token`, when we should start using it as the `mobile_token` header for `/mobile/user/...`, and if/when it expires or refreshes? |
| **Home & Alarms dashboards** | List alarms/events, resolve them, control valves, show per-location device data. | `GET /mobile/user/events`, `PATCH /mobile/user/resolve-event`, `PATCH /mobile/user/change-valve-position`, `GET /mobile/user/location-details` | From the Swagger response schema it isn’t clear if `LocationDetailsResponse` contains tenant/building metadata. Could you confirm whether it does, or if additional fields/endpoints are planned? |
| **Building management & Device organizer** | Show building groups, buildings, apartments, tenants; CRUD for devices; manage “unassigned” devices. | `GET /mobile/user/locations`, `GET /mobile/user/locations-overview`, `GET /mobile/user/location-details`, `POST /mobile/user/create-device`, `PATCH /mobile/user/update-device`, `DELETE /mobile/user/delete-device/{device_id}` | We only spotted “location” and “device” endpoints in Swagger. If there are other APIs for building groups, apartments, tenants, or “unassigned” devices, please point us to them; otherwise we’d appreciate guidance on how you’d like the current concepts mapped. |
| **Statistics / Consumption analytics** | Time-series consumption for apartment/building/group, export data. | Not listed in Swagger (the only telemetry endpoint shown is `POST /device/add-flow-data`, which appears to be device-facing). | Are read endpoints for aggregated consumption on the roadmap? Without them, the statistics/export page would need to remain on mock data. |
| **Notifications & telemetry** | Store FCM token; device gateway uploads. | `POST /mobile/user/firebase-token`, `POST /device/add-flow-data` | We’re assuming the frontend will call the Firebase token endpoint, while `POST /device/add-flow-data` is meant for device firmware/backends. Please let us know if you expect the web client to interact with the telemetry endpoint as well. |

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

