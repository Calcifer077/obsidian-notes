In this version, the UI that will accept the payment will be given to us by Stripe, our job is to send our user to that page with details of the purchase. The rest will be handled by Stripe on its own.

I have used `Next.js`, `app router` for the below steps, can be changed according to your needs.
### Steps involved:
#### 1. Install Dependencies
```bash
npm install stripe @stripe/stripe-js
```
`stripe` -> Node.js SDK (Server side)
`@stripe/stripe.js` -> Client side
#### 2. Setup environment variables
```env
# .env
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
    metadata: { bookingId: String(bookingId) },
  });

  return Response.json({ url: session.url });
}
```
#### 5. Create checkout button or call the above route
You can create a button which user can click say `Buy now`. This button will carry the details of the product with it, and further down the line call the above route.

Instead we can also call above route directly in our component on form submission. Our business logic in this case requires us to create bookings first in the database. The user has the option to pay now or after coming on premises. That's why we first create a booking in the database and than redirect the user to a different page provided by stripe, which will handle the acceptance of payment. We are using `window.location.href` to redirect the user.
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
    await createBookingWithData(formData);
    
    resetRange();
  }

  async function handleSubmitWithPayment() {
    const form = document.querySelector("form");
    
    const formData = new FormData(); 
    
    formData.append("numGuests", form.elements.numGuests.value);
    formData.append("observations", form.elements.observations.value); 

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
            {/* The default action will be called on submission using this */}
              <SubmitButton pendingLabel={"Reserving"}>
                Pay Later (On premise)
              </SubmitButton>
              
              <SubmitButton
	          // By default button is of 'type='submit''. If we were to follow the default way both hanlders will be called on clicking any button. But we don't want that so that's why we made its 'type='button'', so that it will only be called when this button is clicked.
                type="button"
                onClick={handleSubmitWithPayment}
                pendingLabel={"Reserving"}
              >
                Reserve Now (Pay Online)
              </SubmitButton>
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
// app/success/page.js
export default function SuccessPage() {
  return <h1>✅ Payment Successful! Thank you for your order.</h1>;
}

// app/cancel/page.js
export default function CancelPage() {
  return <h1>❌ Payment Cancelled. You have not been charged.</h1>;
}
```

#### 7. Use webhooks ( response from stripe to our api )
Say you want to update your database after successful payment that's where you can use webhooks provided by stripe. After every successful payment / transaction it will call `api/webhook` on your application (only in production). Here you can get access to the `metadata` passed in the checkout session. In case of error on client side say insufficient balance, network problem, problems with card, they will be automatically handled by stripe in the checkout session.

Our use case make sure that we don't have to some extra work in case of payment failure. But if there was to be some extra work to be done, you can `switch` for a failed payment and delete that entry. Say some event like `payment_intent.payment_failed`, than you would have to delete that entry from your database according to your use case.

```js
// app/api/webhook/route.js

import { supabase } from "@/app/_lib/supabase";
import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);  

export async function POST(req) {
  const body = await req.text(); // App Router uses req.text()
  const sig = req.headers.get("stripe-signature");  

  let event;

  try {
    event = stripe.webhooks.constructEvent(
      body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET,
    );
  } catch (err) {
    console.error("Webhook error:", err.message);
    
    return new Response(`Webhook Error: ${err.message}`, { status: 400 });
  } 

  switch (event.type) {
    case "checkout.session.completed":
      const session = event.data.object; 
      console.log("✅ Payment received for session:", session.id);
      
      // This 'bookingId' is what we passed above in metadata
      const bookingId = session.metadata.bookingId;
  
      const { error } = await supabase
        .from("bookings")
        .update({ isPaid: true })
        .eq("id", bookingId);  

      if (error) {
	    // If we return response like this, stripe will recall the hook again with increasing time in between. 
        return new Response("DB update failed", { status: 400 });
      } 

      break;
    default:
      console.log(`Unhandled event type: ${event.type}`);
  } 

  return new Response(JSON.stringify({ received: true }), { status: 200 });
}
```
In local development, you want to test your hook without going through your UI steps, you can use cli tools by stripe. 
The easiest way to do this is:
1. Go to: [github.com/stripe/stripe-cli/releases/latest](https://github.com/stripe/stripe-cli/releases/latest)
2. Download `stripe_X.X.X_windows_x86_64.zip`
3. Extract the zip. you'll get `stripe.exe`
4. Move `stripe.exe` to a folder like `C:\stripe\`
5. Add it to **PATH**:
	- Search **"Environment Variables"** in Windows Start
	- Click **"Environment Variables"**
	- Under **System Variables** → find **Path** → click **Edit**
	- Click **New** → add `C:\stripe\`
	- Click **OK**

In your terminal:
```bash
stripe login
```
Will open a window in your browser.

Then:
```bash
stripe listen --forward-to localhost:3000/api/webhook
```
`localhost:3000/api/webhook` is basically the route that you want to hit in development.
Above will open a never terminating terminal which will show events happening regarding stripe, which will hit `localhost:3000/api/webhook`. When you run the above command for the first time, you will also be provided with a `secret key` which is used in the `api/webhook`.  It updated every time you restart this terminal.

To trigger a event:
```bash
stripe trigger checkout.session.completed
```

Above way won't have the `bookingId` or the `metadata`, for that you will have to use your UI.

In production, you will have a separate `secret key` for webhooks which is available in dashboard of your stripe account.

#### Short notes
1. Install Dependencies
2. Setup environment variables
3. Create a stripe instance (client side)
4. Create checkout session (api route)
5. Create checkout button or call the above route
6. Add success and cancel page 
7. Use webhooks ( response from stripe to our api )
