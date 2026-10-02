> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Create Sandbox

> Provisions a sandbox VM with the requested shape and a dedicated,
persistent Archil disk. By default the response reports `pending` after
the runtime accepts the start. Set `wait=true` to hold for `running`; if
the wait budget expires, the response remains `pending` and startup
continues.




## OpenAPI

````yaml POST /api/sandboxes
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
  /api/sandboxes:
    post:
      tags:
        - Sandboxes
      summary: Create a sandbox
      description: |
        Provisions a sandbox VM with the requested shape and a dedicated,
        persistent Archil disk. By default the response reports `pending` after
        the runtime accepts the start. Set `wait=true` to hold for `running`; if
        the wait budget expires, the response remains `pending` and startup
        continues.
      operationId: createSandbox
      parameters:
        - $ref: '#/components/parameters/Wait'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateSandboxRequest'
      responses:
        '202':
          description: The sandbox was created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_Sandbox'
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '409':
          description: The requested sandbox name already exists in the account
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '500':
          $ref: '#/components/responses/InternalError'
        '503':
          description: No sandbox capacity is available; retry
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
components:
  parameters:
    Wait:
      name: wait
      in: query
      required: false
      description: Hold the request for a completed sandbox lifecycle transition
      schema:
        type: boolean
        default: false
  schemas:
    CreateSandboxRequest:
      type: object
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 63
          pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
          description: >-
            Sandbox name, unique within the account. A random word-list name is
            generated when omitted.
        vcpu_count:
          type: integer
          minimum: 1
          maximum: 32
          default: 1
        mem_size_mib:
          type: integer
          minimum: 256
          maximum: 65536
          default: 2048
        base_image:
          type: string
          default: ubuntu:26.04
          description: >-
            Public Linux OCI image reference. Docker shorthand and tags are
            accepted; the selected platform manifest is pinned at creation.
        ports:
          type: array
          description: TCP ports to expose publicly when the sandbox is created.
          items:
            type: integer
            minimum: 1
            maximum: 65535
        enable_service_ingress:
          type: boolean
          default: false
          description: >-
            Allow services inside the sandbox to expose ingress. When false,
            services still run but their ports must be exposed explicitly
            through the API. Retained across starts, resumes, and forks.
        env:
          type: object
          additionalProperties:
            type: string
          description: Environment variables applied to every process
        network:
          $ref: '#/components/schemas/SandboxNetwork'
        max_ttl_seconds:
          type: integer
          minimum: 60
          maximum: 86400
          default: 86400
          description: >-
            Lifetime budget applied independently to each powered-on session.
            Expiry pauses the sandbox, preserving memory and processes for
            resume.
        idle_ttl_seconds:
          type: integer
          minimum: 0
          maximum: 86400
          description: >-
            Pause after this many seconds without a direct process connection,
            preserving memory and processes for resume. Omit or set to zero to
            disable idle expiry.
        max_concurrent_execs:
          type: integer
          description: >-
            Maximum number of concurrently attached process sessions. Detached
            processes and one-shot process controls do not count.
    ApiResponse_Sandbox:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/Sandbox'
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
    SandboxNetwork:
      type: object
      description: |
        Sandbox network policy. New sandboxes on free plans receive deny-all
        egress with no allowlist exceptions. Start/resume and forks retain the
        stored policy. For paid accounts, egress is unrestricted when omitted.
      properties:
        egress:
          $ref: '#/components/schemas/SandboxEgressPolicy'
    Sandbox:
      type: object
      required:
        - sandbox_id
        - name
        - status
        - vcpu_count
        - mem_size_mib
        - max_ttl_seconds
        - idle_ttl_seconds
        - max_concurrent_execs
        - base_image
        - created_at
        - last_active_at
      properties:
        sandbox_id:
          type: string
          format: uuid
        name:
          type: string
          minLength: 1
          maxLength: 63
          pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
          description: Sandbox name.
        status:
          $ref: '#/components/schemas/SandboxState'
        vcpu_count:
          type: integer
        mem_size_mib:
          type: integer
        max_ttl_seconds:
          type: integer
          description: >-
            Lifetime budget applied independently to each powered-on session.
            Expiry pauses the sandbox, preserving memory and processes for
            resume. Defaults to 24 hours and can be reset with the timeout
            endpoint.
        idle_ttl_seconds:
          type: integer
          description: >-
            Seconds without a direct process connection before the sandbox
            pauses, preserving memory and processes for resume. Zero disables
            idle expiry.
        max_concurrent_execs:
          type: integer
          description: >-
            Maximum number of concurrently attached process sessions. Detached
            processes and one-shot process controls do not count.
        base_image:
          type: string
          description: OCI reference requested when the sandbox was created.
        platform:
          type: string
          enum:
            - arm64
            - amd64
          description: Sandbox CPU architecture.
        endpoints:
          type: array
          description: Public hostnames published by enabled sandbox services.
          items:
            $ref: '#/components/schemas/SandboxEndpoint'
        enable_service_ingress:
          type: boolean
          default: false
          description: >-
            Whether services inside the sandbox can expose ingress. Explicit API
            port exposure remains available regardless of this setting.
        created_at:
          type: string
          format: date-time
        running_at:
          type: string
          format: date-time
        finished_at:
          type: string
          format: date-time
        last_active_at:
          type: string
          format: date-time
        exit_reason:
          type: string
        checkpoint:
          type: string
          description: |
            Disk checkpoint the current session leaves behind. Present while
            pausing, paused, stopping, or stopped, and committed once the
            sandbox is paused or stopped. Pass it to the fork endpoint to fork
            exactly this state.
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
    SandboxState:
      type: string
      enum:
        - pending
        - running
        - pausing
        - paused
        - stopping
        - stopped
        - exited
        - failed
        - deleting
        - deleted
    SandboxEndpoint:
      type: object
      required:
        - port
        - hostname
      properties:
        port:
          type: integer
          minimum: 1
          maximum: 65535
        hostname:
          type: string
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
    ValidationError:
      description: Validation error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    Unauthorized:
      description: Invalid or missing authentication credentials
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
  securitySchemes:
    ApiKeyAuth:
      type: apiKey
      in: header
      name: Authorization
      description: API key

````

This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
