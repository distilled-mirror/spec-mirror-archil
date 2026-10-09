> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Build Image

> Builds an OCI image for use by sandboxes. Poll [Get Image](/api-reference/images/get-image)
until it is `ready`, then pass its `image_id` when creating a sandbox.

Repeated requests join an active build. Completed tags rebuild; completed digest references
are reused. During a rebuild, sandboxes can use the last successful digest. Submit the request
again to retry a failed or unresponsive build.




## OpenAPI

````yaml POST /api/images
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
  /api/images:
    post:
      tags:
        - Sandboxes
      summary: Build a sandbox image
      description: >
        Builds an OCI image for use by sandboxes. Poll [Get
        Image](/api-reference/images/get-image)

        until it is `ready`, then pass its `image_id` when creating a sandbox.


        Repeated requests join an active build. Completed tags rebuild;
        completed digest references

        are reused. During a rebuild, sandboxes can use the last successful
        digest. Submit the request

        again to retry a failed or unresponsive build.
      operationId: createImage
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateImageRequest'
            example:
              source: ghcr.io/acme/agent-runtime:v2
              registry_auth:
                username: YOUR_REGISTRY_USERNAME
                password: YOUR_REGISTRY_TOKEN
      responses:
        '202':
          description: The image build is running or finished
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_Image'
              example:
                success: true
                data:
                  image_id: >-
                    0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
                  source: ghcr.io/acme/agent-runtime:v2
                  private: true
                  status: building
                  created_at: '2026-10-06T12:00:00Z'
                  updated_at: '2026-10-06T12:00:00Z'
        '400':
          description: >-
            Invalid request body, image reference, or registry credentials
            format
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                invalidBody:
                  summary: Invalid JSON request body
                  value:
                    success: false
                    error: Invalid request body
                    code: bad_request
                incompleteRegistryAuth:
                  summary: Missing registry username or password
                  value:
                    success: false
                    error: registry_auth requires a username and password
                    code: bad_request
        '401':
          $ref: '#/components/responses/PlainTextUnauthorized'
        '500':
          $ref: '#/components/responses/SandboxInternalError'
        '503':
          description: >-
            No build capacity is available or sandbox support is disabled in
            this region
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                noCapacity:
                  summary: No build capacity available
                  value:
                    success: false
                    error: No sandbox capacity is available; retry
                    code: no_capacity
                notEnabled:
                  summary: Sandbox support is disabled
                  value:
                    success: false
                    error: Sandboxes are not enabled on this controlplane
                    code: not_enabled
        '504':
          $ref: '#/components/responses/SandboxTimeout'
components:
  schemas:
    CreateImageRequest:
      type: object
      required:
        - source
      properties:
        source:
          type: string
          description: Linux OCI image reference. Docker shorthand and tags are accepted.
        registry_auth:
          $ref: '#/components/schemas/RegistryAuth'
    ApiResponse_Image:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/Image'
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
    RegistryAuth:
      type: object
      description: >-
        Registry credentials required for private images. Supply them on every
        build request; Archil does not store them. Omit for public images.
      required:
        - username
        - password
      properties:
        username:
          type: string
          minLength: 1
          description: Registry username.
        password:
          type: string
          minLength: 1
          description: Password or access token with pull access to the image.
    Image:
      type: object
      required:
        - image_id
        - source
        - private
        - status
        - created_at
        - updated_at
      properties:
        image_id:
          type: string
          pattern: ^[0-9a-f]{64}$
          description: >-
            Stable ID for the normalized source and its public or private scope.
            Private image IDs are scoped to your account. Adding or removing
            registry credentials changes the scope and the ID; rebuilding in the
            same scope keeps the ID.
        source:
          type: string
          description: Normalized OCI reference the image is pulled from.
        private:
          type: boolean
          description: Whether the image is pulled with registry credentials.
        status:
          $ref: '#/components/schemas/ImageState'
        digest:
          type: string
          pattern: ^sha256:[0-9a-f]{64}$
          description: >-
            The most recent successful build. Sandboxes created from this
            `image_id` boot it, and keep it after the image is rebuilt.
        canonical_source:
          type: string
          description: The pulled manifest, as `repository@sha256:...`.
        failure_reason:
          type: string
          enum:
            - invalid_source
            - interrupted
          description: >-
            Set when `failed`. `invalid_source`: the registry reported the image
            missing or denied access, so requesting it again will not help.
            `interrupted`: the build did not finish; request the image again.
        created_at:
          type: string
          format: date-time
        updated_at:
          type: string
          format: date-time
    ImageState:
      type: string
      enum:
        - building
        - ready
        - failed
  responses:
    PlainTextUnauthorized:
      description: Missing or invalid authentication credentials
      content:
        text/plain:
          schema:
            type: string
          examples:
            missingAuthorization:
              summary: Missing Authorization header
              value: Must provide an Authorization header
            invalidCredentials:
              summary: Invalid credentials
              value: Unauthorized
    SandboxInternalError:
      description: Internal server error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
          example:
            success: false
            error: Internal error processing sandbox request
            code: internal_server_error
    SandboxTimeout:
      description: Request timed out; retry
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
          example:
            success: false
            error: Sandbox request timed out; retry
            code: timeout
  securitySchemes:
    ApiKeyAuth:
      type: apiKey
      in: header
      name: Authorization
      description: API key

````

This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
