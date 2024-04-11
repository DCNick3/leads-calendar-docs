# Leads Calendar

This documentation aims to help build the Leads Calendar application. The application is a calendar that allows users to
book paid appointments for individual and influencers.

## Functionality

The core flow of the functionality consists of the user finding an influencer, selecting a time slot, and paying for the
appointment. The app handles the payment processing and will create an event in the influencer's Google Calendar.

After delivering that, additional features can be implemented like:

- Analytics for the influencer
- Integration with other payment processors
- Integration with other calendar services

## Technical stack

The backend of the application will be written in Rust language, using the [`axum`](https://docs.rs/axum/latest/axum/)
web framework.
The application will use the [`sqlx`](https://docs.rs/sqlx/latest/sqlx/) crate for database operations. It also
integrates with three external APIs to provide its services:

- [Google Calendar API](https://developers.google.com/calendar)
- [Paypal API](https://developer.paypal.com/)
- [Binance Pay API](https://merchant.binance.com/en/)

For the frontend, the application will use the [`React`](https://reactjs.org/) library with
the [`Next.js`](https://nextjs.org/) framework.

It will use the [`tailwindcss`](https://tailwindcss.com/) utility-first CSS framework for
styling [MUI](https://mui.com/) components.
