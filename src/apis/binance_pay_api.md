# Binance Pay API

The [binance-pay-rs](https://docs.rs/binance-pay-rs/latest/bpay/) crate is used to communicate with the Binance Pay API.

The flow is very similar to the [PayPal API](./paypal_api.md):

- The application creates an order with the desired amount and currency with the Binance Pay API
- The user is redirected to the Binance Pay website to complete the payment.
- The user completes the payment.
- The application confirms the payment status either via API or by receiving a webhook.

```rust no_run
use bpay::api::order::create::{
    Currency, Env, Goods, GoodsCategory, GoodsType, Request as OrderRequest, TerminalType,
};
use bpay::client::Client;
use bpay::utils::create_nonce;

/// Create a PayPal API client from config
pub async fn init_bpay_client(bpay_config: &crate::config::BinancePay) -> anyhow::Result<Client> {
    let client = Client::new(
        bpay_config.api_key.clone(),
        bpay_config.secret.clone(),
    );

    Ok(client)
}

/// Creates a 1-dollar order for a booking
pub async fn create_order(bpay_client: &Client, context: ApplicationContext) -> anyhow::Result<BinancePayOrder> {
    let order = OrderRequest {
        merchant_id: "your_merchant_id".to_string(),
        terminal_id: "your_terminal_id".to_string(),
        nonce: create_nonce(),
        currency: Currency::USD,
        total_amount: 1.0,
        subject: "Booking".to_string(),
        goods: Goods {
            goods_id: "your_goods_id".to_string(),
            goods_name: "Booking".to_string(),
            goods_num: 1,
            goods_category: GoodsCategory::Digital,
            body: "Booking".to_string(),
            show_url: "https://your-website.com".to_string(),
            goods_type: GoodsType::Virtual,
        },
        terminal_type: TerminalType::WEB,
        notify_url: "https://your-website.com/bpay/notify".to_string(),
        return_url: "https://your-website.com/bpay/return".to_string(),
    };

    let order_created = bpay_client.create_order(&order).await?;

    Ok(BinancePayOrder::new(order_created))
}

pub async fn query_order(bpay_client: &Client, order_id: &str) -> anyhow::Result<BinancePayOrder> {
    let order_query = OrderQuery {
        merchant_id: "your_merchant_id".to_string(),
        terminal_id: "your_terminal_id".to_string(),
        order_id: order_id.to_string(),
        nonce: create_nonce(),
    };

    let order_queried = bpay_client.query_order(&order_query).await?;

    Ok(BinancePayOrder::new(order_queried))
}
```