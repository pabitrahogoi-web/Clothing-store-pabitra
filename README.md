# PABITRA Premium Clothing Store

A responsive React + TypeScript + Vite storefront starter with a fashion-editorial dark UI, product catalogue, filters, product detail pages, persistent cart, WhatsApp order-message builder, Canvas scroll-sequence component, and a Supabase-backed admin area.

## Honest launch status

This is a working application scaffold with local demo catalogue data. It is **not yet a live production store** because no Supabase project, WhatsApp business number, real inventory/product photography, shipping policy, or animation-frame ZIP has been supplied/configured. The UI labels demo content and does not claim WhatsApp requests are confirmed orders. Do not take live orders until you configure the backend and complete the launch checklist below.

## Requirements

- Node.js 20+
- npm
- Optional: a Supabase project for persistent catalogue/admin data

## Run locally

```bash
npm install
cp .env.example .env
npm run dev
```

Open the local URL Vite prints. Production build:

```bash
npm run build
npm run preview
```

## Configuration

1. Copy `.env.example` to `.env`.
2. Add `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` after applying the SQL migration below.
3. Set `VITE_WHATSAPP_NUMBER` to the business number in international digits-only format (example format: country code + number, no plus sign/spaces). Do not use a personal number unless intended for business.
4. Edit `src/config.ts` for brand announcement, Instagram, email, store address and map URL.
5. Replace sample products in `src/data.ts` with verified product data, photos, variants and stock, or publish them through the database after building out the complete admin forms.
6. Replace illustrative Unsplash campaign/category imagery with licensed PABITRA photography. Product cards explicitly treat the product images as demo imagery.

Vite `VITE_*` values are public to browsers. Never place Supabase service-role keys, payment secrets or other privileged credentials in `.env` variables prefixed with `VITE_`.

## Supabase setup

1. Create a Supabase project.
2. Run `supabase/migrations/001_initial_schema.sql` in the SQL editor or use the Supabase CLI.
3. Create an admin user in Supabase Auth.
4. Create that user's profile row if needed and promote it through a trusted project-owner SQL session:

```sql
insert into public.profiles (id, full_name, role)
values ('YOUR_AUTH_USER_UUID', 'Store Admin', 'admin')
on conflict (id) do update set role='admin';
```

Never allow public users to assign themselves the `admin` role. The migration's profile insert policy restricts self-created profiles to `customer`.

The admin route checks Supabase Auth and the RLS policy `public.is_admin()` protects privileged data. The initial admin UI currently supports authenticated overview, product list, order list/status changes and settings/inventory guidance. Full image upload/product CRUD, customer management, and inventory editing still need their forms and server-side flows completed before launch.

### Important order-processing gap

The storefront's current checkout opens a WhatsApp message and clearly states that it is only an order request. It does not persist an order row before opening WhatsApp yet. Do not represent WhatsApp clicks as confirmed orders. For production, add a trusted server-side Edge Function/RPC that:

- accepts product/variant IDs and quantities plus validated delivery details;
- reloads prices from `products` and checks availability from `product_variants`;
- atomically reserves/decrements inventory to prevent overselling;
- creates `orders` + `order_items` in one transaction with `pending_whatsapp_confirmation` status;
- returns a genuine order reference and a server-calculated total;
- marks confirmed only after an authorized confirmation action;
- handles cancellations/restocking and audit logs.

Do not permit direct anonymous writes to `orders` or `order_items`. The schema intentionally does not provide public write policies for private order/customer data.

## Scroll-controlled image sequence

The Canvas/GSAP ScrollTrigger section is implemented. No original frame ZIP was available in the current project input, so it does not fabricate substitute frames. To add the source sequence:

1. Extract the supplied ZIP into `public/sequence/`.
2. Preserve original filenames and numeric ordering. Ignore `__MACOSX`, `.DS_Store`, and unrelated files.
3. Edit `public/sequence/manifest.json` and list every valid frame filename in correct numeric order, e.g. `{ "frames": ["frame_0001.jpg", "frame_0002.jpg", "frame_0003.jpg"] }`.
4. Confirm every listed frame loads. Keep frame sizes optimized for mobile and desktop; hundreds of full-resolution frames can use substantial memory. For very large sequences, implement staged frame loading / downsampled variants before production.

The component loads the first frame first, loads the rest asynchronously, maps scroll progress deterministically to frame index, reverses when scrolling upward, uses Canvas drawing with a capped device-pixel ratio, and cleans up ScrollTrigger/listeners. If no frames are listed, it displays a setup message.

## Main routes

- `/` — homepage
- `/shop` — searchable/filterable catalogue
- `/product/:slug` — product detail and variant selection
- `/cart` — persistent cart and WhatsApp checkout form
- `/faq` — editable placeholder policy FAQ
- `/admin/login` — Supabase Auth login
- `/admin` — authenticated admin dashboard

## Before launch checklist

- [ ] Upload and inspect the actual animation ZIP, then build the numeric manifest.
- [ ] Replace all demo products, imagery, prices, sizes, colours and stock.
- [ ] Configure and verify WhatsApp number and contact channels.
- [ ] Finish product CRUD, secure image uploads, inventory edits and customer/order management.
- [ ] Implement the server-side order creation + atomic stock reservation flow described above.
- [ ] Add real shipping, returns, privacy, terms and cancellation policies.
- [ ] Configure public storage bucket policies for product images; never make private customer uploads public.
- [ ] Test Supabase RLS as anonymous, customer and admin users.
- [ ] Add abuse/rate limiting, input validation, logging, backups and monitoring.
- [ ] Test keyboard navigation, mobile layout, reduced motion, image loading and WhatsApp link behavior on real devices.
- [ ] Deploy to a host (Vercel, Cloudflare Pages or equivalent) and add domain/HTTPS.

## Notes

- No real address, reviews, discount claims, customer testimonials or live sales metrics are invented.
- Newsletter is intentionally disabled until a consent-aware email service is configured.
- Supabase browser integration uses only the anon key and RLS. Service-role credentials must remain server-side.

## Completion pass (17 October 2026)

Added legal-information draft routes: `/privacy-policy`, `/terms-and-conditions`, `/cookie-policy`, `/refund-policy`, `/about`, and `/contact`; a theme toggle; cookie preference notice; skip link; scroll progress; back-to-top control; print styles; favicon; `robots.txt`, `sitemap.xml`, `llms.txt`; and `LEGAL_AUDIT.md`.

**Not production-ready until configured:** replace the placeholder origin in `public/robots.txt` and `public/sitemap.xml`; provide verified seller/contact details and final policies; confirm third-party asset licences; configure and test Supabase Auth/RLS/storage and server-side order/stock processing; verify actual cookies and network requests; and run build, security, accessibility and production-hosting tests. The cookie notice is not a full CMP and does not by itself block trackers. Legal pages are drafts, not legal advice.
