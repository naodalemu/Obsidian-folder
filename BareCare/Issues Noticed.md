#### First thing you need to check is understand the old code, what it does, how it does it. The whole controller. And compare it with the new one manually.
- [x] The address is not being added when an order is sent. It's going empty
- [ ] When no cart items, the “last 10 orders” table is no longer shown. 🔄️🔄️🔄️
- [x] Client-side invoice-field validation/required toggling (jQuery) was removed; now only server validation remains.
- [x] Order creation no longer saves shipping/client fields (shipping_method, shipping_*), return_status, order_type, or wc_id, nor price_products/price_promocode_discounted/promocode_code.
- [x] Order line items no longer store base_price, additional_discount, promocode_code/discount; only price and total_price are saved.
- [x] Inventory update no longer creates a shop inventory row if missing; it only decrements existing rows.


Code that might help fix issues, recommended from Co-pilot:
```
<?php
// ...existing code...
    public function create(Request $request)
    {
        $request->validate([
            'shop' => 'required|string',
            'company_name' => 'nullable|string|min:2|max:255',
            'company_invoice_number' => 'nullable|string|min:2|max:255|regex:/^\S+$/',
            'company_address' => 'nullable|string|min:2|max:255',
        ]);

        // dd($request); // remove this

        $cart = $this->getCartSummary();

        abort_if(empty($cart['items']), 400, 'Empty cart');

        $voucher = session('voucher')
            ? Voucher::usable(session('voucher'))->first()
            : null;

        DB::transaction(function () use ($request, $cart, $voucher) {
            $order = $this->saveOrder($cart, $request, $voucher);
            $this->saveOrderProducts($cart['items'], $order, $request->shop);
        });

        session()->forget(['cart_products', 'promocode', 'voucher']);

        return redirect()->route('admin.order')
            ->with('message', 'Order created successfully');
    }
// ...existing code...

    private function saveOrder(array $cart, Request $request, ?Voucher $voucher): Order
    {
        $order = new Order();

        $promoCode = session('promocode');

        // Derive totals similar to old code
        $priceProducts = 0;
        $promoDiscount = 0;
        $adminDiscountSum = 0;

        foreach ($cart['items'] as $item) {
            $priceProducts += $item['qty'] * $item['shop_price'];
            $adminDiscountSum += ($item['admin_discount'] ?? 0);
            // promo per unit = (shop_price - admin_discount) - after_discount
            $promoPerUnit = max(($item['shop_price'] - ($item['admin_discount'] ?? 0)) - $item['after_discount'], 0);
            $promoDiscount += $promoPerUnit * $item['qty'];
        }

        $order->ref = OrderService::generateOrderRef();
        $order->status = Order::ORDER_STATUS_NEW;
        $order->payment_status = Order::ORDER_PAYMENT_STATUS_PAID;
        $order->order_from = Order::ORDER_FROM_SHOP;

        // Ensure shop is persisted
        $order->shop = $request->shop;

        // Restore commonly used order meta
        $order->payment_method = Order::ORDER_PAYMENT_METHOD_STANDARD;
        $order->client_note = '';
        $order->shipping_note = '';
        $order->shipping_method = 5;
        $order->shipping_first_name = \App\Constants\ShopConstants::SHOP_LABELS[$request->shop] ?? '';
        $order->shipping_last_name = '';
        $order->shipping_phone = '';
        $order->shipping_city = 1;
        $order->shipping_city_id = null;

        // Prices
        $order->price_products = round($priceProducts, 2);
        $order->price_promocode_discounted = round($promoDiscount, 2);
        $order->price_shipping = 0;
        $order->price_total_payable = $cart['total'];

        // Promo on order level (as before)
        $order->promocode_code = $promoCode ?? null;

        // Order type logic (old behavior)
        $order->order_type = $adminDiscountSum
            ? \App\Models\Order::ORDER_TYPE_THE_MALL_PLUS
            : \App\Constants\ShopConstants::SHOP_ORDER_TYPES[$request->shop] ?? null;

        // Identity
        $order->email = Auth::user()->email ?? 'office@barecare.bg';
        $order->user_id = null;

        // Invoice fields
        $order->wants_invoice = $request->boolean('wants_invoice');
        $order->company_name = $request->company_name;
        $order->company_invoice_number = $request->company_invoice_number;
        $order->company_address = $request->company_address;

        $order->save();

        if ($voucher) {
            (new OrderService())->saveVoucherInfo($order, $voucher);
        }

        return $order;
    }

    private function saveOrderProducts(array $items, Order $order, string $shop): void
    {
        $promoCode = session('promocode');

        foreach ($items as $item) {
            $variation = ProductVariation::withTrashed()->find($item['variant_id']);

            // Insert line with richer details
            DB::table('orders_products')->insert([
                'order_id' => $order->id,
                'variant_id' => $item['variant_id'],
                'qty' => $item['qty'],
                'base_price' => $variation->base_price ?? null,
                'price' => $item['after_discount'],
                'additional_discount' => $item['admin_discount'] ?? 0,
                'promocode_code' => $promoCode ?? null,
                // promo per unit = (shop_price - admin_discount) - after_discount
                'promocode_discount' => max(($item['shop_price'] - ($item['admin_discount'] ?? 0)) - $item['after_discount'], 0),
                'total_price' => $item['qty'] * $item['after_discount'],
            ]);

            // Decrement or create inventory row
            $inv = ProductInventoryShop::where('variant_id', $item['variant_id'])
                ->where('shop', $shop)
                ->first();

            if ($inv) {
                $inv->decrement('qty', $item['qty']);
            } else {
                $inv = new ProductInventoryShop();
                $inv->variant_id = $item['variant_id'];
                $inv->shop = $shop;
                $inv->qty = 0 - $item['qty'];
                $inv->save();
            }
        }
    }
// ...existing code...
```


Old Look
![[Pasted image 20251218154345.png]]
![[Pasted image 20251218155949.png]]


### More issues noticed after the full ajax situation is implemented
- [x] The clear order button is not working
- [ ] 
- [x] Promo Code is failing
- [x] Voucher Code is failing
- [x] Figure out the gap at the top between the title and the rest of the page
- [x] There is no confirmation or error messages being sent
- [x] Add loading to ajax requests.
- [x] Add a price calculation at the top right corner when there is a discount


### Tasks to do now
- [ ] When the last item is removed it doesn't fully remove the collection and it creates an issue to use together with the old code ⚠️ (ignored) ⚠️

### Before PR checklist
- [x] Change the language as it was
- [x] Remove comments all around the page
- [x] Look for flakey code, clumsy unmaintainable flow
- [x] Ask about the two tasks that say (other task) on jira

### Github Changes that made issues
#### Old:
```
public function getPaymentMethodLabel()

{
return self::ORDER_PAYMENT_METHOD_PAID_ARR[$this->payment_method] ?? self::ORDER_PAYMENT_STATUS_PAID_ARR[self::ORDER_PAYMENT_STATUS_NOT_PAID];

}

public function getPaymentStatusLabel(): string

{

if ($this->isCardPayment()) {

return $this->payment ? $this->payment->getPaymentStatusLabel() : "Unknown";

}

return self::ORDER_PAYMENT_STATUS_PAID_ARR[$this->payment_status] ?? self::ORDER_PAYMENT_STATUS_PAID_ARR[self::ORDER_PAYMENT_STATUS_NOT_PAID];

}
```

#### New:
```
    public function getPaymentMethodLabel()

    {

        return __(self::ORDER_PAYMENT_METHOD_PAID_ARR[$this->payment_method]) ?? __(self::ORDER_PAYMENT_STATUS_PAID_ARR[self::ORDER_PAYMENT_STATUS_NOT_PAID]);

    }
    
    public function getPaymentStatusLabel(): string

    {

        if ($this->isCardPayment()) {

            return $this->payment ? $this->payment->getPaymentStatusLabel() : __(self::ORDER_PAYMENT_STATUS_PAID_ARR['unknown']);

        }

  

        return __(self::ORDER_PAYMENT_STATUS_PAID_ARR[$this->payment_status]) ?? __(self::ORDER_PAYMENT_STATUS_PAID_ARR[self::ORDER_PAYMENT_STATUS_NOT_PAID]);

    }
```
