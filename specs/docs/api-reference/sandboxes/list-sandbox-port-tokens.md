> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# List Sandbox Port Tokens

> Lists unexpired private port token metadata without returning raw tokens.



## OpenAPI

````yaml GET /api/sandboxes/{sid}/port-tokens
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
  /api/sandboxes/{sid}/port-tokens:
    get:
      tags:
        - Sandboxes
      summary: List sandbox port tokens
      description: >-
        Lists unexpired private port token metadata without returning raw
        tokens.
      operationId: listSandboxPortTokens
      parameters:
        - $ref: '#/components/parameters/SandboxId'
        - $ref: '#/components/parameters/Limit'
        - $ref: '#/components/parameters/Cursor'
      responses:
        '200':
          description: Sandbox port tokens
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_SandboxPortTokenList'
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
    SandboxId:
      name: sid
      in: path
      required: true
      description: Sandbox UUID
      schema:
        type: string
        format: uuid
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
    ApiResponse_SandboxPortTokenList:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/SandboxPortTokenList'
        nextCursor:
          type: string
          description: Set when more port tokens remain.
    SandboxPortTokenList:
      type: object
      required:
        - tokens
      properties:
        tokens:
          type: array
          items:
            $ref: '#/components/schemas/SandboxPortToken'
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
    SandboxPortToken:
      type: object
      required:
        - id
        - port
        - created_at
      properties:
        id:
          type: string
          pattern: ^[0-9a-f]{64}$
          description: Stable token identifier used for get and revoke operations.
        port:
          type: integer
          minimum: 1
          maximum: 65535
        created_at:
          type: string
          format: date-time
        expires_at:
          type: string
          format: date-time
          description: When the token expires. Absent for a non-expiring token.
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
