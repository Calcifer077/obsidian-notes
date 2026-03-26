Let's imagine a scenario that someone booked a room from our application or bought something. It is possible that the user cancelled their reservation or returned our product. In such cases we need to refund the amount paid to us. It can be easily done using `stripe`. As we have already accepted payments in [Stripe Checkouts, How to accept basic payments](Stripe%20Checkouts,%20How%20to%20accept%20basic%20payments.md). 

First few steps like installation, getting payments are defined in [Stripe Checkouts, How to accept basic payments](Stripe%20Checkouts,%20How%20to%20accept%20basic%20payments.md). We have to follow below steps:
1. We call stripe refund api with payment intent ID
2. Stripe refund the money back to the customer's card
3. We update the database, delete the entry, set `paid` to false

#### 1. First we need `payment_intent`, so update our webhook.
```js
// app/api/webhook/route.js
case "checkout.session.completed": {
  const session = event.data.object;
  const bookingId = session.metadata?.bookingId;

  const { error } = await supabase
    .from("bookings")
    .update({
      isPaid: true,
      stripePaymentIntentId: session.payment_intent, // this
    })
    .eq("id", Number(bookingId));

  if (error) {
    console.error("Supabase update failed:", error.message);
    return new Response("DB update failed", { status: 400 });
  }

  break;
}
```

#### 2. Create a refund api route
```js
// app/api/refund/route.js

import Stripe from "stripe";
import { createClient } from "@/utils/supabase/server";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export async function POST(req) {
  const { bookingId } = await req.json();
  const supabase = createClient();

  // 1. Get the booking from Supabase
  const { data: booking, error: fetchError } = await supabase
    .from("bookings")
    .select("*")
    .eq("id", bookingId)
    .single();

  if (fetchError || !booking) {
    return Response.json({ error: "Booking not found" }, { status: 404 });
  }

  // 2. If not paid online, just delete — no refund needed
  if (!booking.isPaid || !booking.stripePaymentIntentId) {
    await supabase.from("bookings").delete().eq("id", bookingId);
    return Response.json({ message: "Booking deleted, no refund needed" });
  }

  // 3. Issue Stripe refund
  try {
    const refund = await stripe.refunds.create({
      payment_intent: booking.stripePaymentIntentId, // refund using the payment_intent which we got in the above step
    });

    console.log("Refund issued:", refund.id);
  } catch (err) {
    console.error("Stripe refund failed:", err.message);
    return Response.json({ error: "Refund failed" }, { status: 500 });
  }

  // 4. Delete or update booking in Supabase
  const { error: deleteError } = await supabase
    .from("bookings")
    .delete()
    .eq("id", bookingId);

  if (deleteError) {
    console.error("Booking deletion failed:", deleteError.message);
    return Response.json({ error: "Booking deletion failed" }, { status: 500 });
  }

  return Response.json({ message: "Booking cancelled and refund issued" });
}
```

#### 3. Call the Refund API route when user tries to delete
```js
// wherever our delete button is
async function handleDeleteReservation(bookingId) {
  const res = await fetch("/api/refund", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ bookingId }),
  });

  const data = await res.json();

  if (!res.ok) {
    alert("Failed to cancel: " + data.error);
    return;
  }

  alert(data.message); // "Booking cancelled and refund issued"
}
```

#### Testing in DEV mode
In one terminal run `stripe listen --forward-to localhost:3000/api/webhook`. Using previous command whatever event happens regarding stripe will be shown in terminal.

In my case I have done it in server action itself.
```js
'use server';

import Stripe from "stripe";
import { revalidatePath } from "next/cache";
import { auth } from "./auth";
import { supabase } from "./supabase";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export async function deleteBooking(bookingId) {
  // Check if the user is authenticated
  const session = await auth(); 

  if (!session)
    throw new Error("You must be logged in to perform this action.");

  const guestBookings = await getBookings(session.user.guestId);
  const guestBookingIds = guestBookings.map((booking) => booking.id);

  if (!guestBookingIds.includes(bookingId))
    throw new Error("You are not allowed to delete this booking");  

  const { data: booking, error: fetchError } = await supabase
    .from("bookings")
    .select("*")
    .eq("id", bookingId)
    .single(); 

  if (fetchError || !booking) {
    throw new Error("Booking not found.");
  } 

  if (!booking.isPaid || !booking.stripePaymentIntentId) {
    await supabase.from("bookings").delete().eq("id", bookingId); 

    console.log("No payment for this booking");
    return;
  } 

  try {
    const refund = await stripe.refunds.create({
      payment_intent: booking.stripePaymentIntentId,
    });

    console.log("Refund issued", refund.id);
  } catch (err) {
    throw new Error("Refund failed");
  }  

  const { error } = await supabase
    .from("bookings")
    .delete()
    .eq("id", bookingId); 

  if (error) throw new Error("Reservation could not be deleted.");  

  revalidatePath("/account/reservations");
}
```

Short steps:
1. Get payment intent ID after accepting payment
2. Create a refund api route
3. Call this route

