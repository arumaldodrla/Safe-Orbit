# Affiliate Marketing & Payment Integration System

**Comprehensive Technical Specification**

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture](#architecture)
3. [Zoho Thrive Integration (Affiliate Marketing)](#zoho-thrive-integration)
4. [Payment Processing Options](#payment-processing-options)
5. [Zoho Billing Integration (Subscription Management)](#zoho-billing-integration)
6. [Authorize.net Integration](#authorizenet-integration)
7. [Zoho Books Integration (Invoicing)](#zoho-books-integration)
8. [Discount Code System](#discount-code-system)
9. [Affiliate Portal](#affiliate-portal)
10. [Commission Tracking & Payouts](#commission-tracking)
11. [Implementation Roadmap](#implementation-roadmap)
12. [API Reference](#api-reference)

---

## System Overview

SafeOrbit will implement a comprehensive affiliate marketing and payment system that allows:

1. **Anyone to become an affiliate** with custom discount codes
2. **Flexible payment processing** via Authorize.net
3. **Automated subscription management** via Zoho Billing
4. **Professional invoicing** via Zoho Books
5. **Commission tracking and payouts** via Zoho Thrive

### Key Capabilities

✅ **Self-Service Affiliate Signup**: Anyone can join the affiliate program  
✅ **Custom Discount Codes**: Affiliates create their own branded codes  
✅ **Automatic Commission Tracking**: Real-time tracking of referrals and earnings  
✅ **Flexible Payment Options**: Direct charge via Authorize.net OR Zoho Billing subscriptions  
✅ **Automated Invoicing**: Zoho Books generates invoices for all transactions  
✅ **Multi-Tier Commissions**: Support for different commission structures  
✅ **Affiliate Dashboard**: Real-time earnings, conversion stats, and marketing materials  

---

## Architecture

### High-Level Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         SafeOrbit Platform                      │
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    │
│  │   Customer   │───▶│   Checkout   │───▶│   Payment    │    │
│  │   Applies    │    │   Process    │    │  Processing  │    │
│  │ Discount Code│    │              │    │              │    │
│  └──────────────┘    └──────────────┘    └──────────────┘    │
│                             │                     │            │
│                             ▼                     ▼            │
│                      ┌──────────────┐    ┌──────────────┐    │
│                      │ Zoho Thrive  │    │ Authorize.net│    │
│                      │  (Affiliate  │    │      OR      │    │
│                      │   Tracking)  │    │ Zoho Billing │    │
│                      └──────────────┘    └──────────────┘    │
│                             │                     │            │
│                             ▼                     ▼            │
│                      ┌──────────────┐    ┌──────────────┐    │
│                      │ Commission   │    │  Zoho Books  │    │
│                      │   Tracking   │    │  (Invoicing) │    │
│                      └──────────────┘    └──────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Technology Stack

**Affiliate Marketing**: Zoho Thrive  
**Subscription Management**: Zoho Billing  
**Payment Gateway**: Authorize.net  
**Invoicing**: Zoho Books  
**Backend**: SafeOrbit tRPC API  
**Database**: MySQL (domains, users, affiliate_links, commissions)  

---

## Zoho Thrive Integration

### What is Zoho Thrive?

Zoho Thrive is a performance-based marketing platform that provides:
- Affiliate program management
- Referral tracking
- Commission calculation
- Discount code management
- Affiliate portal and dashboards

### Key Concepts

**Brand Code**: Unique identifier for SafeOrbit in Zoho Thrive  
**Secret Key**: Used for HMAC encryption when pushing data to Zoho Thrive  
**OAuth Scope**: `Thrive.Referral.CREATE` for purchase tracking  
**Purchase API**: Endpoint to push transaction data for commission tracking  

### Setup Steps

1. **Create Zoho Thrive Account**
   - Sign up at https://www.zoho.com/thrive/
   - Create a new "Brand" for SafeOrbit
   - Note the Brand Code and Secret Key

2. **Configure Affiliate Program**
   - Set commission structure (e.g., 20% recurring for 12 months)
   - Define referral tasks:
     - **Referral Signup**: Commission when referred user creates account
     - **Referral Purchase**: Commission when referred user subscribes
   - Set payout thresholds (e.g., minimum $50 to withdraw)

3. **Enable Discount Codes**
   - Allow affiliates to create custom discount codes
   - Set discount ranges (e.g., 10-30% off first month)
   - Link discount codes to affiliate tracking

4. **Generate OAuth Credentials**
   - Create OAuth app in Zoho Developer Console
   - Request scope: `Thrive.Referral.CREATE`
   - Store Client ID and Client Secret securely

### Integration Flow

#### Step 1: Affiliate Signup

When someone wants to become an affiliate:

```typescript
// User clicks "Become an Affiliate" on SafeOrbit
// SafeOrbit creates affiliate account in Zoho Thrive

POST https://thrive.zoho.com/api/v1/affiliates
Authorization: Bearer {oauth_token}

{
  "email": "affiliate@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "company": "Web Agency Inc",
  "phone": "+1234567890"
}

// Response includes affiliate_id and unique referral link
{
  "affiliate_id": "AFF-12345",
  "referral_link": "https://safeorbit.com?ref=johndoe",
  "status": "active"
}
```

#### Step 2: Generate Discount Code

Affiliate creates custom discount code:

```typescript
// Affiliate chooses code: "WEBAGENCY20"
// SafeOrbit creates discount in Zoho Thrive + Zoho Billing

// In Zoho Thrive: Link code to affiliate
POST https://thrive.zoho.com/api/v1/discount-codes
{
  "code": "WEBAGENCY20",
  "affiliate_id": "AFF-12345",
  "discount_type": "percentage",
  "discount_value": 20,
  "applies_to": "first_month"
}

// In Zoho Billing: Create coupon
POST https://billing.zoho.com/api/v1/coupons
{
  "coupon_code": "WEBAGENCY20",
  "discount_type": "percentage",
  "discount": 20,
  "apply_to": "invoice",
  "coupon_type": "one_time"
}
```

#### Step 3: Customer Uses Discount Code

Customer applies discount code at checkout:

```typescript
// Frontend: Customer enters "WEBAGENCY20"
// Backend validates code in Zoho Billing

GET https://billing.zoho.com/api/v1/coupons/WEBAGENCY20

// Response includes discount details
{
  "coupon_id": "12345",
  "coupon_code": "WEBAGENCY20",
  "discount": 20,
  "discount_type": "percentage",
  "status": "active"
}

// Store affiliate tracking in session/cookie
{
  "affiliate_code": "johndoe",
  "discount_code": "WEBAGENCY20",
  "affiliate_id": "AFF-12345"
}
```

#### Step 4: Push Purchase to Zoho Thrive

After successful payment, track the purchase:

```typescript
// Generate OAuth token
const oauthToken = await generateZohoThriveToken();

// Push purchase details to Zoho Thrive
POST https://thrive.zoho.com/thrive-publicapi/v1/{brand-code}/purchase
Authorization: Bearer {oauthToken}
Content-Type: application/json

{
  "email": "customer@example.com",
  "amount": 1456, // Amount in cents
  "order_id": "purchase_pd42119_3",
  "zt_customer_id": "cust_165113031049",
  "thrive_digest": "fa04d2467feb9bb70be70fece58d6ebc",
  "product_plan_name": "Starter Monthly"
}

// Response: 204 Success
// Commission is automatically calculated and credited to affiliate
```

#### Step 5: Affiliate Views Earnings

Affiliate logs into their dashboard:

```typescript
// SafeOrbit fetches affiliate stats from Zoho Thrive

GET https://thrive.zoho.com/api/v1/affiliates/AFF-12345/stats

{
  "total_referrals": 15,
  "successful_conversions": 8,
  "total_earnings": 456.00,
  "pending_payout": 456.00,
  "lifetime_value": 1200.00,
  "conversion_rate": 53.3
}
```

---

## Payment Processing Options

SafeOrbit will support **two payment processing approaches**:

### Option A: Zoho Billing (Recommended for Subscriptions)

**Best For**: Recurring subscriptions with automated billing

**Pros**:
- Native subscription management
- Automatic renewals and dunning
- Built-in customer portal
- Integrated with Zoho Books for invoicing
- Supports multiple payment gateways (Authorize.net, Stripe, PayPal)

**Cons**:
- Additional Zoho Billing subscription cost
- Slightly more complex setup

**Flow**:
```
Customer → Zoho Billing Checkout → Authorize.net → Payment Success
         → Zoho Billing creates subscription
         → Zoho Books generates invoice
         → SafeOrbit receives webhook
         → Zoho Thrive tracks commission
```

---

### Option B: Direct Authorize.net Charge + Zoho Books Invoice

**Best For**: One-time payments or custom billing logic

**Pros**:
- Direct control over payment flow
- No Zoho Billing subscription needed
- Simpler for one-time payments

**Cons**:
- Manual subscription management
- Must build own renewal logic
- More code to maintain

**Flow**:
```
Customer → SafeOrbit Checkout → Authorize.net API → Payment Success
         → SafeOrbit creates subscription record
         → Zoho Books API generates invoice
         → Zoho Thrive tracks commission
```

---

### Recommendation: Hybrid Approach

**Use Zoho Billing for**:
- Monthly/annual subscriptions (Starter, Professional, Business plans)
- Automated renewals
- Customer self-service portal

**Use Direct Authorize.net for**:
- One-time add-ons
- Custom enterprise deals
- Manual invoicing scenarios

---

## Zoho Billing Integration

### Setup

1. **Create Zoho Billing Account**
   - Sign up at https://www.zoho.com/billing/
   - Connect Authorize.net as payment gateway
   - Configure tax settings and currency

2. **Create Products & Plans**

```json
// Starter Plan
{
  "plan_code": "starter-monthly",
  "name": "Starter Plan - Monthly",
  "recurring_price": 15.00,
  "interval": 1,
  "interval_unit": "months",
  "trial_period": 14,
  "trial_period_unit": "days"
}

// Professional Plan
{
  "plan_code": "professional-monthly",
  "name": "Professional Plan - Monthly",
  "recurring_price": 49.00,
  "interval": 1,
  "interval_unit": "months",
  "trial_period": 14,
  "trial_period_unit": "days"
}

// Business Plan
{
  "plan_code": "business-monthly",
  "name": "Business Plan - Monthly",
  "recurring_price": 149.00,
  "interval": 1,
  "interval_unit": "months",
  "trial_period": 14,
  "trial_period_unit": "days"
}
```

3. **Generate API Credentials**
   - Go to Settings → Developer Space
   - Create new OAuth app
   - Note Organization ID and Auth Token

### API Implementation

#### Create Subscription

```typescript
// server/services/zoho-billing.ts

import axios from 'axios';

const ZOHO_BILLING_API = 'https://billing.zoho.com/api/v1';
const ZOHO_ORG_ID = process.env.ZOHO_BILLING_ORG_ID;
const ZOHO_AUTH_TOKEN = process.env.ZOHO_BILLING_AUTH_TOKEN;

export async function createSubscription(data: {
  customerId?: string;
  customerEmail: string;
  customerName: string;
  planCode: string;
  couponCode?: string;
  startsAt?: string;
}) {
  const payload: any = {
    plan: {
      plan_code: data.planCode
    },
    starts_at: data.startsAt || new Date().toISOString(),
  };

  // New customer
  if (!data.customerId) {
    payload.customer = {
      email: data.customerEmail,
      display_name: data.customerName,
    };
  } else {
    payload.customer_id = data.customerId;
  }

  // Apply coupon if provided
  if (data.couponCode) {
    payload.coupon_code = data.couponCode;
  }

  const response = await axios.post(
    `${ZOHO_BILLING_API}/subscriptions`,
    payload,
    {
      headers: {
        'Authorization': `Zoho-oauthtoken ${ZOHO_AUTH_TOKEN}`,
        'X-com-zoho-subscriptions-organizationid': ZOHO_ORG_ID,
        'Content-Type': 'application/json',
      },
    }
  );

  return response.data.subscription;
}
```

#### Handle Webhooks

```typescript
// server/routers/webhooks.ts

import { router, publicProcedure } from '../_core/trpc';
import { z } from 'zod';

export const webhookRouter = router({
  zohoBilling: publicProcedure
    .input(z.object({
      event_type: z.string(),
      data: z.any(),
    }))
    .mutation(async ({ input, ctx }) => {
      const { event_type, data } = input;

      switch (event_type) {
        case 'subscription_created':
          await handleSubscriptionCreated(data);
          break;

        case 'subscription_renewed':
          await handleSubscriptionRenewed(data);
          break;

        case 'subscription_cancelled':
          await handleSubscriptionCancelled(data);
          break;

        case 'payment_succeeded':
          await handlePaymentSucceeded(data);
          break;

        case 'payment_failed':
          await handlePaymentFailed(data);
          break;

        default:
          console.log(`Unhandled webhook event: ${event_type}`);
      }

      return { success: true };
    }),
});

async function handleSubscriptionCreated(data: any) {
  const subscription = data.subscription;

  // Update user's subscription status in SafeOrbit database
  await db.update(users)
    .set({
      subscriptionId: subscription.subscription_id,
      subscriptionStatus: 'active',
      planCode: subscription.plan.plan_code,
      subscriptionStartsAt: new Date(subscription.current_term_starts_at),
      subscriptionEndsAt: new Date(subscription.current_term_ends_at),
    })
    .where(eq(users.email, subscription.customer.email));

  // Push purchase to Zoho Thrive for affiliate commission
  await pushPurchaseToZohoThrive({
    email: subscription.customer.email,
    amount: subscription.amount * 100, // Convert to cents
    orderId: subscription.subscription_id,
    customerId: subscription.customer.customer_id,
    planName: subscription.plan.name,
  });
}

async function handlePaymentSucceeded(data: any) {
  const payment = data.payment;

  // Send receipt email
  await sendReceiptEmail({
    email: payment.customer.email,
    amount: payment.amount,
    invoiceNumber: payment.invoice_number,
    paymentDate: payment.payment_date,
  });

  // Notify owner if it's a new customer
  if (payment.is_first_payment) {
    await notifyOwner({
      title: 'New Subscription',
      content: `${payment.customer.display_name} (${payment.customer.email}) subscribed to ${payment.plan.name} for $${payment.amount}/month`,
    });
  }
}
```

---

## Authorize.net Integration

### Setup

1. **Create Authorize.net Account**
   - Sign up at https://www.authorize.net/
   - Get API Login ID and Transaction Key
   - Enable test mode for development

2. **Connect to Zoho Billing** (if using Option A)
   - In Zoho Billing: Settings → Payment Gateways
   - Select Authorize.net
   - Enter API credentials
   - Test connection

3. **Direct Integration** (if using Option B)

```typescript
// server/services/authorize-net.ts

import axios from 'axios';

const AUTHORIZE_NET_API = process.env.NODE_ENV === 'production'
  ? 'https://api.authorize.net/xml/v1/request.api'
  : 'https://apitest.authorize.net/xml/v1/request.api';

const API_LOGIN_ID = process.env.AUTHORIZE_NET_API_LOGIN_ID;
const TRANSACTION_KEY = process.env.AUTHORIZE_NET_TRANSACTION_KEY;

export async function chargeCard(data: {
  amount: number;
  cardNumber: string;
  expirationDate: string; // MMYY
  cardCode: string; // CVV
  customerEmail: string;
  customerName: string;
  invoiceNumber: string;
  description: string;
}) {
  const payload = {
    createTransactionRequest: {
      merchantAuthentication: {
        name: API_LOGIN_ID,
        transactionKey: TRANSACTION_KEY,
      },
      transactionRequest: {
        transactionType: 'authCaptureTransaction',
        amount: data.amount.toFixed(2),
        payment: {
          creditCard: {
            cardNumber: data.cardNumber,
            expirationDate: data.expirationDate,
            cardCode: data.cardCode,
          },
        },
        customer: {
          email: data.customerEmail,
        },
        billTo: {
          firstName: data.customerName.split(' ')[0],
          lastName: data.customerName.split(' ').slice(1).join(' '),
        },
        order: {
          invoiceNumber: data.invoiceNumber,
          description: data.description,
        },
      },
    },
  };

  const response = await axios.post(AUTHORIZE_NET_API, payload, {
    headers: { 'Content-Type': 'application/json' },
  });

  const result = response.data.transactionResponse;

  if (result.responseCode === '1') {
    // Success
    return {
      success: true,
      transactionId: result.transId,
      authCode: result.authCode,
      message: result.messages[0].description,
    };
  } else {
    // Declined or error
    throw new Error(result.errors[0].errorText);
  }
}

export async function createRecurringSubscription(data: {
  amount: number;
  intervalLength: number; // 1 for monthly, 12 for annual
  intervalUnit: 'months' | 'days';
  startDate: string; // YYYY-MM-DD
  totalOccurrences: number; // 9999 for indefinite
  cardNumber: string;
  expirationDate: string;
  customerEmail: string;
  customerName: string;
  subscriptionName: string;
}) {
  const payload = {
    ARBCreateSubscriptionRequest: {
      merchantAuthentication: {
        name: API_LOGIN_ID,
        transactionKey: TRANSACTION_KEY,
      },
      subscription: {
        name: data.subscriptionName,
        paymentSchedule: {
          interval: {
            length: data.intervalLength,
            unit: data.intervalUnit,
          },
          startDate: data.startDate,
          totalOccurrences: data.totalOccurrences,
        },
        amount: data.amount.toFixed(2),
        payment: {
          creditCard: {
            cardNumber: data.cardNumber,
            expirationDate: data.expirationDate,
          },
        },
        customer: {
          email: data.customerEmail,
        },
        billTo: {
          firstName: data.customerName.split(' ')[0],
          lastName: data.customerName.split(' ').slice(1).join(' '),
        },
      },
    },
  };

  const response = await axios.post(AUTHORIZE_NET_API, payload, {
    headers: { 'Content-Type': 'application/json' },
  });

  const result = response.data;

  if (result.messages.resultCode === 'Ok') {
    return {
      success: true,
      subscriptionId: result.subscriptionId,
    };
  } else {
    throw new Error(result.messages.message[0].text);
  }
}
```

---

## Zoho Books Integration

### Purpose

Zoho Books generates professional invoices for all transactions, regardless of whether payment was processed through Zoho Billing or directly via Authorize.net.

### Setup

1. **Create Zoho Books Account**
   - Sign up at https://www.zoho.com/books/
   - Configure company details (logo, address, tax info)
   - Set up invoice templates

2. **Connect to Zoho Billing** (if using Option A)
   - Zoho Billing automatically syncs invoices to Zoho Books
   - No additional setup needed

3. **API Integration** (if using Option B)

```typescript
// server/services/zoho-books.ts

import axios from 'axios';

const ZOHO_BOOKS_API = 'https://books.zoho.com/api/v3';
const ZOHO_BOOKS_ORG_ID = process.env.ZOHO_BOOKS_ORG_ID;
const ZOHO_BOOKS_AUTH_TOKEN = process.env.ZOHO_BOOKS_AUTH_TOKEN;

export async function createInvoice(data: {
  customerName: string;
  customerEmail: string;
  lineItems: Array<{
    name: string;
    description: string;
    rate: number;
    quantity: number;
  }>;
  discountAmount?: number;
  notes?: string;
  paymentGateway: 'Authorize.net' | 'Zoho Billing';
  transactionId: string;
}) {
  // Step 1: Create or get customer
  const customer = await getOrCreateCustomer({
    name: data.customerName,
    email: data.customerEmail,
  });

  // Step 2: Create invoice
  const invoicePayload = {
    customer_id: customer.contact_id,
    line_items: data.lineItems.map(item => ({
      name: item.name,
      description: item.description,
      rate: item.rate,
      quantity: item.quantity,
    })),
    discount: data.discountAmount || 0,
    notes: data.notes || '',
    payment_options: {
      payment_gateways: [
        {
          gateway_name: data.paymentGateway,
          configured: true,
        },
      ],
    },
  };

  const invoiceResponse = await axios.post(
    `${ZOHO_BOOKS_API}/invoices`,
    invoicePayload,
    {
      headers: {
        'Authorization': `Zoho-oauthtoken ${ZOHO_BOOKS_AUTH_TOKEN}`,
        'Content-Type': 'application/json',
      },
      params: {
        organization_id: ZOHO_BOOKS_ORG_ID,
      },
    }
  );

  const invoice = invoiceResponse.data.invoice;

  // Step 3: Record payment
  await recordPayment({
    invoiceId: invoice.invoice_id,
    amount: invoice.total,
    paymentMode: data.paymentGateway,
    reference: data.transactionId,
    date: new Date().toISOString().split('T')[0],
  });

  return invoice;
}

async function getOrCreateCustomer(data: {
  name: string;
  email: string;
}) {
  // Try to find existing customer
  const searchResponse = await axios.get(
    `${ZOHO_BOOKS_API}/contacts`,
    {
      headers: {
        'Authorization': `Zoho-oauthtoken ${ZOHO_BOOKS_AUTH_TOKEN}`,
      },
      params: {
        organization_id: ZOHO_BOOKS_ORG_ID,
        email: data.email,
      },
    }
  );

  if (searchResponse.data.contacts.length > 0) {
    return searchResponse.data.contacts[0];
  }

  // Create new customer
  const createResponse = await axios.post(
    `${ZOHO_BOOKS_API}/contacts`,
    {
      contact_name: data.name,
      contact_type: 'customer',
      email: data.email,
    },
    {
      headers: {
        'Authorization': `Zoho-oauthtoken ${ZOHO_BOOKS_AUTH_TOKEN}`,
        'Content-Type': 'application/json',
      },
      params: {
        organization_id: ZOHO_BOOKS_ORG_ID,
      },
    }
  );

  return createResponse.data.contact;
}

async function recordPayment(data: {
  invoiceId: string;
  amount: number;
  paymentMode: string;
  reference: string;
  date: string;
}) {
  await axios.post(
    `${ZOHO_BOOKS_API}/invoices/${data.invoiceId}/payments`,
    {
      amount: data.amount,
      payment_mode: data.paymentMode,
      reference_number: data.reference,
      date: data.date,
    },
    {
      headers: {
        'Authorization': `Zoho-oauthtoken ${ZOHO_BOOKS_AUTH_TOKEN}`,
        'Content-Type': 'application/json',
      },
      params: {
        organization_id: ZOHO_BOOKS_ORG_ID,
      },
    }
  );
}
```

---

## Discount Code System

### User Experience

**For Affiliates**:
1. Go to Affiliate Dashboard
2. Click "Create Discount Code"
3. Enter code (e.g., "WEBAGENCY20")
4. Select discount type (percentage or fixed amount)
5. Set discount value (10-30% range)
6. Choose which plans it applies to
7. Code is instantly active and trackable

**For Customers**:
1. Enter website URL on SafeOrbit homepage
2. See health score and issues
3. Click "Upgrade to Fix Issues"
4. Enter discount code at checkout
5. See discount applied in real-time
6. Complete payment
7. Affiliate earns commission automatically

### Implementation

#### Database Schema

```typescript
// drizzle/schema.ts

export const affiliates = mysqlTable('affiliates', {
  id: int('id').autoincrement().primaryKey(),
  userId: int('user_id').references(() => users.id),
  zohoThriveAffiliateId: varchar('zoho_thrive_affiliate_id', { length: 64 }),
  referralCode: varchar('referral_code', { length: 32 }).unique(),
  status: mysqlEnum('status', ['pending', 'active', 'suspended']).default('active'),
  totalReferrals: int('total_referrals').default(0),
  successfulConversions: int('successful_conversions').default(0),
  totalEarnings: int('total_earnings').default(0), // in cents
  pendingPayout: int('pending_payout').default(0), // in cents
  createdAt: timestamp('created_at').defaultNow(),
});

export const discountCodes = mysqlTable('discount_codes', {
  id: int('id').autoincrement().primaryKey(),
  affiliateId: int('affiliate_id').references(() => affiliates.id),
  code: varchar('code', { length: 32 }).unique().notNull(),
  discountType: mysqlEnum('discount_type', ['percentage', 'fixed']).notNull(),
  discountValue: int('discount_value').notNull(), // percentage or cents
  appliesTo: mysqlEnum('applies_to', ['first_month', 'first_year', 'all_payments']).default('first_month'),
  maxUses: int('max_uses'), // null = unlimited
  currentUses: int('current_uses').default(0),
  expiresAt: timestamp('expires_at'),
  status: mysqlEnum('status', ['active', 'expired', 'disabled']).default('active'),
  createdAt: timestamp('created_at').defaultNow(),
});

export const affiliateConversions = mysqlTable('affiliate_conversions', {
  id: int('id').autoincrement().primaryKey(),
  affiliateId: int('affiliate_id').references(() => affiliates.id),
  discountCodeId: int('discount_code_id').references(() => discountCodes.id),
  customerId: int('customer_id').references(() => users.id),
  subscriptionId: varchar('subscription_id', { length: 64 }),
  orderAmount: int('order_amount').notNull(), // in cents
  commissionAmount: int('commission_amount').notNull(), // in cents
  commissionRate: int('commission_rate').notNull(), // percentage
  status: mysqlEnum('status', ['pending', 'approved', 'paid']).default('pending'),
  zohoThriveTransactionId: varchar('zoho_thrive_transaction_id', { length: 64 }),
  createdAt: timestamp('created_at').defaultNow(),
});
```

#### tRPC Procedures

```typescript
// server/routers/affiliate.ts

import { router, protectedProcedure, publicProcedure } from '../_core/trpc';
import { z } from 'zod';

export const affiliateRouter = router({
  // Join affiliate program
  signup: protectedProcedure
    .input(z.object({
      company: z.string().optional(),
      phone: z.string().optional(),
    }))
    .mutation(async ({ input, ctx }) => {
      // Create affiliate in Zoho Thrive
      const zohoAffiliate = await createZohoThriveAffiliate({
        email: ctx.user.email,
        firstName: ctx.user.name?.split(' ')[0] || '',
        lastName: ctx.user.name?.split(' ').slice(1).join(' ') || '',
        company: input.company,
        phone: input.phone,
      });

      // Create affiliate record in SafeOrbit database
      const affiliate = await db.insert(affiliates).values({
        userId: ctx.user.id,
        zohoThriveAffiliateId: zohoAffiliate.affiliate_id,
        referralCode: generateReferralCode(ctx.user.name || ctx.user.email),
        status: 'active',
      });

      return {
        affiliateId: affiliate.id,
        referralLink: `https://safeorbit.com?ref=${affiliate.referralCode}`,
        status: 'active',
      };
    }),

  // Create discount code
  createDiscountCode: protectedProcedure
    .input(z.object({
      code: z.string().min(4).max(32).regex(/^[A-Z0-9]+$/),
      discountType: z.enum(['percentage', 'fixed']),
      discountValue: z.number().min(1).max(100),
      appliesTo: z.enum(['first_month', 'first_year', 'all_payments']),
      maxUses: z.number().optional(),
      expiresAt: z.date().optional(),
    }))
    .mutation(async ({ input, ctx }) => {
      // Get affiliate record
      const affiliate = await db.query.affiliates.findFirst({
        where: eq(affiliates.userId, ctx.user.id),
      });

      if (!affiliate) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'You must join the affiliate program first',
        });
      }

      // Check if code already exists
      const existing = await db.query.discountCodes.findFirst({
        where: eq(discountCodes.code, input.code),
      });

      if (existing) {
        throw new TRPCError({
          code: 'CONFLICT',
          message: 'This discount code is already taken',
        });
      }

      // Create code in Zoho Thrive
      await createZohoThriveDiscountCode({
        code: input.code,
        affiliateId: affiliate.zohoThriveAffiliateId,
        discountType: input.discountType,
        discountValue: input.discountValue,
        appliesTo: input.appliesTo,
      });

      // Create code in Zoho Billing
      await createZohoBillingCoupon({
        code: input.code,
        discountType: input.discountType,
        discountValue: input.discountValue,
        appliesTo: input.appliesTo,
      });

      // Save to SafeOrbit database
      const discountCode = await db.insert(discountCodes).values({
        affiliateId: affiliate.id,
        code: input.code,
        discountType: input.discountType,
        discountValue: input.discountValue,
        appliesTo: input.appliesTo,
        maxUses: input.maxUses,
        expiresAt: input.expiresAt,
        status: 'active',
      });

      return {
        id: discountCode.id,
        code: input.code,
        status: 'active',
        shareUrl: `https://safeorbit.com?code=${input.code}`,
      };
    }),

  // Get affiliate dashboard stats
  getStats: protectedProcedure
    .query(async ({ ctx }) => {
      const affiliate = await db.query.affiliates.findFirst({
        where: eq(affiliates.userId, ctx.user.id),
      });

      if (!affiliate) {
        return null;
      }

      // Fetch stats from Zoho Thrive
      const zohoStats = await getZohoThriveAffiliateStats(
        affiliate.zohoThriveAffiliateId
      );

      // Get discount codes
      const codes = await db.query.discountCodes.findMany({
        where: eq(discountCodes.affiliateId, affiliate.id),
      });

      // Get recent conversions
      const conversions = await db.query.affiliateConversions.findMany({
        where: eq(affiliateConversions.affiliateId, affiliate.id),
        orderBy: desc(affiliateConversions.createdAt),
        limit: 10,
      });

      return {
        referralLink: `https://safeorbit.com?ref=${affiliate.referralCode}`,
        totalReferrals: zohoStats.total_referrals,
        successfulConversions: zohoStats.successful_conversions,
        conversionRate: zohoStats.conversion_rate,
        totalEarnings: zohoStats.total_earnings,
        pendingPayout: zohoStats.pending_payout,
        discountCodes: codes,
        recentConversions: conversions,
      };
    }),

  // Validate discount code (public endpoint for checkout)
  validateCode: publicProcedure
    .input(z.object({
      code: z.string(),
      planCode: z.string(),
    }))
    .query(async ({ input }) => {
      const code = await db.query.discountCodes.findFirst({
        where: and(
          eq(discountCodes.code, input.code.toUpperCase()),
          eq(discountCodes.status, 'active')
        ),
      });

      if (!code) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'Invalid discount code',
        });
      }

      // Check if expired
      if (code.expiresAt && code.expiresAt < new Date()) {
        throw new TRPCError({
          code: 'BAD_REQUEST',
          message: 'This discount code has expired',
        });
      }

      // Check max uses
      if (code.maxUses && code.currentUses >= code.maxUses) {
        throw new TRPCError({
          code: 'BAD_REQUEST',
          message: 'This discount code has reached its usage limit',
        });
      }

      return {
        valid: true,
        discountType: code.discountType,
        discountValue: code.discountValue,
        appliesTo: code.appliesTo,
      };
    }),
});
```

---

## Affiliate Portal

### Features

**Dashboard**:
- Real-time earnings counter
- Conversion rate graph
- Recent referrals list
- Payment history

**Marketing Materials**:
- Shareable referral link
- Custom discount codes
- Pre-made banners and graphics
- Email templates
- Social media posts

**Performance Analytics**:
- Traffic sources
- Click-through rates
- Conversion funnel
- Top-performing codes

**Payout Management**:
- Current balance
- Payment threshold status
- Payout history
- Payment method setup (PayPal, bank transfer)

### UI Mockup

```
┌─────────────────────────────────────────────────────────────────┐
│  SafeOrbit Affiliate Dashboard                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Total        │  │ Pending      │  │ Conversion   │         │
│  │ Earnings     │  │ Payout       │  │ Rate         │         │
│  │              │  │              │  │              │         │
│  │  $1,234.56   │  │  $456.00     │  │  12.5%       │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                 │
│  Your Referral Link:                                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ https://safeorbit.com?ref=johndoe              [Copy]   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Your Discount Codes:                                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Code          | Discount | Uses  | Earnings | Actions   │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │ WEBAGENCY20   | 20% off  | 15/∞  | $234.50  | [Edit]    │   │
│  │ SAVE30        | 30% off  | 8/20  | $189.00  | [Edit]    │   │
│  │ NEWSITE10     | 10% off  | 3/∞   | $33.06   | [Edit]    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  [+ Create New Discount Code]                                   │
│                                                                 │
│  Recent Conversions:                                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Date       | Customer          | Plan    | Commission  │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │ Dec 1      | john@example.com  | Starter | $15.00      │   │
│  │ Nov 28     | jane@example.com  | Pro     | $49.00      │   │
│  │ Nov 25     | bob@example.com   | Starter | $15.00      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  [Request Payout] (Available when balance ≥ $50)                │
└─────────────────────────────────────────────────────────────────┘
```

---

## Commission Tracking

### Commission Structure

**Recommended Rates**:

| Plan | Monthly Price | Commission (First Month) | Recurring Commission |
|:-----|:--------------|:------------------------|:---------------------|
| Starter | $15/mo | $15 (100%) | $3/mo (20%) for 12 months |
| Professional | $49/mo | $49 (100%) | $9.80/mo (20%) for 12 months |
| Business | $149/mo | $149 (100%) | $29.80/mo (20%) for 12 months |

**Total Lifetime Value per Referral**:
- Starter: $15 + ($3 × 12) = **$51**
- Professional: $49 + ($9.80 × 12) = **$166.60**
- Business: $149 + ($29.80 × 12) = **$506.60**

### Payout Rules

**Minimum Payout**: $50  
**Payout Frequency**: Monthly (1st of each month)  
**Payment Methods**: PayPal, Bank Transfer, Wise  
**Cookie Duration**: 30 days  
**Attribution**: Last-click  

### Fraud Prevention

✅ **Email Verification**: Affiliates must verify email  
✅ **Manual Review**: First 3 conversions reviewed manually  
✅ **IP Tracking**: Detect self-referrals  
✅ **Chargeback Clawback**: Commission reversed if customer refunds  
✅ **Suspicious Pattern Detection**: Flag unusual activity  

---

## Implementation Roadmap

### Phase 1: Foundation (Week 1-2)

**Week 1: Setup & Configuration**
- [ ] Create Zoho Thrive account and configure brand
- [ ] Create Zoho Billing account and set up plans
- [ ] Create Zoho Books account and configure invoicing
- [ ] Create Authorize.net account and get API credentials
- [ ] Generate OAuth tokens for all Zoho services
- [ ] Store credentials securely in environment variables

**Week 2: Database & Backend**
- [ ] Add affiliate tables to database schema
- [ ] Implement affiliate signup procedure
- [ ] Implement discount code creation procedure
- [ ] Build Zoho Thrive integration service
- [ ] Build Zoho Billing integration service (Option A)
- [ ] Build Authorize.net integration service (Option B)
- [ ] Build Zoho Books integration service
- [ ] Implement webhook handlers

---

### Phase 2: Frontend & UX (Week 3-4)

**Week 3: Affiliate Portal**
- [ ] Create affiliate dashboard page
- [ ] Build discount code management UI
- [ ] Display earnings and stats
- [ ] Add referral link sharing
- [ ] Implement marketing materials download

**Week 4: Checkout Flow**
- [ ] Add discount code input to checkout
- [ ] Show discount applied in real-time
- [ ] Integrate with Zoho Billing checkout (Option A)
- [ ] Build custom checkout with Authorize.net (Option B)
- [ ] Add payment confirmation page
- [ ] Send receipt emails

---

### Phase 3: Testing & Launch (Week 5-6)

**Week 5: Testing**
- [ ] Test affiliate signup flow
- [ ] Test discount code creation and validation
- [ ] Test checkout with discount codes
- [ ] Test commission tracking in Zoho Thrive
- [ ] Test webhook handling
- [ ] Test invoice generation in Zoho Books
- [ ] Test payout requests

**Week 6: Launch**
- [ ] Create affiliate program landing page
- [ ] Write affiliate program terms and conditions
- [ ] Set up affiliate onboarding email sequence
- [ ] Launch beta with 10-20 affiliates
- [ ] Monitor for issues
- [ ] Public launch

---

## API Reference

### Zoho Thrive API

**Base URL**: `https://thrive.zoho.com/api/v1`  
**Authentication**: OAuth 2.0  
**Scope**: `Thrive.Referral.CREATE`

#### Create Affiliate

```http
POST /affiliates
Authorization: Bearer {oauth_token}
Content-Type: application/json

{
  "email": "affiliate@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "company": "Web Agency Inc",
  "phone": "+1234567890"
}
```

#### Push Purchase

```http
POST /thrive-publicapi/v1/{brand-code}/purchase
Authorization: Bearer {oauth_token}
Content-Type: application/json

{
  "email": "customer@example.com",
  "amount": 1500,
  "order_id": "sub_12345",
  "zt_customer_id": "cust_67890",
  "product_plan_name": "Starter Monthly"
}
```

#### Get Affiliate Stats

```http
GET /affiliates/{affiliate_id}/stats
Authorization: Bearer {oauth_token}
```

---

### Zoho Billing API

**Base URL**: `https://billing.zoho.com/api/v1`  
**Authentication**: OAuth 2.0  
**Headers**: `X-com-zoho-subscriptions-organizationid: {org_id}`

#### Create Subscription

```http
POST /subscriptions
Authorization: Zoho-oauthtoken {auth_token}
X-com-zoho-subscriptions-organizationid: {org_id}
Content-Type: application/json

{
  "customer": {
    "email": "customer@example.com",
    "display_name": "John Doe"
  },
  "plan": {
    "plan_code": "starter-monthly"
  },
  "coupon_code": "WEBAGENCY20",
  "starts_at": "2024-12-01"
}
```

#### Create Coupon

```http
POST /coupons
Authorization: Zoho-oauthtoken {auth_token}
X-com-zoho-subscriptions-organizationid: {org_id}
Content-Type: application/json

{
  "coupon_code": "WEBAGENCY20",
  "discount_type": "percentage",
  "discount": 20,
  "apply_to": "invoice",
  "coupon_type": "one_time"
}
```

---

### Authorize.net API

**Base URL**: `https://api.authorize.net/xml/v1/request.api` (production)  
**Test URL**: `https://apitest.authorize.net/xml/v1/request.api`

#### Charge Credit Card

```http
POST /xml/v1/request.api
Content-Type: application/json

{
  "createTransactionRequest": {
    "merchantAuthentication": {
      "name": "{api_login_id}",
      "transactionKey": "{transaction_key}"
    },
    "transactionRequest": {
      "transactionType": "authCaptureTransaction",
      "amount": "15.00",
      "payment": {
        "creditCard": {
          "cardNumber": "4111111111111111",
          "expirationDate": "1225",
          "cardCode": "123"
        }
      },
      "customer": {
        "email": "customer@example.com"
      }
    }
  }
}
```

---

### Zoho Books API

**Base URL**: `https://books.zoho.com/api/v3`  
**Authentication**: OAuth 2.0  
**Params**: `organization_id={org_id}`

#### Create Invoice

```http
POST /invoices?organization_id={org_id}
Authorization: Zoho-oauthtoken {auth_token}
Content-Type: application/json

{
  "customer_id": "12345",
  "line_items": [
    {
      "name": "SafeOrbit Starter Plan",
      "description": "Monthly subscription",
      "rate": 15.00,
      "quantity": 1
    }
  ],
  "discount": 3.00,
  "notes": "Thank you for your business!"
}
```

#### Record Payment

```http
POST /invoices/{invoice_id}/payments?organization_id={org_id}
Authorization: Zoho-oauthtoken {auth_token}
Content-Type: application/json

{
  "amount": 12.00,
  "payment_mode": "Authorize.net",
  "reference_number": "trans_12345",
  "date": "2024-12-01"
}
```

---

## Environment Variables

```bash
# Zoho Thrive
ZOHO_THRIVE_BRAND_CODE=your_brand_code
ZOHO_THRIVE_SECRET_KEY=your_secret_key
ZOHO_THRIVE_CLIENT_ID=your_client_id
ZOHO_THRIVE_CLIENT_SECRET=your_client_secret
ZOHO_THRIVE_OAUTH_TOKEN=your_oauth_token

# Zoho Billing
ZOHO_BILLING_ORG_ID=your_org_id
ZOHO_BILLING_AUTH_TOKEN=your_auth_token

# Zoho Books
ZOHO_BOOKS_ORG_ID=your_org_id
ZOHO_BOOKS_AUTH_TOKEN=your_auth_token

# Authorize.net
AUTHORIZE_NET_API_LOGIN_ID=your_api_login_id
AUTHORIZE_NET_TRANSACTION_KEY=your_transaction_key
AUTHORIZE_NET_ENVIRONMENT=sandbox # or production

# SafeOrbit
SAFEORBIT_BASE_URL=https://safeorbit.com
SAFEORBIT_AFFILIATE_COMMISSION_RATE=20 # percentage
SAFEORBIT_AFFILIATE_PAYOUT_THRESHOLD=5000 # cents ($50)
```

---

## Security Considerations

### API Keys & Secrets

✅ **Never expose** Zoho Thrive Secret Key to frontend  
✅ **Never expose** Authorize.net Transaction Key to frontend  
✅ **Use environment variables** for all credentials  
✅ **Rotate keys** every 90 days  
✅ **Use OAuth** instead of API keys when possible  

### Payment Security

✅ **PCI Compliance**: Use Authorize.net hosted payment forms or tokenization  
✅ **SSL/TLS**: All payment pages must use HTTPS  
✅ **Input Validation**: Validate all payment inputs on backend  
✅ **Fraud Detection**: Monitor for suspicious patterns  

### Affiliate Fraud Prevention

✅ **Email Verification**: Required for affiliate signup  
✅ **Manual Review**: First 3 conversions reviewed manually  
✅ **IP Tracking**: Detect self-referrals  
✅ **Cookie Tracking**: 30-day attribution window  
✅ **Chargeback Handling**: Reverse commission if customer refunds  

---

## Testing Checklist

### Affiliate Program

- [ ] User can sign up as affiliate
- [ ] Affiliate receives unique referral link
- [ ] Affiliate can create discount codes
- [ ] Discount codes are validated at checkout
- [ ] Commission is tracked in Zoho Thrive
- [ ] Affiliate dashboard shows correct stats
- [ ] Payout requests work correctly

### Payment Processing

- [ ] Customer can subscribe with credit card
- [ ] Discount codes apply correctly
- [ ] Payment is processed via Authorize.net
- [ ] Subscription is created in Zoho Billing (Option A)
- [ ] Invoice is generated in Zoho Books
- [ ] Receipt email is sent to customer
- [ ] Webhook updates subscription status

### Edge Cases

- [ ] Expired discount code shows error
- [ ] Invalid discount code shows error
- [ ] Max uses reached shows error
- [ ] Payment failure is handled gracefully
- [ ] Duplicate affiliate signup is prevented
- [ ] Self-referral is detected and blocked

---

**Document Version**: 1.0  
**Last Updated**: December 2024  
**Status**: Technical Specification
