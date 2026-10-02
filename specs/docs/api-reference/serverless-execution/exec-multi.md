> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Execute Command with Multiple Disks

> Launches a container with the supplied set of disks each mounted at its
own relative path under `/mnt/archil`, runs the command to completion,
and shuts down the container. Activation is atomic: every disk mounts
or none of them do.

Relative paths must be non-empty, non-absolute, and contain no `.` /
`..` segments. Mounting two disks at the same relative path is an
error.




## OpenAPI

````yaml POST /api/exec
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
  /api/exec:
    post:
      tags:
        - Serverless Execution
      summary: Execute a command with multiple disks mounted
      description: |
        Launches a container with the supplied set of disks each mounted at its
        own relative path under `/mnt/archil`, runs the command to completion,
        and shuts down the container. Activation is atomic: every disk mounts
        or none of them do.

        Relative paths must be non-empty, non-absolute, and contain no `.` /
        `..` segments. Mounting two disks at the same relative path is an
        error.
      operationId: exec
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ExecRequest'
      responses:
        '200':
          description: Command completed
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_Exec'
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'
        '500':
          $ref: '#/components/responses/InternalError'
        '504':
          description: Command timed out
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
components:
  schemas:
    ExecRequest:
      type: object
      required:
        - disks
        - command
      properties:
        disks:
          type: object
          description: |
            Map of relative path under `/mnt/archil` to the disk to mount
            there. At least one entry is required. Relative paths must be
            non-empty, non-absolute, and contain no `.` or `..` segments.

            Each value is either a plain disk ID string (mounts the disk's
            root, read-write) or an object that additionally selects a
            subdirectory of the disk and/or marks the mount as read-only.
          additionalProperties:
            oneOf:
              - type: string
                description: Disk ID; mounts the disk's root as read-write
                example: dsk-abc123
              - $ref: '#/components/schemas/ExecMount'
          example:
            data: dsk-abc123
            logs:
              disk: dsk-def456
              subdirectory: app/logs
              readOnly: true
        command:
          type: string
          description: Shell command to execute inside the container
          example: ls -la /mnt/archil/data /mnt/archil/logs
    ApiResponse_Exec:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/ExecDiskResult'
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
    ExecMount:
      type: object
      required:
        - disk
      properties:
        disk:
          type: string
          description: Disk ID to mount at this relative path
          example: dsk-abc123
        subdirectory:
          type: string
          description: |
            Subdirectory of the disk to expose at the mountpoint. Must be a
            relative path with no `.` or `..` segments. When omitted, the
            disk's root is exposed.
          example: app/logs
        readOnly:
          type: boolean
          default: false
          description: |
            When true, the disk is mounted read-only inside the container.
            Writes against the mount fail with EROFS.
    ExecDiskResult:
      type: object
      required:
        - exitCode
        - stdout
        - stderr
        - timing
      properties:
        exitCode:
          type: integer
          description: Exit code of the command (0 = success)
          example: 0
        stdout:
          type: string
          description: Standard output from the command
        stderr:
          type: string
          description: Standard error from the command
        timing:
          $ref: '#/components/schemas/ExecTiming'
    ExecTiming:
      type: object
      description: Server-measured timings for an exec request.
      required:
        - totalMs
        - queueMs
        - executeMs
      properties:
        totalMs:
          type: integer
          description: >
            End-to-end wall clock measured on the server, from request arrival
            to response.
          example: 2450
        queueMs:
          type: integer
          description: >
            Time spent queueing, scheduling, booting/claiming a VM, and mounting
            the filesystem before the command started running.
          example: 150
        executeMs:
          type: integer
          description: |
            Time the user's command itself ran, measured by the runtime.
          example: 2300
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
  securitySchemes:
    ApiKeyAuth:
      type: apiKey
      in: header
      name: Authorization
      description: API key

````

This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
