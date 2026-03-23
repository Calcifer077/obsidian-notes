In this version, the UI that will accept the payment will be given to us by Stripe, our job is to send our user to that page with details of the purchase. The rest will be handled by Stripe on its own.

I have used `Next.js` for the below steps, can be changed according to your needs.
### Steps involved:
#### 1. Install Dependencies
```bash
npm install stripe @stripe/stripe-js
```
`stripe` -> Node.js SDK (Server side)
`@stripe/stripe.js` -> Client side
#### 2. Setup environment variables
```env
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_your_publishable_key
STRIPE_SECRET_KEY=sk_test_your_secret_key
```
The above keys are present in the dashboard. Never expose `STRIPE_SECRET_KEY`. Variables prefixed with `NEXT_PUBLIC_` are exposed to the browser.
#### 3. Create a stripe instance (client side)
```js
// lib/stripe.js
import { loadStripe } from '@stripe/stripe-js';

let stripePromise;

export const getStripe = () => {
  if (!stripePromise) {
    stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY);
  }
  return stripePromise;
};
```
#### 4. Create checkout session (api route)
`route.js` are special files in `Next`. They are created only for API purpose and you can call this file route anywhere in your app, and that particular method `(GET, POST)` will run. You can also traverse over some kind of array (say of products) when handling payments. `session` returns a lot of things what matters to us most in this case is `url`( A UI which the user can use to do payments.)

```js
// app/api/create-checkout-session/route.js

import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export async function POST(req) {
  const { bookingId, cabinPrice, numNights, cabinName } = await req.json();
  const origin = req.headers.get("origin");

  const session = await stripe.checkout.sessions.create({
    payment_method_types: ["card"],
    line_items: [
      {
        price_data: {
          currency: "usd",
          // 'product_data' will be shown in the dashboard.
          product_data: {
            name: `Cabin booking: ${cabinName}`,
            description: `${numNights} night(s) stay`,
          },
          unit_amount: cabinPrice * 100, // convert to cents
        },
        quantity: 1,
      },
    ],
    mode: "payment",
    success_url: `http://localhost:3000/success`,
    cancel_url: `http://localhost:3000/cancel`,
  });

  return Response.json({ url: session.url });
}
```
#### 5. Create checkout button or call the above route
You can create a button which user can click say `Buy now`. This button will carry the details of the product with it, and further down the line call the above route.

Instead we can also call above route directly in our component on form submission. Our business logic in this case requires us to create bookings first in the database. The user has the option to pay after coming on premises. That's why we first create a booking in the database and than redirect the user to a different page provided by stripe, which will handle the acceptance of payment. We are using `window.location.href` to redirect the user.
```jsx
"use client";

import { differenceInDays } from "date-fns";
import { useReservation } from "./ReservationContext";
import { createBooking } from "../_lib/actions";
import SubmitButton from "./SubmitButton";

function ReservationForm({ cabin, user }) {
  const { range, resetRange } = useReservation();
  const { maxCapacity, regularPrice, discount, id, name } = cabin;  

  const startDate = range?.from;
  const endDate = range?.to;

  const numNights = differenceInDays(endDate, startDate);
  const cabinPrice = numNights * (regularPrice - discount);
  
  const bookingData = {
    startDate,
    endDate,
    numNights,
    cabinPrice,
    cabinId: id,
  };

  // 'createBookingWithData' will be prepopulated with bookingData.
  const createBookingWithData = createBooking.bind(null, bookingData);  

  async function handleSubmit(formData) {
    const booking = await createBookingWithData(formData);  

    const res = await fetch("/api/create-checkout-session", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        bookingId: booking.id,
        cabinPrice,
        numNights,
        cabinName: name,
      }),
    });  

    const { url } = await res.json();
    
    window.location.href = url;
    
    resetRange();
  }

  return (
    // ...
      <form
        action={handleSubmit}
        className="bg-primary-900 py-10 px-16 text-lg flex gap-5 flex-col"
      >
        // ...
            <SubmitButton pendingLabel={"Reserving"}>Reserve Now</SubmitButton>
          )}
      </form>
    // ... 
  );
} 

export default ReservationForm;
```
#### 6. Add success and cancel page
Below are just pages which will be shown on success or cancelation. The route for these pages are defined in `route.js` above. Meaning it is not us who call these routes but stripe itself.
```jsx
// pages/success.js  OR  app/success/page.js
export default function SuccessPage() {
  return <h1>✅ Payment Successful! Thank you for your order.</h1>;
}

// pages/cancel.js  OR  app/cancel/page.js
export default function CancelPage() {
  return <h1>❌ Payment Cancelled. You have not been charged.</h1>;
}
```

#### Short notes
1. Install Dependencies
2. Setup environment variables
3. Create a stripe instance (client side)
4. Create checkout session (api route)
5. Create checkout button or call the above route
6. Add success and cancel page 
 
