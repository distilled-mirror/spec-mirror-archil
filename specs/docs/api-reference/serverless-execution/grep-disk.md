> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Search Files (grep)

> Searches files under a directory on the disk for lines matching a
regular expression. Listing and matching are fanned out across
ephemeral exec containers, so the request finishes within the user's
time budget regardless of the size of the directory.

The user controls cost and latency with three knobs:

- `maxDurationSeconds` is the wall-clock deadline (capped at 30s).
- `concurrency` is the maximum number of parallel grep workers.
  Higher concurrency finishes larger datasets within the deadline,
  at proportionally more compute.
- `maxResults` causes the search to short-circuit after the
  aggregator has collected this many matches.

With `recursive: false` only files directly under `directory` are
searched. With `recursive: true` subdirectories are walked
breadth-first and grep workers are dispatched as soon as each level
finishes listing — listing and matching overlap.

The matches returned when stopping early on `maxResults` are a sample
of whichever workers reported first, not the lexicographically first
N. The response surfaces `stoppedReason` so callers can distinguish
completion from early termination.




## OpenAPI

````yaml POST /api/disks/{id}/grep
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
  /api/disks/{id}/grep:
    post:
      tags:
        - Serverless Execution
      summary: Parallel grep over a directory on a disk
      description: |
        Searches files under a directory on the disk for lines matching a
        regular expression. Listing and matching are fanned out across
        ephemeral exec containers, so the request finishes within the user's
        time budget regardless of the size of the directory.

        The user controls cost and latency with three knobs:

        - `maxDurationSeconds` is the wall-clock deadline (capped at 30s).
        - `concurrency` is the maximum number of parallel grep workers.
          Higher concurrency finishes larger datasets within the deadline,
          at proportionally more compute.
        - `maxResults` causes the search to short-circuit after the
          aggregator has collected this many matches.

        With `recursive: false` only files directly under `directory` are
        searched. With `recursive: true` subdirectories are walked
        breadth-first and grep workers are dispatched as soon as each level
        finishes listing — listing and matching overlap.

        The matches returned when stopping early on `maxResults` are a sample
        of whichever workers reported first, not the lexicographically first
        N. The response surfaces `stoppedReason` so callers can distinguish
        completion from early termination.
      operationId: grepDisk
      parameters:
        - $ref: '#/components/parameters/DiskId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/GrepDiskRequest'
      responses:
        '200':
          description: Grep completed (possibly stopped early)
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_GrepDisk'
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
    GrepDiskRequest:
      type: object
      required:
        - directory
        - pattern
      properties:
        directory:
          type: string
          description: |
            Directory on the disk to search, relative to the disk root.
            An empty string or "/" means the disk root.
          example: data/logs
        pattern:
          type: string
          description: Extended regular expression (passed to `grep -E`)
          example: ERROR|FATAL
        recursive:
          type: boolean
          description: When true, walks subdirectories breadth-first.
          default: false
        maxDurationSeconds:
          type: integer
          minimum: 1
          maximum: 30
          default: 30
          description: |
            Wall-clock deadline for the entire request. Capped at 30
            seconds because the runtime exec container itself is bounded
            at ~30s; longer requests would have their workers killed
            mid-scan.
        concurrency:
          type: integer
          minimum: 1
          maximum: 100
          default: 50
          description: |
            Maximum number of parallel grep workers. Higher values finish
            larger datasets within the deadline but consume proportionally
            more runtime capacity.
        maxResults:
          type: integer
          minimum: 1
          maximum: 10000
          default: 1000
          description: |
            Stop scanning once the aggregator has this many matches.
            Returned matches are a sample of whichever workers reported
            first, not the lexicographically first N.
    ApiResponse_GrepDisk:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/GrepDiskResult'
    GrepDiskResult:
      type: object
      required:
        - matches
        - stoppedReason
        - filesScanned
        - containersDispatched
        - computeSecondsUsed
        - durationMs
        - listingMs
        - grepMs
      properties:
        matches:
          type: array
          items:
            $ref: '#/components/schemas/GrepMatch'
        stoppedReason:
          $ref: '#/components/schemas/GrepStoppedReason'
        filesScanned:
          type: integer
          description: Files actually fed to a grep container.
        containersDispatched:
          type: integer
          description: Number of grep containers started for this request.
        computeSecondsUsed:
          type: number
          format: double
          description: |
            Sum of per-container execution time in seconds, measured by the
            runtime. Approximates billable container-seconds.
        durationMs:
          type: integer
          description: End-to-end wall clock measured by the server.
        listingMs:
          type: integer
          description: |
            Wall-clock time spent enumerating files via listObjects, from
            the request's start to the moment listing fully drained (or
            was canceled). Listing and matching overlap, so listingMs +
            grepMs typically exceeds durationMs.
        grepMs:
          type: integer
          description: |
            Wall-clock time spent matching, from the first grep container
            being dispatched to the last container reporting results. 0 if
            no batches ran.
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
    GrepMatch:
      type: object
      required:
        - file
        - line
        - text
      properties:
        file:
          type: string
          description: Path to the file (relative to the disk root).
          example: data/logs/2026-05-03.log
        line:
          type: integer
          description: 1-based line number where the match occurred.
          example: 142
        text:
          type: string
          description: The matching line.
    GrepStoppedReason:
      type: string
      enum:
        - completed
        - incomplete
        - max_results
        - deadline
        - list_failed
      description: |
        Why the search stopped.
        - `completed`: every file under the directory was scanned successfully.
        - `incomplete`: pipeline ran to its natural end but one or more
          batches errored (invalid regex, unreadable file, runtime issue).
          Results may be partial or wrong; do not rely on completeness.
        - `max_results`: hit `maxResults` before scanning everything.
        - `deadline`: hit `maxDurationSeconds`.
        - `list_failed`: directory listing failed; partial results
          may be present.
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
