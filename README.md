# Jumiä Marketplace — development build

An independent Kenyan multi-vendor marketplace starter built with Next.js 15, React 19 and Supabase. It is not affiliated with Jumia. Seller commission is configured at **10%**. The intended collection model is Option A: customer payments are collected by Jumiä's approved merchant account, seller net earnings are recorded in the database, and payouts are handled separately.

## Included in this build

- Customer email/password registration and sign-in using Supabase Auth.
- Server-side role checks for customer, seller and administrator areas. New signups are always customers; users cannot select their own privileged role.
- Seller application and administrator approval workflow. Approval grants seller access.
- Seller dashboard: product creation, JPG/PNG/WebP image upload, inventory quantity updates, order items, fulfilment status, courier/tracking details, seller balance and payout requests.
- Administrator dashboard: seller approvals, product moderation, order status management, M-Pesa payment reconciliation and payout review.
- Storefront reads active products from Supabase; customer cart creates an order through a server-side database function that checks stock and calculates 10% commission.
- M-Pesa Daraja STK Push request flow and callback event storage. Admin reconciliation queries Daraja and requires a successful callback with matching amount and receipt before marking an order paid.
- Customer order history and seller delivery tracking.
- Supabase Row Level Security policies and a private server-only service-role client.

## 1. Requirements

- Node.js 20 or later
- A Supabase project
- An approved Safaricom Daraja merchant setup for production payments
- A deployed HTTPS URL for Daraja callbacks

## 2. Install and run

```bash
npm install
cp .env.example .env.local
npm run dev
```

Open `http://localhost:3000`.

## 3. Configure Supabase

1. Create a Supabase project.
2. Open **SQL Editor** and run all of `supabase/schema.sql`.
3. In **Project Settings → API**, copy the Project URL and publishable key into `.env.local`.
4. Copy the service-role key into `SUPABASE_SERVICE_ROLE_KEY`. This key is server-only. Never add it to a `NEXT_PUBLIC_` variable or commit it.
5. In Supabase Auth, configure email confirmation and your allowed redirect URLs for local development and your production domain.
6. Create a customer account through `/auth`.
7. Bootstrap the first administrator manually in the SQL Editor, replacing the email with the administrator's account email:

```sql
update public.profiles
set role = 'admin'
where id = (select id from auth.users where email = 'ADMIN_EMAIL_HERE');
```

Only a trusted project owner should perform this bootstrap. Do not make public admin registration available.

## 4. Configure M-Pesa Daraja

1. Register the marketplace business with the appropriate legal operator and obtain Safaricom approval for the merchant product and permitted marketplace collection/settlement flow.
2. Create a Daraja application and test with sandbox credentials first.
3. Set `MPESA_ENV=sandbox` and fill the Daraja variables in `.env.local`.
4. Set `MPESA_CALLBACK_URL` to the public HTTPS callback URL, e.g. `https://YOUR_DOMAIN/api/mpesa/callback`.
5. Test the full STK flow and transaction query in sandbox before production approval.
6. Only switch to production after Safaricom approves the merchant account, credentials, callback URL, fund flow and settlement arrangements.

A Daraja STK request being accepted only means the prompt was initiated. The application does not treat that as payment. Callback data is stored as evidence; an administrator must run Daraja transaction reconciliation before the order is marked paid. Do not rely on a browser redirect or screenshot as proof of payment.

Official portal: https://developer.safaricom.co.ke/

## 5. Important operating and safety requirements

- If the project operator is under 18, a parent/guardian or legally eligible adult business operator must handle business registration, merchant onboarding, identity checks, bank/M-Pesa settlement setup, and any legal agreements. Do not bypass provider age or KYC requirements.
- Confirm with Safaricom and qualified legal/accounting support whether the proposed marketplace collection and seller settlement model is permitted for the chosen merchant product. A personal M-Pesa number or ordinary till should not be assumed to support marketplace settlement.
- Seller payouts in this build are manually reviewed and recorded by an administrator. No automated bank/M-Pesa payout API is enabled.
- The current order function reserves stock when an order is created. A production cancellation/expiry job must restore stock for abandoned or failed payments.
- Add rate limiting, bot protection, email verification, seller identity/business verification, audit logs, delivery integrations, notifications, refunds/disputes, tax/accounting records, privacy policy, terms, and support processes before launch.
- Add automated tests, security review, database backups, monitoring and a staging deployment before accepting real orders.

## 6. Useful routes

- `/` — marketplace storefront
- `/auth` — customer registration/sign-in
- `/account` — customer orders and delivery tracking
- `/sell` — seller application
- `/seller` — seller dashboard (after administrator approval)
- `/admin` — administrator dashboard

## 7. Environment variables

See `.env.example`. Never commit `.env.local`, credentials, passkeys, or service-role keys.
