# @sekolahcode/devpulse-node

Node.js SDK for DevPulse — server-side error tracking with Express/Connect middleware support.

## Requirements

- Node.js ≥ 18
- A running DevPulse server

## Installation

```bash
npm install @sekolahcode/devpulse-node
```

No external runtime dependencies — uses only Node.js built-in modules.

## Quick Start

```js
const { DevPulse } = require('@sekolahcode/devpulse-node');

DevPulse.init({
  dsn: 'https://your-devpulse-host/api/ingest/YOUR_API_KEY',
  environment: 'production',
  release: '1.0.0',
});
```

After `init()`, uncaught exceptions and unhandled promise rejections are captured automatically.

## Express Integration

Mount `errorHandler()` **after** all other middleware and routes:

```js
const express = require('express');
const { DevPulse } = require('@sekolahcode/devpulse-node');

DevPulse.init({ dsn: 'https://your-devpulse-host/api/ingest/YOUR_API_KEY' });

const app = express();

app.get('/', (req, res) => res.send('Hello'));

// Must be last — catches errors passed via next(err)
app.use(DevPulse.errorHandler());

app.listen(3000);
```

## API

### `DevPulse.init(config)`

| Option | Default | Description |
|---|---|---|
| `dsn` | *(required)* | `https://<host>/api/ingest/<api_key>` |
| `environment` | `"production"` | Environment tag attached to every event |
| `release` | `null` | Release/version tag (e.g. `"1.2.3"`) |
| `enabled` | `true` | Enable / disable the SDK globally |
| `timeout` | `3000` | HTTP request timeout in milliseconds |
| `maxBreadcrumbs` | `20` | Maximum breadcrumbs retained per event |
| `captureUnhandled` | `true` | Auto-capture `uncaughtException` and `unhandledRejection` |
| `beforeSend` | `null` | Hook to inspect/modify or drop events before sending |

### `DevPulse.capture(error, extra?)`

Manually capture an `Error` or any thrown value.

```js
try {
  await riskyOperation();
} catch (err) {
  await DevPulse.capture(err, { orderId: 42 });
}
```

### `DevPulse.captureMessage(message, level?, extra?)`

Capture a plain string message. `level` defaults to `"info"`.

```js
DevPulse.captureMessage('Payment gateway timeout', 'warning');
```

### `DevPulse.setUser(user)` / `DevPulse.clearUser()`

Attach user identity to all subsequent events.

```js
DevPulse.setUser({ id: 123, email: 'user@example.com' });

// Later, on logout:
DevPulse.clearUser();
```

### `DevPulse.addBreadcrumb(crumb)`

Manually add a breadcrumb to the trail included with the next event.

```js
DevPulse.addBreadcrumb({
  category: 'db',
  message: 'SELECT users WHERE id = ?',
  level: 'info',
});
```

### `DevPulse.errorHandler()`

Returns an Express/Connect-compatible 4-argument error handler middleware. Captures the error, attaches request context (method, URL, scrubbed headers), then calls `next(err)` to pass the error along.

```js
app.use(DevPulse.errorHandler());
```

Request headers are automatically scrubbed — `authorization`, `cookie`, `set-cookie`, and `x-api-key` are replaced with `[redacted]` before being sent.

### `beforeSend` hook

Inspect or modify the payload before it is sent. Return `null` or `false` to drop the event entirely.

```js
DevPulse.init({
  dsn: '...',
  beforeSend(event) {
    // Drop health-check noise
    if (event.request?.url?.includes('/health')) return null;
    return event;
  },
});
```

## TypeScript

The package ships with TypeScript declarations (`src/index.d.ts`). No `@types/` package needed.

```ts
import { DevPulse, DevPulseNodeConfig } from '@sekolahcode/devpulse-node';

const config: DevPulseNodeConfig = {
  dsn: 'https://your-devpulse-host/api/ingest/YOUR_API_KEY',
  environment: 'production',
};

DevPulse.init(config);
```

## License

MIT — see [LICENSE](../../LICENSE)
