> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Get Summary

> Returns a billing period's statement as billing itself states it: `lines`, each line invoiced for the period or previewed for its end (a fixed fee, a meter's usage billed past its grants, a credit taken off, anything else under its own name), and `billedCost`, their sum, which is what the period invoices or would if it ended now. On the current period also `credits`, the grants the account still draws on, and a projection to the period end at the recent daily pace. Nothing here is derived from the plan's terms: a plan billing can express, this route reports. Money is in dollars to the cent, before tax. A line is before discounts, as billing reports it, and what a reward takes off follows as an `other` line named `Discount`: off each line previewed for the period end, and off each invoice as a whole (its total under its lines' sum), on the span of the invoice's fixed fee when it has one, else of its first line on the period. So the sum is what the period invoiced, or will.




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
      summary: Get a billing period's statement
      description: >
        Returns a billing period's statement as billing itself states it:
        `lines`, each line invoiced for the period or previewed for its end (a
        fixed fee, a meter's usage billed past its grants, a credit taken off,
        anything else under its own name), and `billedCost`, their sum, which is
        what the period invoices or would if it ended now. On the current period
        also `credits`, the grants the account still draws on, and a projection
        to the period end at the recent daily pace. Nothing here is derived from
        the plan's terms: a plan billing can express, this route reports. Money
        is in dollars to the cent, before tax. A line is before discounts, as
        billing reports it, and what a reward takes off follows as an `other`
        line named `Discount`: off each line previewed for the period end, and
        off each invoice as a whole (its total under its lines' sum), on the
        span of the invoice's fixed fee when it has one, else of its first line
        on the period. So the sum is what the period invoiced, or will.
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
        starts on selects it. A past period's lines are read whole from the
        invoices issued for it, named in `invoices`, under the plan they were
        issued for, whatever the account is on now.
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
        - currency
        - billingPeriod
        - plan
        - lines
        - billedCost
        - credits
      properties:
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
        lines:
          type: array
          description: >
            The statement's lines, oldest invoice first, then the lines
            previewed for the period end: on the current period, what was billed
            as it opened (the base price, a proration) and what billing will
            bill as it closes (the usage past the grants, the credits taken
            off); on a past period, everything the invoices issued for it carry.
            Empty on a plan that bills nothing.
          items:
            $ref: '#/components/schemas/BillingLine'
        billedCost:
          type: number
          description: >-
            What the period invoices, or would if it ended now; the lines' sum,
            to the cent.
        credits:
          type: array
          description: >
            The grants the account still draws on, on the current period only:
            each with something left or that will refill. A spent one-time grant
            and an expired one are not listed; a past period lists none (its
            `credit` lines show the draw). This is a grant's standing, not a
            charge: a `credit` line in `lines` is what a grant took off this
            period's bill, in money, and the two need not pair up. A grant
            nothing has drawn from yet has no line; an included quantity (tool
            calls, disks) is spent without one, since billing invoices only the
            usage past it; and several grants on one feature draw as one line.
          items:
            $ref: '#/components/schemas/BillingCreditGrant'
        projected:
          description: >-
            The period end at the recent daily pace; absent for a past period, a
            plan that prices no usage, or without an elapsed day of usage.
          allOf:
            - $ref: '#/components/schemas/BillingSummaryProjection'
        invoices:
          type: array
          description: >-
            The invoices the lines with an `invoiceId` are on, oldest first.
            Absent when nothing is invoiced for the period yet.
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
        - name
      properties:
        id:
          type: string
          description: The plan id, such as `enterprise`.
        name:
          type: string
          description: The plan's name as billing shows it.
    BillingLine:
      type: object
      description: >-
        One line of the period's statement, as billing invoiced it or previews
        it.
      required:
        - type
        - name
        - start
        - end
        - amount
      properties:
        type:
          type: string
          enum:
            - fixed
            - usage
            - credit
            - other
          description: >
            The key to branch on. `fixed` is a charge over the whole period (the
            plan's base price; an item paid for up front per period, such as a
            block of sandboxes, on its feature with its quantity); `usage` is a
            meter's usage billed past its grants, with its quantity; `credit` is
            a credit taken off, on the feature it draws, in money (what is left
            of the grant is in `credits`); `other` is anything else billed
            inside the period, such as a proration, a one-off purchase or a
            reward (`Discount`), carried with its own `name`.
        featureId:
          type: string
          description: >-
            On a `usage` or `credit` line, and a `fixed` line for an item paid
            up front, the feature it is on.
        name:
          type: string
          description: The line's label as billing wrote it.
        quantity:
          type: number
          description: On a line with a `featureId`, the quantity billed, in `unit`.
        unit:
          type: string
          description: The quantity's unit, such as vCPU-hours or USD.
        start:
          type: integer
          format: int64
          description: >-
            Unix timestamp in seconds; the span the line covers. A line is on
            the period its span ends in, or for an item billed over several
            periods (a yearly add-on) the period it starts in, and on no other.
        end:
          type: integer
          format: int64
          description: Unix timestamp in seconds, exclusive.
        amount:
          type: number
          description: In dollars to the cent; negative for a credit.
        invoiceId:
          type: string
          description: >-
            The invoice the line is on, one of `invoices`; absent for a line
            previewed for the period end and not invoiced yet.
    BillingCreditGrant:
      type: object
      description: >-
        One grant the account draws before it is billed, described by when it
        refills or expires rather than by a kind.
      required:
        - featureId
        - name
        - unit
        - granted
        - remaining
        - resetsAt
        - expiresAt
      properties:
        featureId:
          type: string
          description: The feature the grant is on, such as `prepaid_credit`.
        name:
          type: string
          description: The feature's name, such as `Prepaid credit (USD)`.
        unit:
          type: string
          description: The unit the grant is counted in, such as USD or disks.
        granted:
          type: number
          description: What the grant carries, per period for one that refills.
        remaining:
          type: number
          description: What is left on it, rounded to two decimals.
        resetsAt:
          type: integer
          format: int64
          nullable: true
          description: >-
            Unix timestamp in seconds when the grant refills; null for a
            one-time grant.
        expiresAt:
          type: integer
          format: int64
          nullable: true
          description: >-
            Unix timestamp in seconds when the grant lapses; null when it does
            not.
    BillingSummaryProjection:
      type: object
      required:
        - billedCost
        - dailyRate
      properties:
        billedCost:
          type: number
          description: >-
            What the period is projected to invoice, in dollars; what is billed
            so far, plus the usage still to come once it has drawn what is left
            of each grant.
        dailyRate:
          type: number
          description: >-
            The pace the projection extends, in dollars of usage per day; the
            mean over the last three elapsed UTC days inside the period.
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
          enum:
            - draft
            - open
            - paid
            - uncollectible
          description: >-
            The invoice's status with the billing processor. `draft` is a
            renewal the processor has not finalized yet, for about an hour after
            the period rolls; it already carries every line it will bill. Voided
            invoices are not listed.
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
