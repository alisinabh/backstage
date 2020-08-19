---
id: call-existing-api
title: Call Existing API
---

This article describes the various options that Backstage frontend plugins have,
in communicating with service APIs that already exist. Each section below
describes a possible choice, and the circumstances under which it fits.

## Issuing Requests Directly

The most basic choice available is to issue requests directly from the plugin
frontend code to the API, using for example `fetch` or a support library such as
`axios`.

Example:

```ts
// Inside your component
fetch('https://frobs.example.com/list')
  .then(response => response.json())
  .then(payload => setFrobs(payload as Frob[]));
```

This can be used when:

- The API already does/exposes exactly what you need.
- The request/response patterns of the API match real world usage needs in
  Backstage frontend plugins. For example, if the end use case is to show a
  small summary in Backstage, but the only available API endpoint gives a 30
  megabyte blob with large amounts of redundant information, it would hurt the
  end user experience. Particularly on mobile. The same goes for cases where you
  want to show many individual pieces of information: if a common use case is to
  show large tables where one API request per cell is necessary, the browser
  will quickly become swamped and you may want to consider performing
  aggregation elsewhere instead.
- The API can maintain interactive request/response times at your required peak
  request rates. The end user experience will be degraded if they spend a lot of
  time waiting for the data to arrive.
- The API is exposed over HTTPS (not just HTTP), and properly handles
  [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS). These are
  requirements that the user's browser will impose for security reasons, and the
  requests will be rejected otherwise.
- The API endpoint is easily reachable, in terms of network conditions, by end
  users. This may be particularly relevant if your end users are outside of your
  perimeter.
- The API endpoint is highly available. The browser does not have builtin
  facilities for service discovery, retries, health checks, circuit breaking and
  similar. If the endpoint is frequently down even for short periods of time
  (e.g. during deploys), end users will quickly notice.
- The requests do not require secrets to be passed. This limitation does not
  apply to OAuth tokens, which the frontend can negotiate and make proper use
  of.

## Using The Backstage Proxy

Backstage has an optional proxy plugin for the backend, that can be used to
easily add proxy routes to downstream APIs.

Example:

```yaml
# In app-config.yaml
proxy:
  '/circleci/api':
    target: 'https://circleci.com/api/v1.1'
    changeOrigin: true
    pathRewrite:
      '^/proxy/circleci/api/': '/'
```

```ts
// Inside your component
const backendUrl = config.getString('backend.baseUrl');
fetch(`${backendUrl}/proxy/circleci/api/x/y/z`)
  .then(response => response.json())
  .then(payload => setBuilds(payload as SomeCircleciType[]));
```

## Creating a Backstage Backend Plugin

## Extending the GraphQL Model
