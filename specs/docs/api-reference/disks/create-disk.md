> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Create Disk

> Creates a new disk with the specified configuration. A default disk
token user is automatically generated and returned in the response,
so the disk is immediately mountable. The one-time disk token appears
in `authorizedUsers[].token` and cannot be retrieved again. (This
disk token is scoped to the new disk and is separate from the API
key you used to make this request.)

To provide your own users instead, pass the deprecated `authMethods`
field or call AddDiskUser after creation.




## OpenAPI

````yaml POST /api/disks
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
    post:
      tags:
        - Disks
      summary: Create a new disk
      description: |
        Creates a new disk with the specified configuration. A default disk
        token user is automatically generated and returned in the response,
        so the disk is immediately mountable. The one-time disk token appears
        in `authorizedUsers[].token` and cannot be retrieved again. (This
        disk token is scoped to the new disk and is separate from the API
        key you used to make this request.)

        To provide your own users instead, pass the deprecated `authMethods`
        field or call AddDiskUser after creation.
      operationId: createDisk
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateDiskRequest'
      responses:
        '200':
          description: >-
            Disk with this name already exists and configuration matches
            (idempotent)
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_CreateDisk'
        '201':
          description: Disk created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_CreateDisk'
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '403':
          description: Access denied (e.g., preview region not enabled)
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '409':
          description: Disk with this name already exists but configuration differs
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '500':
          $ref: '#/components/responses/InternalError'
components:
  schemas:
    CreateDiskRequest:
      type: object
      required:
        - name
      properties:
        name:
          type: string
          description: Disk name (alphanumeric, dashes, underscores)
          minLength: 1
          maxLength: 100
          pattern: ^[a-zA-Z0-9_-]+$
          example: my-data-disk
        mounts:
          type: array
          description: Storage mount to attach. Omit for archil-managed storage.
          minItems: 0
          maxItems: 1
          items:
            $ref: '#/components/schemas/MountConfig'
        authMethods:
          type: array
          deprecated: true
          description: >
            Deprecated. Use AddDiskUser after creation instead. When provided,
            suppresses the default auto-generated token user.
          items:
            $ref: '#/components/schemas/DiskUser'
    ApiResponse_CreateDisk:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          type: object
          properties:
            diskId:
              type: string
              example: dsk-0123456789abcdef
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
    MountConfig:
      oneOf:
        - $ref: '#/components/schemas/S3Mount'
        - $ref: '#/components/schemas/GCSMount'
        - $ref: '#/components/schemas/R2Mount'
        - $ref: '#/components/schemas/S3CompatibleMount'
        - $ref: '#/components/schemas/AzureBlobMount'
      discriminator:
        propertyName: type
        mapping:
          s3: '#/components/schemas/S3Mount'
          gcs: '#/components/schemas/GCSMount'
          r2: '#/components/schemas/R2Mount'
          s3-compatible: '#/components/schemas/S3CompatibleMount'
          azure-blob: '#/components/schemas/AzureBlobMount'
    DiskUser:
      oneOf:
        - $ref: '#/components/schemas/TokenUser'
        - $ref: '#/components/schemas/AwsStsUser'
      discriminator:
        propertyName: type
        mapping:
          token: '#/components/schemas/TokenUser'
          awssts: '#/components/schemas/AwsStsUser'
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
    S3Mount:
      type: object
      title: S3
      description: Mount configuration for Amazon S3 buckets
      required:
        - type
        - bucketName
      properties:
        type:
          type: string
          enum:
            - s3
          description: Mount type identifier
        bucketName:
          type: string
          description: S3 bucket name
          example: my-bucket
        accessKeyId:
          type: string
          description: AWS access key ID (optional for public buckets or IAM role auth)
        secretAccessKey:
          type: string
          description: AWS secret access key
          format: password
          writeOnly: true
        sessionToken:
          type: string
          description: Session token for temporary credentials
          writeOnly: true
        bucketPrefix:
          type: string
          description: Prefix within the bucket
          example: data/
    GCSMount:
      type: object
      title: Google Cloud Storage
      description: Mount configuration for Google Cloud Storage buckets
      required:
        - type
        - bucketName
        - accessKeyId
        - secretAccessKey
      properties:
        type:
          type: string
          enum:
            - gcs
          description: Mount type identifier
        bucketName:
          type: string
          description: GCS bucket name
          example: my-gcs-bucket
        accessKeyId:
          type: string
          description: HMAC access key ID
          writeOnly: true
        secretAccessKey:
          type: string
          description: HMAC secret access key
          format: password
          writeOnly: true
        bucketPrefix:
          type: string
          description: Prefix within the bucket (optional)
          example: data/
    R2Mount:
      type: object
      title: Cloudflare R2
      description: Mount configuration for Cloudflare R2 buckets
      required:
        - type
        - bucketName
        - bucketEndpoint
        - accessKeyId
        - secretAccessKey
      properties:
        type:
          type: string
          enum:
            - r2
          description: Mount type identifier
        bucketName:
          type: string
          description: R2 bucket name
          example: my-r2-bucket
        bucketEndpoint:
          type: string
          description: R2 endpoint URL
          example: https://accountid.r2.cloudflarestorage.com
        accessKeyId:
          type: string
          description: R2 access key ID
          writeOnly: true
        secretAccessKey:
          type: string
          description: R2 secret access key
          format: password
          writeOnly: true
        bucketPrefix:
          type: string
          description: Prefix within the bucket (optional)
          example: data/
    S3CompatibleMount:
      type: object
      title: S3-Compatible
      description: >-
        Mount configuration for S3-compatible storage (MinIO, Backblaze B2,
        etc.)
      required:
        - type
        - bucketName
        - bucketEndpoint
        - accessKeyId
        - secretAccessKey
      properties:
        type:
          type: string
          enum:
            - s3-compatible
          description: Mount type identifier
        bucketName:
          type: string
          description: Bucket name
          example: my-bucket
        bucketEndpoint:
          type: string
          description: S3-compatible endpoint URL
          example: https://s3.us-west-001.backblazeb2.com
        accessKeyId:
          type: string
          description: Access key ID
          writeOnly: true
        secretAccessKey:
          type: string
          description: Secret access key
          format: password
          writeOnly: true
        bucketPrefix:
          type: string
          description: Prefix within the bucket (optional)
          example: data/
    AzureBlobMount:
      type: object
      title: Azure Blob Storage
      description: Mount configuration for Azure Blob Storage containers
      required:
        - type
        - containerName
        - tenantId
        - clientId
        - clientSecret
      properties:
        type:
          type: string
          enum:
            - azure-blob
          description: Mount type identifier
        containerName:
          type: string
          description: Azure blob container name
          example: my-container
        endpoint:
          type: string
          description: Azure blob endpoint URL (optional if storageAccountName provided)
          example: https://myaccount.blob.core.windows.net
        storageAccountName:
          type: string
          description: Azure storage account name (used to derive endpoint if not provided)
          example: myaccount
        tenantId:
          type: string
          description: Azure AD tenant ID
        clientId:
          type: string
          description: Azure AD application client ID
        clientSecret:
          type: string
          description: Azure AD application client secret
          format: password
          writeOnly: true
        bucketPrefix:
          type: string
          description: Prefix within the container
          example: data/
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
