> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Add Disk User

> Adds an authorized user to a disk. Users can authenticate via:
- **token**: A shared token with a nickname and 4-character suffix
- **awssts**: AWS STS role assumption with an IAM principal ARN
- **s3**: An access key pair for the disk's
  [S3 API](/protocols/s3-api). The server generates both keys and
  returns the secret key only in this response. The keys reach the
  whole disk, read-write unless `readOnly` is set, so keep them on
  your server and give browsers and other untrusted clients
  presigned URLs instead.




## OpenAPI

````yaml POST /api/disks/{id}/users
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


    All endpoints require an API key, sent as a bearer token:


    ```

    Authorization: Bearer key-...

    ```


    Use the full key, including its `key-` prefix. The older form without the
    `Bearer` scheme is still accepted.


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
  /api/disks/{id}/users:
    post:
      tags:
        - Disk Users
      summary: Add user to disk
      description: |
        Adds an authorized user to a disk. Users can authenticate via:
        - **token**: A shared token with a nickname and 4-character suffix
        - **awssts**: AWS STS role assumption with an IAM principal ARN
        - **s3**: An access key pair for the disk's
          [S3 API](/protocols/s3-api). The server generates both keys and
          returns the secret key only in this response. The keys reach the
          whole disk, read-write unless `readOnly` is set, so keep them on
          your server and give browsers and other untrusted clients
          presigned URLs instead.
      operationId: addDiskUser
      parameters:
        - $ref: '#/components/parameters/DiskId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/DiskUser'
      responses:
        '201':
          description: User added successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_AuthorizedUser'
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '500':
          $ref: '#/components/responses/InternalError'
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
    DiskUser:
      oneOf:
        - $ref: '#/components/schemas/TokenUser'
        - $ref: '#/components/schemas/AwsStsUser'
        - $ref: '#/components/schemas/S3User'
      discriminator:
        propertyName: type
        mapping:
          token: '#/components/schemas/TokenUser'
          awssts: '#/components/schemas/AwsStsUser'
          s3: '#/components/schemas/S3User'
    ApiResponse_AuthorizedUser:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/AuthorizedUser'
    TokenUser:
      type: object
      required:
        - type
        - nickname
      properties:
        type:
          type: string
          enum:
            - token
        principal:
          type: string
          maxLength: 2048
          deprecated: true
          description: >
            Deprecated. Client-provided token. If omitted, the server generates
            a cryptographically secure token and returns it in the response.
        nickname:
          type: string
          maxLength: 255
        tokenSuffix:
          type: string
          minLength: 4
          maxLength: 4
          deprecated: true
          description: >
            Deprecated. Last 4 characters of the token. Required when principal
            is provided; ignored when the server generates the token.
        ttl:
          type: string
          description: >
            Optional time-to-live as a Go duration string (e.g. "1h", "30m",
            "24h"). Maximum 365 days ("8760h"). Mutually exclusive with
            expiresAt.
        expiresAt:
          type: string
          format: date-time
          description: >
            Optional absolute expiration time as an RFC3339 timestamp (e.g.
            "2026-07-15T00:00:00Z"). Must be in the future. Mutually exclusive
            with ttl.
        oneUse:
          type: boolean
          description: >
            When true, the token can be used for only one mount. Later mount
            attempts with it are rejected.
        readOnly:
          type: boolean
          description: |
            When true, mounts made with this token reject writes with EROFS.
    AwsStsUser:
      type: object
      required:
        - type
        - principal
      properties:
        type:
          type: string
          enum:
            - awssts
        principal:
          type: string
          description: IAM principal ARN
          maxLength: 2048
        ttl:
          type: string
          description: >
            Optional time-to-live as a Go duration string (e.g. "1h", "30m",
            "24h"). Maximum 365 days ("8760h"). Mutually exclusive with
            expiresAt.
        expiresAt:
          type: string
          format: date-time
          description: >
            Optional absolute expiration time as an RFC3339 timestamp (e.g.
            "2026-07-15T00:00:00Z"). Must be in the future. Mutually exclusive
            with ttl.
        oneUse:
          type: boolean
          description: |
            When true, the credential can be used for only one mount.
        readOnly:
          type: boolean
          description: >
            When true, mounts made with this credential reject writes with
            EROFS.
    S3User:
      type: object
      required:
        - type
      properties:
        type:
          type: string
          enum:
            - s3
        nickname:
          type: string
          maxLength: 255
        ttl:
          type: string
          description: >
            Optional time-to-live as a Go duration string (e.g. "1h", "30m").
            Maximum 365 days ("8760h"). Requests signed with an expired key,
            including presigned URLs, are rejected. Mutually exclusive with
            expiresAt.
        expiresAt:
          type: string
          format: date-time
          description: >
            Optional absolute expiration time as an RFC3339 timestamp. Must be
            in the future. Mutually exclusive with ttl.
        readOnly:
          type: boolean
          description: >
            When true, the keys can only make GET and HEAD requests. Writes,
            deletes, and multipart uploads return AccessDenied.
    AuthorizedUser:
      type: object
      properties:
        type:
          type: string
          enum:
            - token
            - awssts
            - s3
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
        s3AccessKeyId:
          type: string
          description: The S3 access key ID. Only present for s3 users.
        s3SecretKey:
          type: string
          description: >
            The generated S3 secret access key. Only present in the creation
            response for s3 users. This value is shown exactly once and cannot
            be retrieved again.
        identifier:
          type: string
          description: >
            Stable identifier for this user, returned in creation and list
            responses. Use this value with DELETE
            /api/disks/{id}/users/{type}?identifier={identifier} to remove the
            user. For awssts users, this is the IAM ARN; for s3 users, the
            access key ID.
        createdAt:
          type: string
          format: date-time
        expiresAt:
          type: string
          format: date-time
          description: When the user expires. Absent for users without an expiry.
        oneUse:
          type: boolean
          description: Whether the user can be used for only one mount.
        consumedAt:
          type: string
          format: date-time
          description: |
            When a one-use user was consumed by its mount. Absent until then.
        readOnly:
          type: boolean
          description: Whether the user grants read-only access.
        status:
          type: string
          enum:
            - active
            - expired
            - consumed
            - in_use
          description: >
            Current lifecycle status. `in_use` means a one-use user is being
            consumed by a mount that is still in progress.
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
      type: http
      scheme: bearer
      description: Archil API key, including its `key-` prefix

````

This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
