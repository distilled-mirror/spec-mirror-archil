> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Get Sandbox Network

> Returns the last committed network policy for a running, paused, or stopped sandbox.



## OpenAPI

````yaml GET /api/sandboxes/{sid}/network
openapi: 3.1.0
info:
  title: Archil Control Plane API
  description: >
    The Archil Control Plane API provides programmatic access to manage disks,

    persistent sandboxes, mounts, and API keys in the Archil distributed

    filesystem platform.


    API keys authenticate requests to this control plane and are scoped to

    your account. They are distinct from *disk tokens*, which are per-disk

    credentials used by clients when mounting a disk.


    ## Authentication


    All endpoints require an API key:


    ```

    Authorization: {API_KEY}

    ```


    Create API keys in the [Archil Console](https://console.archil.com) or via
    the API.


    ## Response Format


    All responses use a consistent envelope:


    ```json

    {
      "success": true,
      "data": { ... }
    }

    ```


    Or on error:


    ```json

    {
      "success": false,
      "error": "Error message"
    }

    ```
  version: 1.0.0
  contact:
    email: support@archil.com
    url: https://archil.com
servers:
  - url: https://control.green.us-east-1.aws.prod.archil.com
    description: AWS US East (N. Virginia) — aws-us-east-1
  - url: https://control.green.eu-west-1.aws.prod.archil.com
    description: AWS EU West (Ireland) — aws-eu-west-1
  - url: https://control.green.us-west-2.aws.prod.archil.com
    description: AWS US West (Oregon) — aws-us-west-2
  - url: https://control.blue.us-central1.gcp.prod.archil.com
    description: GCP US Central (Iowa) — gcp-us-central1
security:
  - ApiKeyAuth: []
tags:
  - name: Disks
    description: Create, read, update, and delete disks
  - name: Serverless Execution
    description: Run commands on a disk without provisioning compute
  - name: Sandboxes
    description: Manage persistent sandbox virtual machines
  - name: Disk Users
    description: Manage authorized users on disks
  - name: API Tokens
    description: >-
      Manage API keys (also called API tokens) used to authenticate Control
      Plane API requests. Distinct from disk tokens.
paths:
  /api/sandboxes/{sid}/network:
    get:
      tags:
        - Sandboxes
      summary: Get a sandbox's network policy
      description: >-
        Returns the last committed network policy for a running, paused, or
        stopped sandbox.
      operationId: getSandboxNetwork
      parameters:
        - $ref: '#/components/parameters/SandboxId'
      responses:
        '200':
          description: The last committed network policy
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_SandboxNetwork'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'
        '500':
          $ref: '#/components/responses/InternalError'
        '503':
          $ref: '#/components/responses/RetryableUnavailable'
components:
  parameters:
    SandboxId:
      name: sid
      in: path
      required: true
      description: Sandbox UUID
      schema:
        type: string
        format: uuid
  schemas:
    ApiResponse_SandboxNetwork:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/SandboxNetwork'
    SandboxNetwork:
      type: object
      description: |
        Sandbox network policy. New sandboxes on free plans receive deny-all
        egress with no allowlist exceptions. Start/resume and forks retain the
        stored policy. For paid accounts, egress is unrestricted when omitted.
      properties:
        egress:
          $ref: '#/components/schemas/SandboxEgressPolicy'
    ErrorResponse:
      type: object
      required:
        - success
        - error
      properties:
        success:
          type: boolean
          example: false
        error:
          type: string
          example: Invalid request parameters
        code:
          type: string
          description: Stable machine-readable error code.
    SandboxEgressPolicy:
      type: object
      description: >-
        Deny targets take precedence when an address or domain matches both
        lists. When domain rules or drain selectors are present, TCP ports 80
        and 443 are restricted to HTTP/1.1 or HTTP/2. Plaintext HTTP authority
        is enforced on port 80, but requests matching a transformation rule are
        rejected unless the rule also forwards. Port 443 is TLS-terminated; both
        TLS SNI and HTTP authority are evaluated, and the HTTP authority selects
        the upstream, any request transformations, and any request forwarding.
        DNS to the sandbox's configured resolvers is allowed and UDP port 443 is
        denied.
      required:
        - default
      properties:
        default:
          $ref: '#/components/schemas/SandboxNetworkAction'
        allow:
          type: array
          items:
            oneOf:
              - type: string
              - $ref: '#/components/schemas/SandboxEgressRule'
          description: >-
            Allowed IPv4 addresses, CIDR ranges, exact domains, wildcard domains
            beginning with `*.`, or target objects with optional outbound HTTPS
            request transformations. A wildcard matches subdomains but not the
            apex domain, including when applying transformations.
        deny:
          type: array
          items:
            type: string
          description: >-
            Denied IPv4 addresses, CIDR ranges, exact domains, or wildcard
            domains beginning with `*.`. A wildcard matches subdomains but not
            the apex domain.
        drain_on_pause:
          type: array
          description: >-
            Optional hosts whose HTTP(S) requests should finish before pausing,
            regardless of URL path. Pausing gates new matching requests and
            waits up to ten minutes by default for active responses, including
            streams, before taking a snapshot. Errors stop counting, and expiry
            proceeds with the snapshot. This does not limit requests during
            normal operation. Selectors do not grant network access. Matching
            uses the original request host before transformations or forwarding.
            Only TCP ports 80 and 443 are supported. Connections may need to be
            retried after resume.
          items:
            type: string
            description: >-
              Lowercase hostname pattern without a scheme, port, or path.
              Asterisk matches zero or more characters; *.example.com matches
              subdomains, not example.com. Use * for every host.
            example: bedrock-runtime.*.amazonaws.com
    SandboxNetworkAction:
      type: string
      enum:
        - allow
        - deny
    SandboxEgressRule:
      type: object
      additionalProperties: false
      required:
        - target
      properties:
        target:
          type: string
          description: >-
            Target allowed by this rule. Effective transformations and
            forwarding require an exact or wildcard lowercase domain.
        transform:
          $ref: '#/components/schemas/SandboxEgressTransform'
        forward_url:
          type: string
          format: uri
          description: >-
            Absolute public HTTPS URL that receives this rule's permitted HTTP
            and HTTPS requests instead of their original upstream. The original
            path is appended to this URL, and Archil overwrites the
            archil-forwarded-host, archil-forwarded-scheme,
            archil-forwarded-port, archil-forwarded-path, and archil-sandbox-id
            headers with request metadata. A transform on the same rule is
            applied before forwarding, so the forwarded request carries the
            transformed headers. URLs containing credentials, a query, or a
            fragment are rejected.
    SandboxEgressTransform:
      type: object
      description: >-
        Optional outbound HTTPS request transformations. An omitted or empty
        transform leaves the rule as an ordinary allow without request
        mutations.
      additionalProperties: false
      properties:
        headers:
          type: object
          additionalProperties:
            type: string
          description: >-
            Outbound HTTPS request headers to set, overwriting values supplied
            by the sandbox.
  responses:
    Unauthorized:
      description: Invalid or missing authentication credentials
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    InternalError:
      description: Internal server error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    RetryableUnavailable:
      description: The lifecycle transition is temporarily blocked and can be retried
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
  securitySchemes:
    ApiKeyAuth:
      type: apiKey
      in: header
      name: Authorization
      description: API key

````

This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
