> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# API Reference

> Programmatic access to the Archil Control Plane

The Archil Control Plane API provides programmatic access to manage disks, mounts, and API keys in the Archil distributed filesystem platform.

<Info>
  **API keys vs. disk tokens.** API keys authenticate requests to this Control Plane API and are scoped to your account. [Disk tokens](/concepts/disk-users#disk-token-authorization) are a separate kind of credential, scoped to a single disk, used by clients when mounting. The two are not interchangeable.
</Info>

## Base URL

Each region has its own control plane endpoint:

| Region | Region ID | Endpoint |
| - | - | - |
| AWS US East (N. Virginia) | `aws-us-east-1` | `control.green.us-east-1.aws.prod.archil.com` |
| AWS EU West (Ireland) | `aws-eu-west-1` | `control.green.eu-west-1.aws.prod.archil.com` |
| AWS US West (Oregon) | `aws-us-west-2` | `control.green.us-west-2.aws.prod.archil.com` |
| GCP US Central (Iowa) | `gcp-us-central1` | `control.blue.us-central1.gcp.prod.archil.com` |

## Authentication

All endpoints require an API key:

```
Authorization: {API_KEY}
```

Create API keys in the [Archil Console](https://console.archil.com) or via the API.

## Response format

All responses use a consistent envelope:

<CodeGroup>
  ```json Success Response theme={null}
  {
    "success": true,
    "data": { ... }
  }
  ```

  ```json Error Response theme={null}
  {
    "success": false,
    "error": "Error message"
  }
  ```
</CodeGroup>

## HTTP status codes

| Code | Description |
| - | - |
| `200` | Success |
| `202` | Accepted; an asynchronous operation started |
| `400` | Bad request (validation error) |
| `403` | Forbidden (auth failure or access denied) |
| `404` | Not found |
| `409` | Resource state conflict |
| `429` | Capacity or concurrency limit reached |
| `500` | Internal server error |
| `503` | Feature unavailable or lifecycle transition temporarily blocked |

## Disk ID format

Disks are identified by a unique ID in the format:

```
dsk-{16 hex characters}
```

For example: `dsk-0123456789abcdef`

## Getting started

<Note>
  Signing up for Archil is free with no credit card required. [Create your free account](https://console.archil.com) to get an API key and start building.
</Note>

1. [Create an API key](/api-reference/tokens/create-token) in the console or via API
2. Use the key in the `Authorization` header for all requests
3. [Create your first disk](/api-reference/disks/create-disk) or [persistent sandbox](/api-reference/sandboxes/create-sandbox)


This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
