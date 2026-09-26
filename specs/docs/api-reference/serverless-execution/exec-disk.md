> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Execute Command on Disk

> Launches a container with the specified disk mounted, runs the given
command to completion, shuts down the container, and returns the command
result.




## OpenAPI

````yaml POST /api/disks/{id}/exec
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
  /api/disks/{id}/exec:
    post:
      tags:
        - Serverless Execution
      summary: Execute a command on a disk
      description: |
        Launches a container with the specified disk mounted, runs the given
        command to completion, shuts down the container, and returns the command
        result.
      operationId: execDisk
      parameters:
        - $ref: '#/components/parameters/DiskId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ExecDiskRequest'
      responses:
        '200':
          description: Command completed
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_ExecDisk'
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
  parameters:
    DiskId:
      name: id
      in: path
      required: true
      description: Disk ID (format `dsk-{16 hex chars}`)
      schema:
        type: string
        pattern: ^dsk-[0-9a-f]{16}$
        example: dsk-0123456789abcdef
  schemas:
    ExecDiskRequest:
      type: object
      required:
        - command
      properties:
        command:
          type: string
          description: Shell command to execute inside the container
          example: ls -la /mnt/archil
    ApiResponse_ExecDisk:
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
