# Google Calendar API

The [google_calendar](https://docs.rs/google-calendar/latest/google_calendar/) crate is used to communicate with the
Google Calendar API.

The flow consists of two parts: authenticating user and creating events in their calendar. The first part requires some
user interaction to obtain the necessary tokens. The second part is done by the backend in the background.

```rust no_run
use google_calendar::{Client, AccessToken};

pub async fn init_google_client(google_config: &crate::config::Google) -> anyhow::Result<Client> {
    let client = Client::new(
        google_config.client_id.clone(),
        google_config.client_secret.clone(),
        google_config.redirect_uri.clone(),
        "".to_string(),
        "".to_string()
    );

    Ok(client)
}

pub async fn request_user_auth(client: &Client) -> String {
    let auth_url = client.user_consent_url().await.unwrap();
    // Redirect the user to the auth_url
    auth_url
}

pub async fn handle_user_auth(client: &Client, code: &str, state: &str) -> anyhow::Result<AccessToken> {
    let access_token = client.get_access_token(code, state).await?;
    Ok(())
}

pub async fn create_event(client: &Client, token: &AccessToken, event_info: &EventInfo) -> anyhow::Result<()> {
    let event = client.events().insert(&event_info.calendar_id, 0, 0, false,
                                       SendUpdates::All, false, &event_info.event).await?;
    Ok(())
}
```