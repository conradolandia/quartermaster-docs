1. Stripe utility functions in `app/core/stripe.py`:
   - `create_payment_intent`: Creates a new payment intent
   - `retrieve_payment_intent`: Retrieves a payment intent by ID
   - `refund_payment`: Processes refunds

2. Payment endpoints in `app/api/routes/payments.py`:
   - `POST /payments/create-payment-intent`: Creates a new payment intent
   - `POST /payments/verify-payment/{payment_intent_id}`: Verifies payment status
   - `POST /payments/webhook`: Handles Stripe webhook events

3. Updated booking creation in `app/api/routes/bookings.py`:
   - Creates a payment intent before creating the booking
   - Sets initial booking status to `pending_payment`
   - Associates the payment intent ID with the booking

To complete the integration, you'll need to:

1. Add your Stripe webhook secret to your `.env` file:
```
STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret
```

2. Set up a webhook endpoint in your Stripe dashboard:
   - Go to Developers > Webhooks
   - Add endpoint: `https://your-domain.com/api/v1/payments/webhook`
   - Select events to listen for:
     - `payment_intent.succeeded`
     - `payment_intent.payment_failed`

3. Update your frontend to:
   - Use the payment intent client secret to initialize Stripe Elements
   - Handle payment confirmation
   - Show appropriate success/error messages
