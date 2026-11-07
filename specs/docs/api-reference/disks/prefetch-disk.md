> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Prefetch File Ranges

> Warms byte ranges of files on a disk into Archil's storage cache, so
later reads from any client, serverless exec, or sandbox are served
without going to the backing object storage.

Each range names one file and an optional `offset` and `length`; a
file may appear in several ranges. Paths are relative to the disk
root. Ranges must name regular files: a directory is found but warms
nothing, so list a directory's files to warm its contents.

The response reports one result per range, in request order, and may
return before the data is fully fetched. A path that does not exist
reports `found: false`. A failed range does not fail the others;
retry the ranges whose `error` is `retry`. If the disk is not mounted
anywhere, prefetching loads it.

Warmed data becomes active data and is metered like data your clients
read.




## OpenAPI

````yaml POST /api/disks/{id}/prefetch
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
  /api/disks/{id}/prefetch:
    post:
      tags:
        - Disks
      summary: Prefetch file ranges
      description: |
        Warms byte ranges of files on a disk into Archil's storage cache, so
        later reads from any client, serverless exec, or sandbox are served
        without going to the backing object storage.

        Each range names one file and an optional `offset` and `length`; a
        file may appear in several ranges. Paths are relative to the disk
        root. Ranges must name regular files: a directory is found but warms
        nothing, so list a directory's files to warm its contents.

        The response reports one result per range, in request order, and may
        return before the data is fully fetched. A path that does not exist
        reports `found: false`. A failed range does not fail the others;
        retry the ranges whose `error` is `retry`. If the disk is not mounted
        anywhere, prefetching loads it.

        Warmed data becomes active data and is metered like data your clients
        read.
      operationId: prefetchDisk
      parameters:
        - $ref: '#/components/parameters/DiskId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PrefetchRequest'
            example:
              ranges:
                - path: models/weights.bin
                - path: data/2026/events.parquet
                  offset: 0
                  length: 1048576
      responses:
        '200':
          description: Per-range prefetch results
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_Prefetch'
              example:
                success: true
                data:
                  results:
                    - found: true
                      bytesRequested: 4294967296
                      bytesWarmed: 4294967296
                    - found: true
                      bytesRequested: 1048576
                      bytesWarmed: 0
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'
        '500':
          $ref: '#/components/responses/InternalError'
        '503':
          description: Temporarily unavailable; safe to retry.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
      security:
        - ApiKeyAuth:
            - disk:prefetch:{id}
        - SessionTokenAuth: []
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
    PrefetchRequest:
      type: object
      additionalProperties: false
      required:
        - ranges
      properties:
        ranges:
          type: array
          minItems: 1
          description: Byte ranges to warm. A request may name at most 1000.
          items:
            $ref: '#/components/schemas/PrefetchRange'
    ApiResponse_Prefetch:
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
            - results
          properties:
            results:
              type: array
              description: One result per requested range, in request order.
              items:
                $ref: '#/components/schemas/PrefetchRangeResult'
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
    PrefetchRange:
      type: object
      additionalProperties: false
      required:
        - path
      properties:
        path:
          type: string
          minLength: 1
          description: Path of a regular file, relative to the disk root.
          example: models/weights.bin
        offset:
          type: integer
          format: int64
          minimum: 0
          description: First byte to warm. Defaults to 0.
        length:
          type: integer
          format: int64
          minimum: 0
          description: Bytes to warm from `offset`. Defaults to the end of the file.
    PrefetchRangeResult:
      type: object
      required:
        - found
        - bytesRequested
        - bytesWarmed
      properties:
        found:
          type: boolean
          description: Whether the path exists.
        bytesRequested:
          type: integer
          format: int64
          description: >
            Bytes of the range being warmed, including bytes that were already
            cached.
        bytesWarmed:
          type: integer
          format: int64
          description: >
            Of `bytesRequested`, the bytes fetched (or still being fetched) from
            object storage. The rest were already cached.
        error:
          type: string
          enum:
            - retry
            - failed
          description: >
            Set when this range was not warmed: `retry` for a transient failure
            worth retrying, `failed` otherwise. Other ranges in the request are
            unaffected.
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
      type: http
      scheme: bearer
      description: Archil API key, including its `key-` prefix

````

This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
