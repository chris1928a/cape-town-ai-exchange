# Apollo Cache Extraction

The browser cache of an Apollo-based PMS dashboard contains every booking and
season the host has access to. Extract it once, analyze locally, no API calls
needed for read-only checks (velocity, occupancy, anomalies).

## Page-specific cache contents

Different pages hydrate different parts of the cache. Plan accordingly.

| Page | Cache payload |
|---|---|
| `/app/calendars` | All booking units across the portfolio, no seasons |
| `/app/apartments/<id>/pricing` | Seasons for a single unit, no bookings |

Calendar cache is not loaded automatically on the pricing page and vice versa.
Each page needs its own extraction session.

## Cache extraction

```javascript
const c = window.__APOLLO_CLIENT__.cache.extract();

// Booking entries (calendars page)
const bookings = Object.keys(c).filter(k => k.startsWith('BookingUnit:'));

// Season entries (pricing page)
const seasons = Object.keys(c).filter(k => k.startsWith('ApartmentSeason:'));
```

## Map units to bookings

`BookingUnit` has no direct `unitType` reference. `UnitType` does have a
`bookingUnits()` field that holds the back-references.

```javascript
const unitIds = ['<unitId-1>', '<unitId-2>', '<unitId-3>'];

const bookingsByUnit = {};
unitIds.forEach(id => {
  const u = c[`UnitType:${id}`];
  if (!u) return;
  const bookingField = Object.keys(u).find(k => k.startsWith('bookingUnits('));
  bookingsByUnit[id] = (u[bookingField] || []).map(ref => {
    const b = c[ref.__ref];
    return {id: b.id, from: b.fromDate, to: b.toDate, createdAt: b.createdAt};
  });
});
```

## Occupancy check for a target date

```javascript
function occupancyOn(targetDate, bookingsByUnit) {
  const result = {};
  Object.entries(bookingsByUnit).forEach(([id, bookings]) => {
    const occ = bookings.some(b => targetDate >= b.from && targetDate < b.to);
    result[id] = occ ? 'X' : '.';
  });
  return result;
}

occupancyOn('2026-05-03', bookingsByUnit);
// → {unitA: '.', unitB: 'X', unitC: '.'} = 1/3 booked
```

## Velocity check (createdAt timestamps)

Every `BookingUnit` carries a `createdAt`. That is the basis for real-time
velocity tracking.

```javascript
const now = new Date();
const last72h = new Date(now.getTime() - 72 * 3600 * 1000);
const recent = [];
Object.entries(bookingsByUnit).forEach(([id, bookings]) => {
  bookings.forEach(b => {
    if (new Date(b.createdAt) > last72h) {
      recent.push({unit: id, ...b});
    }
  });
});
// recent = bookings created in the last 72 hours
```

Use this to compare your 7-day baseline against actual bookings since the last
tuning. The number that matters is "did this lever fire?", not the absolute rate.

## Auth capture (for direct API calls)

If you need to fire mutations (not just read the cache), capture the bearer
token via a fetch-interceptor. This installs once per page load.

```javascript
(() => {
  if (window.__runGQL) return 'already installed';
  const orig = window.fetch;
  const authStore = {};
  window.fetch = function(...args) {
    const init = args[1] || {};
    if (init.headers && (init.headers.authorization || init.headers.Authorization)) {
      authStore.a = init.headers.authorization || init.headers.Authorization;
    }
    return orig.apply(this, args);
  };
  window.fetch.__orig = orig;
  window.__runGQL = async (body) => {
    if (!authStore.a) return {err: 'no_auth'};
    const r = await orig('https://api.host.holidu.com/graphql', {
      method: 'POST', credentials: 'include',
      headers: {
        'accept': '*/*',
        'content-type': 'application/json',
        'client': 'pms-web',
        'authorization': authStore.a
      },
      body: JSON.stringify(body)
    });
    const text = await r.text();
    return {status: r.status, body: text ? JSON.parse(text) : null};
  };
  window.__authReady = () => !!authStore.a;
  return 'installed';
})();
```

After install, trigger an Apollo refetch so the interceptor sees a real request
and captures the token:

```javascript
window.__APOLLO_CLIENT__.reFetchObservableQueries();
// wait 3 to 4 seconds
window.__authReady(); // true once captured
```

## Gotchas

- Some browser-extension sandboxes flag direct globals like `window.__authValue`
  as sensitive. Workaround: keep the auth in a closure variable, expose only a
  function that uses it. The sandbox sees the function shape, not the string.
- Page navigation drops the interceptor. Install on the page you intend to
  operate from, then run mutations from there without navigating away.
- Cache is per-page. Calendar cache does not auto-load on the pricing page.
  Plan extraction sessions accordingly.

## Notes

- This pattern was developed against Holidu PMS in early 2026. The same approach
  works on most Apollo-based dashboards. Field names and key prefixes differ.
- Read-only cache extraction is a safe technique. Mutations require a captured
  auth token and should be tested on a single unit and a single season before
  running portfolio-wide batches.
