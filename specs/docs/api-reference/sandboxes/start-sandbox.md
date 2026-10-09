> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Start Sandbox

> Cold-boots from the persisted configuration and disks. If the sandbox
was paused, its memory snapshot is consumed and discarded before the
new VM runs; disk state remains. Without mount changes, starting a running
sandbox or a pending cold start is idempotent. A sandbox that is stopping,
pausing, or pending a resume returns 409. Providing `mounts` replaces the
additional disk mounts for this and later sessions and requires an inactive
sandbox; omit it to preserve saved mounts.




## OpenAPI

````yaml POST /api/sandboxes/{sid}/start
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
  /api/sandboxes/{sid}/start:
    post:
      tags:
        - Sandboxes
      summary: Cold-start a sandbox
      description: >
        Cold-boots from the persisted configuration and disks. If the sandbox

        was paused, its memory snapshot is consumed and discarded before the

        new VM runs; disk state remains. Without mount changes, starting a
        running

        sandbox or a pending cold start is idempotent. A sandbox that is
        stopping,

        pausing, or pending a resume returns 409. Providing `mounts` replaces
        the

        additional disk mounts for this and later sessions and requires an
        inactive

        sandbox; omit it to preserve saved mounts.
      operationId: startSandbox
      parameters:
        - $ref: '#/components/parameters/SandboxId'
        - $ref: '#/components/parameters/Wait'
      requestBody:
        required: false
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/StartSandboxRequest'
            example:
              mounts:
                - disk_id: dsk-0123456789abcdef
                  path: /mnt/data
      responses:
        '200':
          description: The sandbox is running
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_Sandbox'
              example:
                success: true
                data:
                  sandbox_id: 019d158e-7100-7000-8000-0123456789ab
                  name: agent-workspace
                  status: running
                  vcpu_count: 2
                  mem_size_mib: 4096
                  max_ttl_seconds: 86400
                  idle_ttl_seconds: 0
                  max_concurrent_execs: 32
                  enable_service_ingress: false
                  base_image: python:3.13
                  mounts:
                    - disk_id: dsk-0123456789abcdef
                      path: /mnt/data
                  created_at: '2026-10-06T12:00:00Z'
                  last_active_at: '2026-10-06T12:00:05Z'
                  running_at: '2026-10-06T12:00:05Z'
        '202':
          description: The sandbox start is pending
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_Sandbox'
              example:
                success: true
                data:
                  sandbox_id: 019d158e-7100-7000-8000-0123456789ab
                  name: agent-workspace
                  status: pending
                  vcpu_count: 2
                  mem_size_mib: 4096
                  max_ttl_seconds: 86400
                  idle_ttl_seconds: 0
                  max_concurrent_execs: 32
                  enable_service_ingress: false
                  base_image: python:3.13
                  mounts:
                    - disk_id: dsk-0123456789abcdef
                      path: /mnt/data
                  created_at: '2026-10-06T12:00:00Z'
                  last_active_at: '2026-10-06T12:00:00Z'
        '400':
          $ref: '#/components/responses/SandboxValidationError'
        '401':
          $ref: '#/components/responses/PlainTextUnauthorized'
        '404':
          description: Sandbox or mount disk not found or not accessible to this account
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                sandboxNotFound:
                  summary: Sandbox not found
                  value:
                    success: false
                    error: Sandbox not found
                    code: not_found
                diskNotFound:
                  summary: Mount disk not found or not owned by this account
                  value:
                    success: false
                    error: 'mounts[0]: disk dsk-0123456789abcdef not found'
                    code: not_found
        '409':
          description: >-
            Sandbox is stopping, pausing, being deleted, or resuming; mount
            changes also conflict with a running or pending session
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                mountsWhileActive:
                  summary: Mount changes require an inactive sandbox
                  value:
                    success: false
                    error: >-
                      mounts can only change on a cold start of an inactive
                      sandbox
                    code: sandbox_mounts_while_active
                stopping:
                  summary: Sandbox is stopping
                  value:
                    success: false
                    error: Sandbox is still stopping; retry once it is stopped
                    code: sandbox_stopping
                pausing:
                  summary: Sandbox is pausing
                  value:
                    success: false
                    error: Sandbox is still pausing; retry once it is paused
                    code: sandbox_pausing
                startConflict:
                  summary: Resume already in progress
                  value:
                    success: false
                    error: >-
                      Sandbox is already starting with a different lifecycle
                      intent
                    code: sandbox_start_conflict
                terminated:
                  summary: Sandbox is being deleted
                  value:
                    success: false
                    error: Sandbox is no longer accepting requests
                    code: sandbox_terminated
        '500':
          $ref: '#/components/responses/SandboxInternalError'
        '503':
          $ref: '#/components/responses/SandboxUnavailable'
        '504':
          $ref: '#/components/responses/SandboxTimeout'
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
    Wait:
      name: wait
      in: query
      required: false
      description: Hold the request for a completed sandbox lifecycle transition
      schema:
        type: boolean
        default: false
  schemas:
    StartSandboxRequest:
      type: object
      properties:
        mounts:
          type: array
          description: >-
            Replace the sandbox's additional disk mounts for this and later
            sessions. Omit to keep the current ones; an empty list removes all
            additional mounts without deleting their disks. The internal root
            disk is unaffected. Not accepted by resume, and 409 while the
            sandbox is running or pending.
          items:
            $ref: '#/components/schemas/SandboxMount'
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
    SandboxMount:
      type: object
      required:
        - disk_id
      properties:
        disk_id:
          type: string
          pattern: ^dsk-[0-9a-f]{16}$
          description: >-
            Existing disk owned by your account in the sandbox's region. A
            sandbox's internal root disk cannot be used here.
        path:
          type: string
          description: >-
            Absolute guest directory to mount at. Required when more than one
            disk is mounted; a sole mount defaults to /mnt/archil. Paths cannot
            overlap or contain whitespace, control characters, empty components,
            . or .. components, or a trailing slash. The root directory and
            guest system directories (/usr, /etc, /var, /opt, /dev, ...) and
            their descendants are reserved. A disk can be mounted once per
            sandbox.
        subdirectory:
          type: string
          description: Relative subdirectory of the disk to expose instead of its root.
        read_only:
          type: boolean
          default: false
        conditional:
          type: boolean
          default: false
          description: >-
            Send mutating operations directly to the server without a delegation
            checkout, allowing concurrent writers.
        queue_ms:
          type: integer
          minimum: 1
          description: >-
            Milliseconds to wait for the disk's exclusive root delegation before
            the mount fails. Not allowed with read_only or conditional.
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
            resume. Defaults to 24 hours. Timeout resets cannot extend a session
            beyond 24 hours.
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
          description: >-
            OCI reference requested when the sandbox was created. Empty for a
            sandbox created from `image_id`.
        image_digest:
          type: string
          description: Image digest the sandbox was created from, if any.
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
        mounts:
          type: array
          items:
            $ref: '#/components/schemas/SandboxMount'
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
  responses:
    SandboxValidationError:
      description: Invalid request body or parameters
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
          example:
            success: false
            error: Invalid request body
            code: bad_request
    PlainTextUnauthorized:
      description: Missing or invalid authentication credentials
      content:
        text/plain:
          schema:
            type: string
          examples:
            missingAuthorization:
              summary: Missing Authorization header
              value: Must provide an Authorization header
            invalidCredentials:
              summary: Invalid credentials
              value: Unauthorized
    SandboxInternalError:
      description: Internal server error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
          example:
            success: false
            error: Internal error processing sandbox request
            code: internal_server_error
    SandboxUnavailable:
      description: >-
        Sandbox capacity or runtime temporarily unavailable, or sandboxes are
        disabled
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
          examples:
            noCapacity:
              summary: No capacity
              value:
                success: false
                error: No sandbox capacity is available; retry
                code: no_capacity
            runtimeRetryable:
              summary: Retryable runtime failure
              value:
                success: false
                error: Sandbox lifecycle request is temporarily blocked; retry
                code: runtime_retryable
            notEnabled:
              summary: Sandboxes disabled
              value:
                success: false
                error: Sandboxes are not enabled on this controlplane
                code: not_enabled
    SandboxTimeout:
      description: Request timed out; retry
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
          example:
            success: false
            error: Sandbox request timed out; retry
            code: timeout
  securitySchemes:
    ApiKeyAuth:
      type: apiKey
      in: header
      name: Authorization
      description: API key

````

This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
