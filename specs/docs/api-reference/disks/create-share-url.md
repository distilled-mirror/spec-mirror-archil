> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Create Share URL

> Creates a signed, time-limited URL for downloading a file without
authentication. The URL expires after 24 hours by default; pass
`expiresIn` to set a custom lifetime up to 7 days.

The returned URL points to a public endpoint (`GET /api/shared/{token}`)
that verifies the signature, checks expiry, and streams the file with a
sensible `Content-Type` and `Content-Disposition`. Range requests are
supported. No credentials are needed to open the URL.




## OpenAPI

````yaml POST /api/disks/{id}/share
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
  /api/disks/{id}/share:
    post:
      tags:
        - Disks
      summary: Create a share URL for a file
      description: |
        Creates a signed, time-limited URL for downloading a file without
        authentication. The URL expires after 24 hours by default; pass
        `expiresIn` to set a custom lifetime up to 7 days.

        The returned URL points to a public endpoint (`GET /api/shared/{token}`)
        that verifies the signature, checks expiry, and streams the file with a
        sensible `Content-Type` and `Content-Disposition`. Range requests are
        supported. No credentials are needed to open the URL.
      operationId: createShareUrl
      parameters:
        - $ref: '#/components/parameters/DiskId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateShareUrlRequest'
      responses:
        '200':
          description: Share URL created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_ShareUrl'
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'
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
    CreateShareUrlRequest:
      type: object
      required:
        - key
      properties:
        key:
          type: string
          minLength: 1
          description: File key (path) within the disk.
          example: reports/2026-08/summary.csv
        expiresIn:
          type: integer
          minimum: 1
          maximum: 604800
          description: |
            URL lifetime in seconds. Any integer from 1 to 604800 (7 days).
            Defaults to 86400 (24 hours).
          example: 3600
    ApiResponse_ShareUrl:
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
          required:
            - url
            - expiresIn
          properties:
            url:
              type: string
              format: uri
              description: The signed, public download URL.
            expiresIn:
              type: integer
              description: URL lifetime in seconds.
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
