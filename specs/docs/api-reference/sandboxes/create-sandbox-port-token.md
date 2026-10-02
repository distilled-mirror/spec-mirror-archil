> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Create Sandbox Port Token

> Creates token-authorized HTTP access to one port. Send the raw token returned
by this request in the `X-Archil-Token` request header at the stable
`<port>-<sandbox-route-id>.<zone>` hostname. Public ports accept connections
without a token; otherwise a valid token for that sandbox and port is required.
The token is not returned again.




## OpenAPI

````yaml POST /api/sandboxes/{sid}/port-tokens
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
    post:
      tags:
        - Sandboxes
      summary: Create a private sandbox port token
      description: >
        Creates token-authorized HTTP access to one port. Send the raw token
        returned

        by this request in the `X-Archil-Token` request header at the stable

        `<port>-<sandbox-route-id>.<zone>` hostname. Public ports accept
        connections

        without a token; otherwise a valid token for that sandbox and port is
        required.

        The token is not returned again.
      operationId: createSandboxPortToken
      parameters:
        - $ref: '#/components/parameters/SandboxId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateSandboxPortTokenRequest'
      responses:
        '201':
          description: Port token was created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_CreatedSandboxPortToken'
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
  schemas:
    CreateSandboxPortTokenRequest:
      type: object
      required:
        - port
      properties:
        port:
          type: integer
          minimum: 1
          maximum: 65535
        ttl:
          type: string
          description: |
            Token lifetime as a Go duration string (for example, "4m", "1h",
            or "24h"). Maximum 365 days ("8760h"). Omit for no expiration.
          example: 4m
    ApiResponse_CreatedSandboxPortToken:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/CreatedSandboxPortToken'
    CreatedSandboxPortToken:
      allOf:
        - $ref: '#/components/schemas/SandboxPortToken'
        - type: object
          required:
            - hostname
            - token
          properties:
            hostname:
              type: string
              description: Stable sandbox port hostname, shared with public access.
            token:
              type: string
              description: Raw token returned only when the token is created.
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

This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
