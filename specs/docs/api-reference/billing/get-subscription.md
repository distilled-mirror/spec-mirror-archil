> ## Documentation Index
> Fetch the complete documentation index at: https://docs.archil.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Get Subscription

> Returns the account's plan and the state of its subscription: the plan, the subscription's status and billing period, a scheduled change or cancellation if one is pending, payment health (a default payment method on file, an open invoice and when it is retried) and the actions the console offers for it. Accounts that share billing share the subscription and open-invoice information. The plan here is the one the `usage` and `summary` routes price usage under.




## OpenAPI

````yaml GET /api/billing/subscription
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
  /api/billing/subscription:
    get:
      tags:
        - Billing
      summary: Get the subscription's plan and state
      description: >
        Returns the account's plan and the state of its subscription: the plan,
        the subscription's status and billing period, a scheduled change or
        cancellation if one is pending, payment health (a default payment method
        on file, an open invoice and when it is retried) and the actions the
        console offers for it. Accounts that share billing share the
        subscription and open-invoice information. The plan here is the one the
        `usage` and `summary` routes price usage under.
      operationId: getBillingSubscription
      responses:
        '200':
          description: The subscription's plan and state
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiResponse_BillingSubscription'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '502':
          $ref: '#/components/responses/InternalError'
      security:
        - ApiKeyAuth:
            - billing:subscription:get
        - SessionTokenAuth: []
components:
  schemas:
    ApiResponse_BillingSubscription:
      type: object
      required:
        - success
        - data
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/BillingSubscription'
    BillingSubscription:
      type: object
      required:
        - billingOwner
        - billingMode
        - plan
        - subscriptionStatus
        - hasPendingUpdate
        - paymentHealth
        - actions
      properties:
        billingOwner:
          $ref: '#/components/schemas/BillingOwner'
        billingMode:
          type: string
          enum:
            - not_setup
            - self_serve
          description: >-
            `not_setup` until the account has a billing customer; `self_serve`
            once it does.
        plan:
          type: string
          enum:
            - free
            - developer
            - team
            - enterprise
            - unknown
          description: The plan usage is priced under.
        subscriptionStatus:
          type: string
          enum:
            - none
            - active
            - past_due
          description: >-
            Current subscription status; `none` without a paid subscription,
            `past_due` while a failing payment is retried and the plan is kept.
        billingPeriod:
          $ref: '#/components/schemas/BillingPeriod'
        scheduledCancellationAt:
          type: integer
          format: int64
          description: Unix timestamp in seconds.
        scheduledChange:
          $ref: '#/components/schemas/BillingScheduledChange'
        hasPendingUpdate:
          type: boolean
        paymentHealth:
          $ref: '#/components/schemas/BillingPaymentHealth'
        actions:
          $ref: '#/components/schemas/BillingSubscriptionActions'
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
    BillingOwner:
      type: object
      required:
        - accountId
        - type
      properties:
        accountId:
          type: string
          description: Selected account ID.
        type:
          type: string
          enum:
            - personal
            - organization
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
    BillingScheduledChange:
      type: object
      required:
        - type
        - effectiveAt
      properties:
        type:
          type: string
          enum:
            - schedule_transition
            - plan_change
        effectiveAt:
          type: integer
          format: int64
          description: Unix timestamp in seconds.
        targetPlan:
          type: string
          enum:
            - developer
            - team
            - enterprise
    BillingPaymentHealth:
      type: object
      required:
        - status
        - hasDefaultPaymentMethod
      properties:
        status:
          type: string
          enum:
            - not_applicable
            - invoice_open
            - action_required
            - missing_payment_method
            - healthy
        hasDefaultPaymentMethod:
          type: boolean
        openInvoiceId:
          type: string
        amountRemaining:
          type: number
          description: What is still owed on the open invoice, in dollars to the cent.
        openInvoiceCurrency:
          type: string
        openInvoiceDueDate:
          type: integer
          format: int64
        openInvoiceNextPaymentAttempt:
          type: integer
          format: int64
          description: When the payment processor retries automatically, if scheduled.
        pendingOperationId:
          type: string
          description: >-
            An unfinished purchase or plan change awaiting payment, which the
            console resumes.
    BillingSubscriptionActions:
      type: object
      description: >-
        What the console offers the account from its billing page, given the
        subscription's state.
      required:
        - canStartCheckout
        - canManagePaymentMethods
        - canUpdateBillingProfile
        - canViewInvoices
        - canChangePlan
        - canCancel
        - canKeepSubscription
        - canPayOpenInvoice
        - canWithdrawScheduledChange
      properties:
        canStartCheckout:
          type: boolean
        canManagePaymentMethods:
          type: boolean
        canUpdateBillingProfile:
          type: boolean
        canViewInvoices:
          type: boolean
        canChangePlan:
          type: boolean
        canCancel:
          type: boolean
        canKeepSubscription:
          type: boolean
        canPayOpenInvoice:
          type: boolean
        canWithdrawScheduledChange:
          type: boolean
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
