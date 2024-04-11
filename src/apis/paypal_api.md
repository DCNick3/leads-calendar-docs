# Paypal API

The [paypal_rs](https://docs.rs/paypal-rs/latest/paypal_rs/) crate is used to communicate with the PayPal API.

The core object of the PayPal API is the Order. The payment flow is the following:

1. The application create an order with the desired amount and currency with PayPal API
2. The user is redirected to the PayPal website to complete the payment.
3. The user completes the payment.
4. The application captures the payment and completes the order with PayPal API.

```rust no_run
use paypal_rs::{
    Client,
    api::orders::*,
    data::orders::*,
    data::common::Currency,
    PaypalEnv,
};

/// Create a PayPal API client from config
pub async fn init_paypal_client(paypal_config: &crate::config::Paypal) -> anyhow::Result<Client> {
    let client = Client::new(
        paypal_config.client_id.clone(),
        paypal_config.client_secret.clone(),
        PaypalEnv::Sandbox,
    );

    client.get_access_token().await?;

    Ok(client)
}

/// Creates a 1-dollar order for a booking
pub async fn create_order(paypal_client: &Client, context: ApplicationContext) -> anyhow::Result<PaypalOrder> {
    let order = OrderPayloadBuilder::default()
        .intent(Intent::Capture)
        .purchase_units(vec![PurchaseUnit::new(Amount::new(Currency::USD, "1.0"))])
        .application_context(context);

    let create_order = CreateOrder::new(order);

    let order_created = paypal_client.execute(&create_order).await?;

    Ok(PaypalOrder::new(order_created))
}

pub async fn capture_order(paypal_client: &Client, order_id: &str) -> anyhow::Result<PaypalOrder> {
    let capture_order = CaptureOrder::new(order_id);

    let order_captured = paypal_client.execute(&capture_order).await?;

    Ok(PaypalOrder::new(order_captured))
}
```