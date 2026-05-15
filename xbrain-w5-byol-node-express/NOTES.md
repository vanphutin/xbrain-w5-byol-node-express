# Migration Notes: Express → AWS Lambda

## Strategy Chosen: `serverless-http`

The adapter library [`serverless-http`](https://github.com/dougmoscrop/serverless-http) was selected to wrap the existing Express app for Lambda deployment.

The entire integration is a single 3-line file (`lambda.js`):

```javascript
const serverless = require('serverless-http');
const app = require('./app');
module.exports.handler = serverless(app);
```

`app.js` and `server.js` are **not modified** — the app continues to run locally via `node server.js` unchanged.

---

## Why `serverless-http` Over Alternatives

| Option | Code changes | Dependencies | Notes |
|--------|-------------|--------------|-------|
| **serverless-http** ✅ | +1 file (3 lines) | 1 npm package | Simplest JS-based option, battle-tested |
| `@vendia/serverless-express` | +1 file (3 lines) | 1 npm package | Fork of serverless-http, less actively maintained |
| AWS Lambda Web Adapter | 0 JS lines | Lambda Layer | Requires Layer ARN and additional SAM config |
| Roll your own | ~20–50 lines | 0 | Complex, prone to edge-case bugs |

**Reasons for choosing `serverless-http`:**

- Fewest changes among JS-based options — only one new file
- Small package footprint (~15 KB)
- Native support for HTTP API payload format v2.0 (auto-detected from event structure)
- No complex SAM template changes required (unlike AWS Lambda Web Adapter)
- Actively maintained with broad community adoption

---

## Cold Start Measurement

**Measured value:** 256.54 ms

| Field | Value |
|-------|-------|
| Init Duration | **256.54 ms** |
| Duration (handler) | 26.75 ms |
| Billed Duration | 284 ms |
| Memory Size | 512 MB |
| Max Memory Used | 93 MB |

### Method

Measured from AWS CloudWatch Logs → log group `/aws/lambda/byol-node-express` → first invocation's `REPORT` line:

```
REPORT RequestId: a8866874-840b-49ee-ae87-dcd70a56946f  Duration: 26.75 ms  Billed Duration: 284 ms  Memory Size: 512 MB  Max Memory Used: 93 MB  Init Duration: 256.54 ms
```

`Init Duration` only appears on cold start invocations (when the Lambda container is initialized for the first time). Subsequent warm invocations do not include this field.
