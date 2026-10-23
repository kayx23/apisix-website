# Admin API production checks

This suite needs the assembled production site: current pages are Astro,
while versioned and Next pages are Docusaurus. Serve that output, then run:

```sh
cd next
ADMIN_API_BASE_URL=http://127.0.0.1:4322 npm run test:admin-api
```

The existing Playwright configuration runs desktop and mobile Chromium. The
suite checks both locales for current, Next, 3.18, 3.17 and 3.16. Success tests
require the actual synced canonical English specifications; they must not be
reported as passed by substituting another release's JSON. Failure tests
intercept requests to exercise missing, malformed, failed and timed-out JSON,
and a failed Scalar script. A separate context checks disabled JavaScript.

Before calling a PR ready, run `node next/scripts/check-admin-api-sources.mjs`
from the repository root. This readiness gate must pass against real synced
sources; it is separate from production builds, which retain Markdown when a
source is absent or invalid. Sync and build from these apache/apisix refs:

- current: `release/3.19`
- Next: `master`
- archives: `release/3.18`, `release/3.17`, `release/3.16`

The archive sync copies `docs/en/latest/admin-api.json` byte-for-byte to
`doc/static/assets/openapi/apisix/<version>/admin-api.json`. Both locales use
that same asset. Missing sources remove stale assets and retain Markdown.
The assembled deployment already ships `doc/build/assets`, so no external
specification request or additional deployment copy step is needed.

Run the small source/validation checks with Node 22.18+:

```sh
node --test shared/admin-api-spec.test.mjs scripts/lib/admin-api.test.js
```

## Immutable validation fixtures while Apache PRs are pending

`source-fixtures.json` pins the four prepared specifications in `kayx23/apisix`.
These are test inputs only: production synchronization still uses the matching
`apache/apisix` refs. Do not change deployment or synchronization URLs to the
fork. A passing fixture check does not establish upstream availability.

After preparing generated static assets from those exact commits, check them
without fetching or changing any content:

```sh
node next/scripts/check-admin-api-fixtures.mjs /path/to/apisix-fixture-checkout doc/static/assets/openapi/apisix
```

| Intended Apache ref | Prepared source comparison |
| --- | --- |
| `master` | https://github.com/apache/apisix/compare/master...kayx23:apisix:docs/admin-api-master-contract |
| `release/3.18` | https://github.com/apache/apisix/compare/release/3.18...kayx23:apisix:docs/admin-api-openapi-3.18 |
| `release/3.17` | https://github.com/apache/apisix/compare/release/3.17...kayx23:apisix:docs/admin-api-openapi-3.17 |
| `release/3.16` | https://github.com/apache/apisix/compare/release/3.16...kayx23:apisix:docs/admin-api-openapi-3.16 |

These upstream changes must be submitted and merged before canonical-source
end-to-end verification and deployment can be completed. Report fixture runs,
canonical runs, and unavailable browser/build checks separately.
