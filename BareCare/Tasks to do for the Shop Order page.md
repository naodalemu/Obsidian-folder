- [x] Figure out how to make the project faster
- [ ] Figure out the first task fully and list it out below here

### Below are plan on how to implement the best way for the shop page
Here’s a concise plan to redesign `/admin/shop_orders` into a faster, fully dynamic Alpine.js page.

**Current state (key files)**
- View: order.blade.php (heavy server-rendered form, 5k-product select, jQuery).
- Controller: `AdminShopOrdersController` (adds products, promo/voucher handling, order creation).

**Pain points & causes**
- Slow/crashes: 5k options in `<select>` + repeated reloads; multiple jQuery DOM/full page reload flows.
- UX: No loading states; limited validation feedback; price logic partly on client.
- Requirements: All prices must come from backend; dynamic page with Alpine; loading indicator; readable errors; optional offline-ish barcode/pricing cache (research).

**Proposed approach (Alpine-first)**
1. **API-first refactor (Laravel)**
   - Add lightweight JSON endpoints for:
     - Barcode lookup + price, discount, stock (server-authoritative).
     - Product search (typeahead) with pagination.
     - Cart operations: add/update qty/discount/remove; apply/remove promo; apply/remove voucher.
     - Order submit.
   - Each returns `{data, errors, flash}` with human-readable messages; enforce price calculations server-side.
   - Keep existing routes but add `/admin/api/shop-orders/*` to avoid breaking current UI during transition.

2. **Alpine SPA-like page**
   - Replace form reloads with Alpine components:
     - **Scanner panel**: barcode input with debounce; on match -> add line; show inline error if unknown/invalid.
     - **Product search drawer**: async typeahead (no giant select); paginated results.
     - **Cart table**: rows bound to Alpine state; qty/discount edits send PATCH; totals pulled from server response.
     - **Promo/Voucher**: simple inputs with async validate/apply; show messages near fields.
     - **Order meta**: shop selector; invoice toggle with inline validation; submit button with loading.
   - **Loading & errors**: global spinner on API calls; per-control busy state; toast/inline messages for validation.
   - **No client-side price math**: after any change, use server response to refresh cart state (prices, totals).

3. **Performance**
   - Kill 5k-option select; use async search.
   - Keep DOM small; avoid jQuery; Alpine + `fetch`/Axios.
   - Debounce barcode input; lock button to prevent double submit.
   - Lazy-load product images or omit thumbnails if not needed.

4. **Validation UX**
   - Map backend validation messages to friendly text (e.g., “Invalid promo code”, “Barcode not found”, “Insufficient stock”).
   - Show field-level help under inputs.

5. **Receipts / card payment (other tasks)**
   - Prepare hooks in the UI to call local HTTP API (receipt) or write local file (card) after order success; gated by feature flags.

6. **Offline / local cache research (good-to-have)**
   - Since not on Postgres, consider browser-side KV/IndexedDB with periodic sync:
     - Options: Dexie.js (IndexedDB wrapper) or LocalForage for a local barcode → price cache.
     - Keep cache read-only; always re-validate with server before finalize to ensure price correctness.
     - Need a small “version”/ETag endpoint to refresh cache incrementally; viability depends on security policy (prices must still be confirmed server-side).

7. **Migration plan**
   - Phase 1: Add APIs returning same business logic as current controller.
   - Phase 2: Build new Alpine view behind a flag (`/admin/shop_orders2`) while keeping old page.
   - Phase 3: QA with barcode scanners; load tests; then switch routes.

**Questions for @Teodor Bonchev**
- Do we need line-item stock validation on add, or only on submit?
- Required fields for invoices (exact rules/regex for number)? Any localization rules?
- Should images be shown in cart, or can we drop them for speed?
- Expected barcode scan rate (to size debouncing and locking)?
- OK to introduce a small `/admin/api/shop-orders/*` namespace?

If you want, I can draft the new Alpine page skeleton (HTML + data bindings + loading/error states) and the Laravel API route/controller stubs next.

