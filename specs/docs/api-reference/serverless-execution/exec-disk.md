> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Execute Command on Disk

> Launches a Firecracker microVM with the specified disk mounted, runs the
given command to completion, shuts down the microVM, and returns the
command result.




## OpenAPI

````yaml POST /api/disks/{id}/exec
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
  /api/disks/{id}/exec:
    post:
      tags:
        - Serverless Execution
      summary: Execute a command on a disk
      description: |
        Launches a Firecracker microVM with the specified disk mounted, runs the
        given command to completion, shuts down the microVM, and returns the
        command result.
      operationId: execDisk
      parameters:
        - $ref: '#/components/parameters/DiskId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ExecDiskRequest'
      responses:
        '200':
          description: Command completed
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_ExecDisk'
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '403':
          description: |
            The request is not allowed on the account's plan. `code` is one of:

            - `allowance_exhausted`: the plan has used its serverless tool call
              allowance for the month.
            - `tool_call_size_limit_exceeded`: the exec is larger than the plan
              allows. The Free plan allows only the default size (2 vCPU,
              4096 MiB).
            - `network_plan_required`: `network` was set on the Free plan.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '404':
          $ref: '#/components/responses/NotFound'
        '409':
          description: The `imageId` image has no finished build yet (`image_not_ready`).
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '500':
          $ref: '#/components/responses/InternalError'
        '504':
          description: Command timed out
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
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
    ExecDiskRequest:
      type: object
      required:
        - command
      properties:
        command:
          type: string
          description: Shell command to execute inside the microVM
          example: ls -la /mnt/archil
        baseImage:
          type: string
          description: >-
            Public Linux OCI image reference to run the command in, such as
            `python:3.13`. Docker Hub shorthand, tags, and digests are accepted.
            The first exec with an image waits while Archil prepares it; later
            execs reuse it. Defaults to Archil's exec image when neither this
            nor `imageId` is set. Mutually exclusive with `imageId`.
          example: python:3.13
        imageId:
          type: string
          pattern: ^[0-9a-f]{64}$
          description: >-
            An image built with [Build
            Image](/api-reference/images/build-image), for private images or to
            prepare a public image ahead of time. The exec runs the image's
            latest successful build. Mutually exclusive with `baseImage`.
        vcpuCount:
          type: integer
          minimum: 1
          maximum: 32
          description: >-
            vCPUs for the command's microVM. Defaults to 2. An exec larger than
            the default size (2 vCPU, 4096 MiB) bills as that many multiples of
            a call, by whichever of vCPU and memory is the larger multiple,
            rounded up. The Free plan allows only the default size.
          example: 4
        memSizeMib:
          type: integer
          minimum: 256
          maximum: 65536
          description: >-
            Memory in MiB for the command's microVM. Defaults to 4096. Billed as
            described for `vcpuCount`.
          example: 8192
        network:
          $ref: '#/components/schemas/ExecNetwork'
    ApiResponse_ExecDisk:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/ExecDiskResult'
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
    ExecNetwork:
      type: object
      description: |
        Network policy for the command's microVM, using the
        [sandbox firewall](/compute/sandboxes/firewall) egress rules. Egress is
        unrestricted when omitted. The Free plan cannot set one; a request
        with one is rejected with 403 `network_plan_required`.
      properties:
        egress:
          $ref: '#/components/schemas/SandboxEgressPolicy'
    ExecDiskResult:
      type: object
      required:
        - exitCode
        - stdout
        - stderr
        - timing
      properties:
        exitCode:
          type: integer
          description: Exit code of the command (0 = success)
          example: 0
        stdout:
          type: string
          description: Standard output from the command
        stderr:
          type: string
          description: Standard error from the command
        timing:
          $ref: '#/components/schemas/ExecTiming'
    SandboxEgressPolicy:
      type: object
      description: >-
        Deny targets take precedence when an address or domain matches both
        lists. When domain rules or drain selectors are present, TCP ports 80
        and 443 are restricted to HTTP/1.1 or HTTP/2. Plaintext HTTP authority
        is enforced on port 80, but requests matching a transformation rule are
        rejected unless the rule also forwards. Port 443 is TLS-terminated; both
        TLS SNI and HTTP authority are evaluated, and the HTTP authority selects
        the upstream, any request transformations, and any request forwarding.
        DNS to the sandbox's configured resolvers is allowed and UDP port 443 is
        denied.
      required:
        - default
      properties:
        default:
          $ref: '#/components/schemas/SandboxNetworkAction'
        allow:
          type: array
          items:
            oneOf:
              - type: string
              - $ref: '#/components/schemas/SandboxEgressRule'
          description: >-
            Allowed IPv4 addresses, CIDR ranges, exact domains, wildcard domains
            beginning with `*.`, or target objects with optional outbound HTTPS
            request transformations. A wildcard matches subdomains but not the
            apex domain, including when applying transformations.
        deny:
          type: array
          items:
            type: string
          description: >-
            Denied IPv4 addresses, CIDR ranges, exact domains, or wildcard
            domains beginning with `*.`. A wildcard matches subdomains but not
            the apex domain.
        drain_on_pause:
          type: array
          description: >-
            Optional hosts whose HTTP(S) requests should finish before pausing,
            regardless of URL path. Pausing gates new matching requests and
            waits up to ten minutes by default for active responses, including
            streams, before taking a snapshot. Errors stop counting, and expiry
            proceeds with the snapshot. This does not limit requests during
            normal operation. Selectors do not grant network access. Matching
            uses the original request host before transformations or forwarding.
            Only TCP ports 80 and 443 are supported. Connections may need to be
            retried after resume.
          items:
            type: string
            description: >-
              Lowercase hostname pattern without a scheme, port, or path.
              Asterisk matches zero or more characters; *.example.com matches
              subdomains, not example.com. Use * for every host.
            example: bedrock-runtime.*.amazonaws.com
    ExecTiming:
      type: object
      description: Server-measured timings for an exec request.
      required:
        - totalMs
        - queueMs
        - executeMs
      properties:
        totalMs:
          type: integer
          description: >
            End-to-end wall clock measured on the server, from request arrival
            to response.
          example: 2450
        queueMs:
          type: integer
          description: >
            Time spent queueing, scheduling, booting/claiming a VM, and mounting
            the filesystem before the command started running.
          example: 150
        executeMs:
          type: integer
          description: |
            Time the user's command itself ran, measured by the runtime.
          example: 2300
    SandboxNetworkAction:
      type: string
      enum:
        - allow
        - deny
    SandboxEgressRule:
      type: object
      additionalProperties: false
      required:
        - target
      properties:
        target:
          type: string
          description: >-
            Target allowed by this rule. Effective transformations and
            forwarding require an exact or wildcard lowercase domain.
        transform:
          $ref: '#/components/schemas/SandboxEgressTransform'
        forward_url:
          type: string
          format: uri
          description: >-
            Absolute public HTTPS URL that receives this rule's permitted HTTP
            and HTTPS requests instead of their original upstream. Requires a
            lowercase domain; IP addresses, credentials, queries, and fragments
            are rejected. Protected and private destination addresses are
            blocked. The original path and query are appended to this URL's path
            prefix. Archil overwrites the archil-forwarded-host,
            archil-forwarded-scheme, archil-forwarded-port,
            archil-forwarded-path, and archil-sandbox-id headers with request
            metadata. A transform on the same rule is applied before forwarding.
    SandboxEgressTransform:
      type: object
      description: >-
        Optional outbound HTTPS request transformations. An omitted or empty
        transform leaves the rule as an ordinary allow without request
        mutations.
      additionalProperties: false
      properties:
        headers:
          type: object
          additionalProperties:
            type: string
          description: >-
            Outbound HTTPS request headers to set, overwriting values supplied
            by the sandbox.
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
