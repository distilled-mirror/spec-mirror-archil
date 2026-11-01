> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Get Usage

> Returns what the account's meters recorded over a window, for the window as a whole and per interval, in each meter's own unit (such as vCPU-hours, GiB-hours, GiB-months or calls), and what that usage comes to at the account's rates, per meter and in total, before any grant and fixed fee: the cost of what was used, not what will be invoiced, which `/api/billing/summary` reports. The meters are the catalog's metered features, so a meter added there is reported from then on. A meter's rate is the price of the plan item its usage is billed from: the meter's own item, or the credit system whose rate card names it, at the plan's own prices and overrides; a meter the plan does not price has a zero rate, and a tiered item is priced at its first tier. Usage reaches billing once per reporter interval from every region, so an interval's figures can still grow for a few minutes after it ends, longer if a region's reporter is behind. The default window is the seven days (hourly: 24 hours) up to now; a window may be at most 31 days, and is widened to whole intervals. Money is in dollars, to the cent.




## OpenAPI

````yaml GET /api/billing/usage
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
  /api/billing/usage:
    get:
      tags:
        - Billing
      summary: Get metered usage and its cost over a window
      description: >
        Returns what the account's meters recorded over a window, for the window
        as a whole and per interval, in each meter's own unit (such as
        vCPU-hours, GiB-hours, GiB-months or calls), and what that usage comes
        to at the account's rates, per meter and in total, before any grant and
        fixed fee: the cost of what was used, not what will be invoiced, which
        `/api/billing/summary` reports. The meters are the catalog's metered
        features, so a meter added there is reported from then on. A meter's
        rate is the price of the plan item its usage is billed from: the meter's
        own item, or the credit system whose rate card names it, at the plan's
        own prices and overrides; a meter the plan does not price has a zero
        rate, and a tiered item is priced at its first tier. Usage reaches
        billing once per reporter interval from every region, so an interval's
        figures can still grow for a few minutes after it ends, longer if a
        region's reporter is behind. The default window is the seven days
        (hourly: 24 hours) up to now; a window may be at most 31 days, and is
        widened to whole intervals. Money is in dollars, to the cent.
      operationId: getBillingUsage
      parameters:
        - $ref: '#/components/parameters/UsageStart'
        - $ref: '#/components/parameters/UsageEnd'
        - $ref: '#/components/parameters/UsageFor'
        - $ref: '#/components/parameters/UsageGranularity'
        - $ref: '#/components/parameters/UsageTimezone'
      responses:
        '200':
          description: Usage and its cost over the window and per interval
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_BillingUsage'
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '502':
          $ref: '#/components/responses/InternalError'
      security:
        - ApiKeyAuth:
            - billing:usage:get
        - SessionTokenAuth: []
components:
  parameters:
    UsageStart:
      name: start
      in: query
      description: >
        Start of the window, inclusive: an RFC 3339 timestamp, or a YYYY-MM-DD
        date taken at midnight in `timezone`. Defaults to seven days (hourly: 24
        hours) before `end`. Not with `for`.
      schema:
        type: string
        example: '2026-10-13'
    UsageEnd:
      name: end
      in: query
      description: >
        End of the window, exclusive, in the same forms as `start`. Defaults to
        now. Not with `for`.
      schema:
        type: string
        example: '2026-10-20'
    UsageFor:
      name: for
      in: query
      description: >
        A named window relative to now in `timezone`, instead of `start` and
        `end`. Weeks start on Monday. A window may run past now; the intervals
        after it are left out.
      schema:
        type: string
        enum:
          - today
          - yesterday
          - this-week
          - last-week
          - this-month
          - last-month
    UsageGranularity:
      name: granularity
      in: query
      description: Width of each interval. Days are aligned to `timezone`; hours are UTC.
      schema:
        type: string
        enum:
          - 1d
          - 1h
        default: 1d
    UsageTimezone:
      name: timezone
      in: query
      description: IANA timezone that day intervals and dates are aligned to.
      schema:
        type: string
        default: UTC
        example: America/Los_Angeles
  schemas:
    ApiResponse_BillingUsage:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
        data:
          $ref: '#/components/schemas/BillingUsage'
    BillingUsage:
      type: object
      required:
        - granularity
        - timezone
        - currency
        - ratesPerUnit
        - window
        - intervals
      properties:
        granularity:
          type: string
          enum:
            - 1d
            - 1h
        timezone:
          type: string
        currency:
          type: string
        ratesPerUnit:
          type: object
          description: >-
            Cost of one unit of each meter, in dollars per unit as a decimal (a
            rate can be well below a cent), keyed by feature id; zero for a
            meter the plan does not price.
          additionalProperties:
            type: number
        window:
          $ref: '#/components/schemas/BillingUsageWindow'
        intervals:
          type: array
          description: >-
            Newest first, each the window's share of one interval. Intervals
            after now are left out.
          items:
            $ref: '#/components/schemas/BillingUsageInterval'
        billingPeriod:
          description: >-
            The current billing period, which `/api/billing/summary` sums;
            absent on a plan without one.
          allOf:
            - $ref: '#/components/schemas/BillingPeriod'
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
    BillingUsageWindow:
      type: object
      description: >-
        The window as a whole, the summary the intervals break down. Its
        quantities are the intervals' sums; its costs are priced over the whole
        span and rounded once per meter, so they are the same at any
        granularity, while the intervals, each rounded on its own, can add up to
        a little less.
      required:
        - start
        - end
        - cost
        - features
      properties:
        start:
          type: integer
          format: int64
          description: Unix timestamp in seconds.
        end:
          type: integer
          format: int64
          description: Unix timestamp in seconds, exclusive.
        cost:
          type: number
          description: >-
            Usage over the window at the account's rates, in dollars to the
            cent.
        features:
          type: object
          description: Quantity and cost per meter over the window, keyed by feature id.
          additionalProperties:
            $ref: '#/components/schemas/BillingUsageQuantity'
    BillingUsageInterval:
      type: object
      required:
        - start
        - end
        - cost
        - features
      properties:
        start:
          type: integer
          format: int64
          description: Unix timestamp in seconds.
        end:
          type: integer
          format: int64
          description: Unix timestamp in seconds, exclusive.
        cost:
          type: number
          description: >-
            The interval's usage at the account's rates, in dollars to the cent;
            the sum of `features`.
        features:
          type: object
          description: Quantity and cost per meter in the interval, keyed by feature id.
          additionalProperties:
            $ref: '#/components/schemas/BillingUsageQuantity'
    BillingPeriod:
      type: object
      required:
        - start
        - end
      properties:
        start:
          type: integer
          format: int64
          description: Unix timestamp in seconds.
        end:
          type: integer
          format: int64
          description: Unix timestamp in seconds.
    BillingUsageQuantity:
      type: object
      required:
        - quantity
        - unit
        - cost
      properties:
        quantity:
          type: number
          description: In the meter's own unit.
        unit:
          type: string
          description: >-
            The meter's unit, such as vCPU-hours, GiB-hours, GiB-months or
            calls.
        cost:
          type: number
          description: >-
            Quantity at the account's rate, in dollars to the cent; zero for a
            meter the plan does not price.
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
      type: http
      scheme: bearer
      description: Archil API key, including its `key-` prefix

````

This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
