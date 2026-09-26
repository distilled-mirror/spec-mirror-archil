> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# List Disks

> Returns all disks owned by the authenticated account.



## OpenAPI

````yaml GET /api/disks
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
  /api/disks:
    get:
      tags:
        - Disks
      summary: List all disks
      description: Returns all disks owned by the authenticated account.
      operationId: listDisks
      parameters:
        - $ref: '#/components/parameters/Limit'
        - $ref: '#/components/parameters/Cursor'
        - name: name
          in: query
          description: Filter disks by exact name match.
          required: false
          schema:
            type: string
      responses:
        '200':
          description: List of disks
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_DiskList'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '500':
          $ref: '#/components/responses/InternalError'
components:
  parameters:
    Limit:
      name: limit
      in: query
      required: false
      description: Maximum number of items to return
      schema:
        type: integer
        default: 50
        maximum: 100
    Cursor:
      name: cursor
      in: query
      required: false
      description: Pagination cursor from a previous response
      schema:
        type: string
  schemas:
    ApiResponse_DiskList:
      description: >
        All API responses use a standard envelope with `success: boolean` and
        `data` (on success) or `error: string` (on failure). The ApiResponse_*
        schemas each define the specific `data` shape for their endpoint.
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          type: array
          items:
            $ref: '#/components/schemas/DiskResponse'
    DiskResponse:
      type: object
      required:
        - id
        - name
        - organization
        - status
        - provider
        - region
        - createdAt
      properties:
        id:
          type: string
          description: Disk ID
          example: dsk-0123456789abcdef
        name:
          type: string
          description: Disk name
        organization:
          type: string
          description: Owning organization ID
        status:
          type: string
          description: Disk status
          enum:
            - available
            - creating
            - deleting
            - deleted
            - failed
        fsHandlerStatus:
          type: string
          description: Filesystem handler status
        provider:
          type: string
          description: Cloud provider
        region:
          type: string
          description: Disk region (e.g., aws-us-east-1, gcp-us-central1)
        createdAt:
          type: string
          format: date-time
          description: Creation timestamp
        lastAccessed:
          type: string
          format: date-time
          description: Last access timestamp
        dataSize:
          type: integer
          format: int64
          minimum: 0
          description: Total data size in bytes
        monthlyUsage:
          type: string
          description: Monthly usage amount formatted as a currency string (e.g., "$1.23")
        mounts:
          type: array
          items:
            $ref: '#/components/schemas/MountResponse'
        metrics:
          $ref: '#/components/schemas/DiskMetrics'
        connectedClients:
          type: array
          items:
            $ref: '#/components/schemas/ConnectedClient'
        authorizedUsers:
          type: array
          items:
            $ref: '#/components/schemas/AuthorizedUser'
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
    MountResponse:
      type: object
      properties:
        id:
          type: string
          description: Mount identifier
        type:
          type: string
          description: Storage backend type
          enum:
            - s3
            - gcs
            - r2
            - s3-compatible
            - azure-blob
        path:
          type: string
          description: Mount path
        name:
          type: string
          description: Bucket/container name
        accessMode:
          type: string
          description: Access mode
          enum:
            - rw
            - ro
          example: rw
        config:
          $ref: '#/components/schemas/MountConfigResponse'
        connectionStatus:
          type: string
          description: Current connection status
          enum:
            - connected
            - disconnected
        authError:
          type: string
          description: Authentication error message (if disconnected)
        authorizationType:
          type: string
          description: How the mount authenticates to the storage backend
          enum:
            - iam
            - accessKeys
            - oauth
    DiskMetrics:
      type: object
      properties:
        dataTransfer:
          type: string
          description: Data transfer amount with unit (e.g., "1.5 GB")
        requests:
          type: string
          description: Total request count as a formatted string (e.g., "1,234")
        avgResponseTime:
          type: string
          description: Average response time with unit (e.g., "45ms")
    ConnectedClient:
      type: object
      properties:
        id:
          type: string
        ipAddress:
          type: string
        connectedAt:
          type: string
          format: date-time
    AuthorizedUser:
      type: object
      properties:
        type:
          type: string
          enum:
            - token
            - awssts
        principal:
          type: string
          deprecated: true
          description: >
            Use identifier instead. Only populated for awssts type (the IAM
            ARN).
        nickname:
          type: string
        tokenSuffix:
          type: string
        token:
          type: string
          description: >
            The generated disk token (used by clients when mounting the disk).
            Only present in the response when the server generates the token
            (i.e. principal was not provided). This value is shown exactly once
            and cannot be retrieved again.
        identifier:
          type: string
          description: >
            Stable identifier for this user, returned in creation and list
            responses. Use this value with DELETE
            /api/disks/{id}/users/{type}?identifier={identifier} to remove the
            user. For awssts users, this is the IAM ARN.
        createdAt:
          type: string
          format: date-time
    MountConfigResponse:
      type: object
      description: Mount configuration details (secrets omitted)
      properties:
        bucketName:
          type: string
          description: Bucket name
        bucketEndpoint:
          type: string
          description: Storage endpoint URL
        bucketPrefix:
          type: string
          description: Prefix within the bucket
        sessionId:
          type: string
          description: Session identifier for IAM-authorized mounts
  responses:
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
