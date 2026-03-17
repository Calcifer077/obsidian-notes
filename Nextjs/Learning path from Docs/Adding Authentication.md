#### What is authentication?
Authentication is a key part of many web applications today. It's how a system checks if the user is who they say they are.

#### Authentication vs. Authorization
- **Authentication** is about making sure the user is who they say they are. You're proving your identity with something you have like a username and password.
- **Authorization** is the next step. Once a user's identity is confirmed, authorization decides what parts of the application they are allowed to use.

We will be using [NextAuth.js](https://nextjs.authjs.dev/) to add authentication to your application.

#### Setting up NextAuth.js
Install NextAuth.js and get a random key.

Create a `auth.config.ts`
```ts
// Basic config file, only logged in user can access dashboard
import type { NextAuthConfig } from "next-auth";

export const authConfig = {
  // Which page to be shown on 'signIn'
  pages: {
    signIn: "/login",
  },
  callbacks: {
	// It just says that only logged in user can access dashboard and if the user is logged in he can't access login page and can only access dashboard.
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user;
      const isOnDashboard = nextUrl.pathname.startsWith("/dashboard");
      if (isOnDashboard) {
        if (isLoggedIn) return true;
        return false; // Redirect unauthenticated users to login page
      } else if (isLoggedIn) {
        return Response.redirect(new URL("/dashboard", nextUrl));
      }
      return true;
    },
  },
  providers: [], // Add providers with an empty array for now

} satisfies NextAuthConfig;
```
I have added all the code in one go, but you can go through official documentation for step by step.

#### Protecting your routes with Next.js Proxy
This file runs before every request to the server. Kind of like a middleware but with different name. The advantage of employing Proxy for this task is that the protected routes will not even start rendering until the Proxy verifies the authentication, enhancing both the security and performance of your application.
Is created at the root of your app.
```ts
// It basically checks if the user is logged in or not. If the user is logged in he can go to dashboard, else he will be redirected to login page without ever seeing dashboard page.
// The above condition is done in line 'export default NextAuth(authConfig).auth;'
// It just applies the rules written in 'auth.config.ts'

import NextAuth from "next-auth";
import { authConfig } from "./auth.config";

export default NextAuth(authConfig).auth;

export const config = {
  // https://nextjs.org/docs/app/api-reference/file-conventions/proxy#matcher
  // Run the guard on every page except static, images. It is done so that it won't slow down html, css and js files.
  matcher: ["/((?!api|_next/static|_next/image|.*\\.png$).*)"],
};
```

#### Adding the real auth logic
We will create a `auth.ts` at the root of our project which is responsible for the real auth logic.
```ts
// the real auth logic
  
import NextAuth from "next-auth";
// the login-with-email-password functionality
import Credentials from "next-auth/providers/credentials";
import { authConfig } from "./auth.config";
import { z } from "zod";
import type { User } from "@/app/lib/definitions";
import bcrypt from "bcrypt";
import postgres from "postgres";
  
const sql = postgres(process.env.POSTGRES_URL!, { ssl: "require" });
  
// gets user from the database
async function getUser(email: string): Promise<User | undefined> {
  try {
    const user = await sql<User[]>`SELECT * FROM users WHERE email=${email}`;
    return user[0];
  } catch (error) {
    console.error("Failed to fetch user:", error);
    throw new Error("Failed to fetch user.");
  }
}
  
export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  // 'providers' 'Credentials' means that we are using username(email) and password here.
  providers: [
    Credentials({
      async authorize(credentials) {
        // data from the form and validating.
        const parsedCredentials = z
          .object({ email: z.string().email(), password: z.string().min(6) })
          .safeParse(credentials);
  
        if (parsedCredentials.success) {
          const { email, password } = parsedCredentials.data;
          const user = await getUser(email);
          if (!user) return null;
  
          // As when we are storing passwords in database, we encrypt them that's why when checking for match we use bcrypt to compare them.
          const passwordMatch = await bcrypt.compare(password, user.password);
  
          if (passwordMatch) return user;
        } 
        console.log("Invalid credentials");
        return null;
      },
    }),
  ],
});
```

#### In our actions file
```ts
'use server';
 
import { signIn } from '@/auth';
import { AuthError } from 'next-auth';
 
// ...
 
export async function authenticate(
  prevState: string | undefined,
  formData: FormData,
) {
  try {
    await signIn('credentials', formData);
  } catch (error) {
    if (error instanceof AuthError) {
      switch (error.type) {
        case 'CredentialsSignin':
          return 'Invalid credentials.';
        default:
          return 'Something went wrong.';
      }
    }
    throw error;
  }
}
```
You can use the same way of calling the function as done in [Using ZOD for validation](Using%20ZOD%20for%20validation.md) to access the above function. When you call `signIn` `auth.ts` will be invoked and used.
You can also create a similar function in action file for `logout` but lets use a function in the component itself.
```tsx
import { signOut } from '@/auth';
 
export default function SideNav() {
  return (
    // ...
        <form
          action={async () => {
            'use server';
            await signOut({ redirectTo: '/' });
          }}
        />
    // ...
  );
}
```

`auth.config.ts` -> where you config `NextAuth.js`
`proxy.ts` -> the file that runs before every request to our server and stops rendering (protected routes) if the user is not logged in.
`auth.ts` -> file where your real logic resides.