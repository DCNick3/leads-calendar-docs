# Obtaining tokens

All the APIs that the application uses require various tokens and secrets to work. This page breaks down each of the
API requirements and how to obtain them.

## Google Calendar API

To use the Google Calendar API, you need to create a project in
the [Google Cloud Console](https://console.cloud.google.com/) and enable the Google Calendar API. After creating the
project, you need to create OAuth 2.0 credentials. The credentials consist of a client ID and a
client secret. You can create these credentials by following the steps below:

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project.
3. Go to [APIs dashboard](https://console.cloud.google.com/apis/dashboard).
4. Click on "Enable APIs and Services".
5. Search for "Google Calendar API" and enable it. You will be returned to the dashboard.
6. Go to the "Credentials" section.
7. Click on "Create credentials" and select "OAuth client ID".
8. Select "Web application" as the application type.
9. Fill in the "Name" with "Awesome Leads Calendar".
10. Fill in the "Authorized redirect URIs" with `https://awesome-leads-calendar.com/auth/google/callback` (or another
    URI
    of your choice).
11. Click on "Create".
12. You will be shown the client ID and client secret. Copy these values and be ready to provide the client ID and
    client
    secret to the application.

## Paypal API

To use the Paypal API, you need to create a project in the [Paypal Developer Dashboard](https://developer.paypal.com/).
After creating the project, you need to create a REST API app. The app will provide you with a client ID and a client
secret.

1. Go to the [Paypal Developer Dashboard](https://developer.paypal.com/).
2. Create a new project.
3. Go to the "My Apps & Credentials" section.
4. Click on "Create App".
5. Fill in the "App Name" with "Awesome Leads Calendar".
6. Click on "Create App".
7. You will be shown the client ID and client secret. Copy these values and be ready to provide the client ID and client
   secret to the application.

## Binance Pay API

To use Binance Pay API you first need to sign the contract with
Binance. [This page](https://merchant.binance.com/en/onboarding) will lead you through the process. After signing the
contract, you will be provided with an API identity key issued by Binance payment system and a secret key. These keys
are required to authenticate your requests to the Binance Pay API and will need to be provided to the app.