> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Get Sandbox Usage

> Returns the vCPU-hours and GiB-hours a sandbox has been billed for since it was created, and their cost at the account's rates before included allowances and prepaid credit. The figures update about once a minute while the sandbox runs; paused and stopped time is free. Usage stays readable for 90 days after the sandbox is deleted.




## OpenAPI

````yaml GET /api/sandboxes/{sid}/usage
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
  /api/sandboxes/{sid}/usage:
    get:
      tags:
        - Sandboxes
      summary: Get a sandbox's usage and cost
      description: >
        Returns the vCPU-hours and GiB-hours a sandbox has been billed for since
        it was created, and their cost at the account's rates before included
        allowances and prepaid credit. The figures update about once a minute
        while the sandbox runs; paused and stopped time is free. Usage stays
        readable for 90 days after the sandbox is deleted.
      operationId: getSandboxUsage
      parameters:
        - $ref: '#/components/parameters/SandboxId'
      responses:
        '200':
          description: The sandbox's usage and cost
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_SandboxUsage'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          description: >-
            Sandbox not found (`not_found`), or usage billing is unavailable in
            this deployment (`billing_unavailable`)
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '500':
          $ref: '#/components/responses/InternalError'
        '503':
          description: >-
            Pricing is temporarily unavailable (`pricing_unavailable`); safe to
            retry.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
      security:
        - ApiKeyAuth:
            - sandbox:manage:get:{sid}
        - SessionTokenAuth: []
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
    ApiResponse_SandboxUsage:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/SandboxUsage'
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
    SandboxUsage:
      type: object
      required:
        - sandbox_id
        - as_of
        - vcpu_hours
        - memory_gib_hours
        - rates
        - cost
      properties:
        sandbox_id:
          type: string
          format: uuid
        as_of:
          type: string
          format: date-time
          description: When the usage was read.
        vcpu_hours:
          type: number
          format: double
          description: Allocated vCPUs times hours running.
          example: 3
        memory_gib_hours:
          type: number
          format: double
          description: Allocated memory in GiB times hours running.
          example: 6
        rates:
          type: object
          required:
            - vcpu_hour
            - memory_gib_hour
          properties:
            vcpu_hour:
              type: number
              format: double
              description: >-
                Dollars per vCPU-hour at the account's rates; zero on a plan
                that does not price sandbox usage.
              example: 0.055
            memory_gib_hour:
              type: number
              format: double
              description: >-
                Dollars per GiB-hour at the account's rates; zero on a plan that
                does not price sandbox usage.
              example: 0.02
        cost:
          type: number
          format: double
          description: Dollars, before included allowances and prepaid credit.
          example: 0.285
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
      type: http
      scheme: bearer
      description: Archil API key, including its `key-` prefix

````

This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
