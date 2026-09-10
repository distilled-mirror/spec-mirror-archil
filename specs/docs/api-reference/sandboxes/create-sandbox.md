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
        '429':
          description: Sandbox capacity reached
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '500':
          $ref: '#/components/responses/InternalError'
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
            accepted.
        env:
          type: object
          additionalProperties:
            type: string
          description: Environment variables applied to every process
        max_ttl_seconds:
          type: integer
          minimum: 60
          maximum: 28800
          default: 28800
          description: >-
            Lifetime budget for each powered-on session. Activity does not
            extend the deadline; starting or resuming the sandbox begins a fresh
            session.
        max_concurrent_execs:
          type: integer
          minimum: 1
          maximum: 256
          default: 32
          description: Maximum concurrently attached process sessions
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
    Sandbox:
      type: object
      required:
        - sandbox_id
        - name
        - status
        - vcpu_count
        - mem_size_mib
        - max_ttl_seconds
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
            Lifetime budget for each powered-on session. Activity does not
            extend the deadline; starting or resuming the sandbox begins a fresh
            session.
        max_concurrent_execs:
          type: integer
          description: Maximum concurrently attached process sessions
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
          items:
            $ref: '#/components/schemas/SandboxEndpoint'
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
        expires_at:
          type: string
          format: date-time
          description: Current powered-on session deadline; absent while inactive.
        exit_reason:
          type: string
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
