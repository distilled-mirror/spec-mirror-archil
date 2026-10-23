> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Get Summary

> Returns a billing period's statement as a sum: `meteredCostCents`, the period's usage at the account's rates with a breakdown by meter; `adjustments`, the lines the plan applies (the included credits taken off, the base price added); and `billedCostCents`, what they come to, which is what the period invoices or would if it ended now. Also the account's standing on the plan's credit pool and, for the current period, a projection to its end at the recent daily pace. A past period's lines are read from the invoices issued for it. The only billing route where the plan's base price and included credits enter. Money is in integer cents. On a plan that prices no usage, `priced` is false and the amounts are zero.




## OpenAPI

````yaml GET /api/billing/summary
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
  /api/billing/summary:
    get:
      tags:
        - Billing
      summary: Get a billing period's statement as a sum
      description: >
        Returns a billing period's statement as a sum: `meteredCostCents`, the
        period's usage at the account's rates with a breakdown by meter;
        `adjustments`, the lines the plan applies (the included credits taken
        off, the base price added); and `billedCostCents`, what they come to,
        which is what the period invoices or would if it ended now. Also the
        account's standing on the plan's credit pool and, for the current
        period, a projection to its end at the recent daily pace. A past
        period's lines are read from the invoices issued for it. The only
        billing route where the plan's base price and included credits enter.
        Money is in integer cents. On a plan that prices no usage, `priced` is
        false and the amounts are zero.
      operationId: getBillingSummary
      parameters:
        - $ref: '#/components/parameters/SummaryPeriod'
      responses:
        '200':
          description: >-
            The period's statement as a sum, with a projection for the current
            period
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_BillingSummary'
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '502':
          $ref: '#/components/responses/InternalError'
      security:
        - ApiKeyAuth:
            - billing:summary:get
        - SessionTokenAuth: []
components:
  parameters:
    SummaryPeriod:
      name: period
      in: query
      description: >
        Which billing period: `current` (the default), or a YYYY-MM-DD date
        (UTC) inside a past period that has been invoiced; the day a period
        starts on selects it. A past period is read whole from the invoices
        issued for it, named in `invoices`: its metered quantities and cost as
        billed, the included credits taken off, the base price, and any other
        line under its own name.
      schema:
        type: string
        default: current
        example: '2026-09-20'
  schemas:
    ApiResponse_BillingSummary:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
        data:
          $ref: '#/components/schemas/BillingSummary'
    BillingSummary:
      type: object
      required:
        - priced
        - currency
        - billingPeriod
        - plan
        - meteredCostCents
        - meteredCostBreakdown
        - adjustments
        - billedCostCents
      properties:
        priced:
          type: boolean
          description: >-
            False when the plan prices no usage; the quantities are still
            reported, the amounts are zero, and `credit` and `projected` are
            absent.
        currency:
          type: string
        billingPeriod:
          description: >-
            The period read; the UTC calendar month for an account without a
            billing cycle.
          allOf:
            - $ref: '#/components/schemas/BillingPeriod'
        plan:
          $ref: '#/components/schemas/BillingSummaryPlan'
        credit:
          description: >
            Billing's own standing on the plan's credit pool, in credits, on the
            current period only (a past period's is not kept; its `adjustments`
            show the credits drawn). `used` equals `meteredCostCents` at the
            overage price unless a usage report arrived after a period boundary,
            which billing credits to the period it arrived in.
          allOf:
            - $ref: '#/components/schemas/BillingCreditStanding'
        meteredCostCents:
          type: integer
          format: int64
          description: >-
            The period's usage at the account's rates, before any adjustment, in
            integer cents; the sum of `meteredCostBreakdown`. On a past period,
            as its invoice billed it.
        meteredCostBreakdown:
          type: object
          description: >-
            The metered cost by meter, with quantities, keyed by feature id.
            Sums to `meteredCostCents`.
          additionalProperties:
            $ref: '#/components/schemas/BillingFeatureCost'
        adjustments:
          type: array
          description: >-
            The lines between `meteredCostCents` and `billedCostCents`, which
            sum to their difference.
          items:
            $ref: '#/components/schemas/BillingAdjustment'
        billedCostCents:
          type: integer
          format: int64
          description: >-
            What the period invoices, or would if it ended now;
            `meteredCostCents` plus the adjustments, to the cent.
        projected:
          description: >-
            The period end at the recent daily pace; absent for a past period or
            without an elapsed day of usage.
          allOf:
            - $ref: '#/components/schemas/BillingSummaryProjection'
        invoices:
          type: array
          description: >-
            On a past period, the invoices its lines were read from, oldest
            first; the base price is billed as a period starts and the usage as
            it ends. Absent on the current period, which is not invoiced yet.
          items:
            $ref: '#/components/schemas/BillingIssuedInvoice'
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
    BillingSummaryPlan:
      type: object
      required:
        - id
        - basePriceCents
        - includedCredits
        - overagePricePerCreditCents
      properties:
        id:
          type: string
          description: The plan id, such as `enterprise`.
        basePriceCents:
          type: integer
          format: int64
          description: The recurring fee per billing period, in integer cents.
        includedCredits:
          type: number
          description: Credits the plan grants each period.
        overagePricePerCreditCents:
          type: integer
          format: int64
          description: What each credit past the granted ones costs, in integer cents.
    BillingCreditStanding:
      type: object
      required:
        - granted
        - used
        - remaining
      properties:
        granted:
          type: number
          description: Credits the period grants.
        used:
          type: number
          description: Credits drawn so far, rounded to two decimals.
        remaining:
          type: number
          description: Credits left before overage, rounded to two decimals.
    BillingFeatureCost:
      type: object
      required:
        - quantity
        - unit
        - costCents
      properties:
        quantity:
          type: number
          description: In the meter's own unit.
        unit:
          type: string
        costCents:
          type: integer
          format: int64
          description: Quantity at the account's rate, in integer cents.
    BillingAdjustment:
      type: object
      required:
        - type
        - name
        - amountCents
      properties:
        type:
          type: string
          enum:
            - included_credits
            - base_price
            - other
          description: >-
            Which plan term the line applies; the key to branch on. `other` is a
            line the processor billed on a past period that is neither, such as
            a proration, carried with its own `name`.
        name:
          type: string
          description: The line's label, such as `Included credits` or `Base price`.
        amountCents:
          type: integer
          format: int64
          description: In integer cents; negative for a discount.
    BillingSummaryProjection:
      type: object
      required:
        - meteredCostCents
        - billedCostCents
        - dailyRateCents
        - paceStart
        - paceEnd
        - daysRemaining
      properties:
        meteredCostCents:
          type: integer
          format: int64
          description: >-
            Usage at the account's rates the period is projected to reach, in
            integer cents.
        billedCostCents:
          type: integer
          format: int64
          description: What that usage would invoice, in integer cents.
        dailyRateCents:
          type: integer
          format: int64
          description: >-
            The pace the projection extends, in integer cents per day;
            `meteredCostCents` so far plus `dailyRateCents` × `daysRemaining`.
        paceStart:
          type: integer
          format: int64
          description: >-
            Unix timestamp in seconds; the pace is the mean over the elapsed UTC
            days from here to `paceEnd`, the last ones before now inside the
            period.
        paceEnd:
          type: integer
          format: int64
          description: Unix timestamp in seconds, exclusive.
        daysRemaining:
          type: number
          description: Days from now to the period end.
    BillingIssuedInvoice:
      type: object
      required:
        - id
        - status
      properties:
        id:
          type: string
        status:
          type: string
          description: open, paid or uncollectible.
        hostedInvoiceUrl:
          type: string
          description: The invoice as the billing processor hosts it.
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
