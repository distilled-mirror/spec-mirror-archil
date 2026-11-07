> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Search Files (grep)

> Searches files under a directory on the disk for lines matching a
regular expression. Listing and matching are fanned out across
ephemeral exec microVMs, so the request finishes within the user's
time budget regardless of the size of the directory.

The user controls search latency and scope with three knobs:

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

Matching runs on ripgrep. Narrow the files searched with `globs`,
`types`, and `typesNot`; change matching with `ignoreCase` and
`multiline`; and ask for surrounding lines with `contextBefore` and
`contextAfter`, which adds `groups` to the result.

The matches returned when stopping early on `maxResults`, or on the
32 MiB limit on match and context text, are a sample of whichever
workers reported first, not the lexicographically first N. The
response surfaces `stoppedReason` so callers can distinguish
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
  /api/disks/{id}/grep:
    post:
      tags:
        - Serverless Execution
      summary: Parallel grep over a directory on a disk
      description: |
        Searches files under a directory on the disk for lines matching a
        regular expression. Listing and matching are fanned out across
        ephemeral exec microVMs, so the request finishes within the user's
        time budget regardless of the size of the directory.

        The user controls search latency and scope with three knobs:

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

        Matching runs on ripgrep. Narrow the files searched with `globs`,
        `types`, and `typesNot`; change matching with `ignoreCase` and
        `multiline`; and ask for surrounding lines with `contextBefore` and
        `contextAfter`, which adds `groups` to the result.

        The matches returned when stopping early on `maxResults`, or on the
        32 MiB limit on match and context text, are a sample of whichever
        workers reported first, not the lexicographically first N. The
        response surfaces `stoppedReason` so callers can distinguish
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
          description: >
            Regular expression in [ripgrep
            syntax](https://docs.rs/regex/latest/regex/#syntax).

            A newline separates alternative patterns, except with `multiline`,

            where it matches a line break.
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
            seconds because the runtime exec microVM itself is bounded
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
        ignoreCase:
          type: boolean
          description: Match the pattern case-insensitively (`rg -i`). Defaults to false.
        multiline:
          type: boolean
          description: |
            Let a match span lines (`rg -U`): `\n` or a newline character in
            the pattern matches a line break, and `.` matches one only after
            `(?s)`. Files with Windows line endings need `\r?\n` instead of
            `\n`. Matches on adjacent lines come back as one match covering
            all of them, with `endLine` set, and count once toward
            `maxResults`. A file with at least `maxResults` matching lines
            stops the request with `max_results`, even when merging leaves
            fewer matches. Defaults to false.
        globs:
          type: array
          maxItems: 32
          items:
            type: string
            minLength: 1
            maxLength: 1024
          description: |
            Path globs with ripgrep's `-g` rules, matched against paths
            relative to `directory`. A leading `!` excludes. A glob without a
            `/` matches a file or directory name at any depth; a glob with a
            `/` matches the whole relative path, and a leading `/` anchors it
            to `directory`. A trailing `/` matches only directories, so use
            `dir/**` to search the files under `dir`. When several globs
            match a path, the last one wins. A directory excluded this way is
            skipped with everything under it. If any non-`!` glob is given,
            files that match no glob are skipped. A file a glob matches is
            searched or skipped regardless of `types` and `typesNot`.
          example:
            - '*.log'
            - '!archive/**'
        types:
          type: array
          maxItems: 32
          items:
            type: string
            minLength: 1
            maxLength: 32
          description: |
            Only search files of these types, using ripgrep's default type
            names (for example `py`, `ts`, `go`, `md`).
          example:
            - py
        typesNot:
          type: array
          maxItems: 32
          items:
            type: string
            minLength: 1
            maxLength: 32
          description: |
            Skip files of these types, using ripgrep's default type names.
            Takes precedence over `types`.
          example:
            - json
        contextBefore:
          type: integer
          minimum: 0
          maximum: 10
          default: 0
          description: |
            Lines of context to return before each match (`rg -B`). When
            either context field is set, the result includes `groups`.
        contextAfter:
          type: integer
          minimum: 0
          maximum: 10
          default: 0
          description: |
            Lines of context to return after each match (`rg -A`). When
            either context field is set, the result includes `groups`.
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
        groups:
          type: array
          description: |
            Matches with their context lines, grouped the way grep prints
            them. Present only when `contextBefore` or `contextAfter` is set.
            Holds the same matches as `matches`. When the search stops at
            `maxResults`, the last group ends up to `contextAfter` lines after
            its last kept match, and any matches in those lines are shown as
            context. Each line of a group counts toward the 32 MiB response
            limit, so large context and `maxResults` multiply it.
          items:
            $ref: '#/components/schemas/GrepGroup'
        stoppedReason:
          $ref: '#/components/schemas/GrepStoppedReason'
        filesScanned:
          type: integer
          description: Files actually fed to a grep microVM.
        containersDispatched:
          type: integer
          description: Number of grep microVMs started for this request.
        computeSecondsUsed:
          type: number
          format: double
          description: |
            Sum of per-microVM execution time in seconds, measured by the
            runtime. Diagnostic only; each search request is billed as one
            serverless tool call regardless of this value.
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
            Wall-clock time spent matching, from the first grep microVM
            being dispatched to the last microVM reporting results. 0 if
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
        code:
          type: string
          description: Stable machine-readable error code.
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
          description: |
            1-based line number where the match occurred. With `multiline`,
            the match's first line.
          example: 142
        endLine:
          type: integer
          description: With `multiline`, the 1-based number of the match's last line.
        text:
          type: string
          description: |
            The matching line, truncated at 512 KiB, or shorter with context.
            With `multiline`, every line the match covers, joined by `\n`,
            each truncated separately.
    GrepGroup:
      type: object
      required:
        - file
        - lines
      description: |
        A run of consecutive lines from one file, as grep prints it with
        context: each match with its surrounding lines, with nearby matches
        merged into one group so every line appears once.
      properties:
        file:
          type: string
          description: Path to the file (relative to the disk root).
        lines:
          type: array
          items:
            $ref: '#/components/schemas/GrepGroupLine'
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
          batches errored (invalid regex, unreadable file, runtime issue) or
          exceeded the 4 MiB output limit.
          Results may be partial or wrong; do not rely on completeness.
        - `max_results`: hit `maxResults`, or the response size limit
          (32 MiB of match and context text), before scanning everything.
        - `deadline`: hit `maxDurationSeconds`.
        - `list_failed`: directory listing failed; partial results
          may be present.
    GrepGroupLine:
      type: object
      required:
        - line
        - text
        - match
      properties:
        line:
          type: integer
          description: 1-based line number.
        text:
          type: string
          description: The line, truncated like `GrepMatch.text`.
        match:
          type: boolean
          description: True for a matching line, false for a context line.
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
