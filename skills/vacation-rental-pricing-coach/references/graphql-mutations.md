# GraphQL Mutations Reference (Holidu PMS)

Verified shapes for the three core season mutations on the Holidu host API. The
patterns generalize, the field names are Holidu-specific.

**Endpoint:** `https://api.host.holidu.com/graphql`
**Auth:** Bearer token captured via fetch-interceptor (see `cache-extraction.md`)
**Client header:** `client: pms-web`
**Introspection:** disabled, schema discovered via error-pattern probing

## 1. updateUnitTypeSeason

Mutate an existing season (rate, MinStay, end date, weekend price, name).

```graphql
mutation UpsertUnitTypeSeason($input: UpsertUnitTypeSeasonInput!) {
  updateUnitTypeSeason(input: $input) {
    id fromDate toDate nightlyRate weekendPrice minStay
  }
}
```

**Gotcha:** the mutation is named `updateUnitTypeSeason` but the input type is
`UpsertUnitTypeSeasonInput!`. The `seasonId` lives inside the input object, not
as a separate argument.

## 2. createUnitTypeSeason

Create a new season window. Returns the new ID.

```graphql
mutation CreateUnitTypeSeason($input: UpsertUnitTypeSeasonInput!) {
  createUnitTypeSeason(input: $input) {
    id fromDate toDate nightlyRate
  }
}
```

## 3. deleteUnitTypeSeason

Delete a season. Note the different input type.

```graphql
mutation DeleteUnitTypeSeason($deleteUnitTypeSeasonInput: DeleteUnitTypeSeasonInput!) {
  deleteUnitTypeSeason(input: $deleteUnitTypeSeasonInput) {
    seasonId
  }
}
```

Variables shape:

```json
{
  "deleteUnitTypeSeasonInput": {
    "unitTypeId": "<unitId>",
    "seasonId": "<seasonId>"
  }
}
```

## Query: list seasons in a date range

```graphql
query GetSeasons($unitTypeId: ID!, $fromDate: LocalDate, $toDate: LocalDate) {
  unitType(unitTypeId: $unitTypeId) {
    id
    seasons(filter: {from: $fromDate, to: $toDate, excludeFlexibleSeasons: true}) {
      id fromDate toDate nightlyRate weekendPrice minStay name
    }
  }
}
```

## UpsertUnitTypeSeasonInput - required fields

| Field | Type | Notes |
|---|---|---|
| `unitTypeId` | Int | Apartment / unit ID from the host portfolio |
| `fromDate` | LocalDate | "YYYY-MM-DD", inclusive |
| `toDate` | LocalDate | "YYYY-MM-DD", inclusive |
| `nightlyRate` | Int | Host list price |
| `minStay` | Int | Minimum nights |
| `currency` | String | Currency code |
| `minEarningEnabled` | Boolean | Set false unless you understand the dynamic-pricing engine |
| `name` | String | Free text label, e.g. "Spring Bridge" |
| `checkInMonday` ... `checkInSunday` | Boolean | All seven days, all true unless you have a reason |
| `checkOutMonday` ... `checkOutSunday` | Boolean | All seven days, all true unless you have a reason |

**On update only:** add `seasonId` (Int).

## Optional fields

- `weekendPrice` (Int) - used as the Friday and Saturday nightly rate. Sunday
  through Thursday fall back to `nightlyRate`. Omit if there is no weekend uplift.

## Fetch helper

```javascript
async function runGQL(body) {
  const r = await window.fetch.__orig('https://api.host.holidu.com/graphql', {
    method: 'POST',
    credentials: 'include',
    headers: {
      'accept': '*/*',
      'content-type': 'application/json',
      'client': 'pms-web',
      'authorization': window.__capturedAuth
    },
    body: JSON.stringify(body)
  });
  const text = await r.text();
  return {status: r.status, body: text ? JSON.parse(text) : null};
}
```

## Error patterns (schema discovery)

| Error fragment | Meaning |
|---|---|
| `FieldUndefined` | Mutation or field name wrong, check casing |
| `UnknownType` | Input type name wrong |
| `has coerced Null value for NonNull type` | Required field missing, message names which |
| HTTP 401 | Auth token missing or expired, recapture |

## Performance budget

- Single mutation: 300 to 500 ms
- Batch 30 deletes plus 12 creates: 15 to 20 seconds
- Full portfolio rebuild (six units, ~56 seasons each, ~720 calls): 2 to 3 minutes
- Above 45 seconds, chunk the script per unit to avoid browser-side script timeouts

## Discount mutations

Holidu exposes `UpsertUnitTypeDiscount` for the discount stack. Discount types
include `LOS_DISCOUNT` (length-of-stay / weekly), `EARLY_BIRD`, `MOBILE_DISCOUNT`,
`LAST_MINUTE`, `LOYALTY`, `HOST_WEBSITE`, `NON_REFUNDABLE`.

The API only accepts preset blackout types (`PEAK` or `PEAK_AND_HIGH`), not
custom date ranges. To exclude a specific window from a discount, you must
toggle the preset type or temporarily deactivate the discount.

```graphql
mutation UpsertUnitTypeDiscount($input: UpsertUnitTypeDiscountInput!) {
  upsertUnitTypeDiscount(input: $input) {
    discountId discountType status
  }
}
```

Required input fields: `discountId` (existing) or omit (new), `discountType`,
`unitTypeId`, `discountPercent`, `status` (`ACTIVE` or `INACTIVE`),
`excludedSeasonalityEnabled`, `selectedSeasonalityConfigs`.

`EARLY_BIRD` additionally requires `daysBeforeCheckIn` (typical: 30).

## Notes

- These shapes were verified against the Holidu PMS API in early 2026. Schema
  changes are possible without notice, since the API is undocumented.
- Real unit IDs and season IDs are intentionally not in this reference. Pull
  yours from the Apollo cache (see `cache-extraction.md`).
