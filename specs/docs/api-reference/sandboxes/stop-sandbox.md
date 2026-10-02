> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Stop Sandbox

> Gracefully shuts down the guest, then checkpoints its disk before
entering `stopped`. The sandbox and any forks from it cold-boot from
disk state. Stopping a paused sandbox reuses its existing disk
checkpoint. In-flight processes receive the guest shutdown signal.




## OpenAPI

````yaml POST /api/sandboxes/{sid}/stop
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
  /api/sandboxes/{sid}/stop:
    post:
      tags:
        - Sandboxes
      summary: Stop a sandbox
      description: |
        Gracefully shuts down the guest, then checkpoints its disk before
        entering `stopped`. The sandbox and any forks from it cold-boot from
        disk state. Stopping a paused sandbox reuses its existing disk
        checkpoint. In-flight processes receive the guest shutdown signal.
      operationId: stopSandbox
      parameters:
        - $ref: '#/components/parameters/SandboxId'
      responses:
        '200':
          description: The sandbox was already inactive
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_Sandbox'
        '202':
          description: The sandbox stop is in progress
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_Sandbox'
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'
        '409':
          description: A pause is already in progress
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
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
