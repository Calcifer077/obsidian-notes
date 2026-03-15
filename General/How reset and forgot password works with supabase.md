First you need to create pages for forgot password. This page will accept user email to which supabase will send a email. After user submits the form responsible for forgot password we will send a request to the backend (can used tanstack, simple fetch or something else that you like.). I have used tanstack for mutations and a api layer based on axios.
`tanstack hook`
```js
import { useMutation } from "@tanstack/react-query";

import { forgotPassword as forgotPasswordApi } from "../../services/apiAuthentication";

import toast from "react-hot-toast";  

export function useForgotPassword() {

  const {
    mutateAsync: forgotPassword,
    isLoading,
    isError,
    error,
  } = useMutation({
    mutationFn: forgotPasswordApi,
    onSuccess: () => {
      toast.success("A email with reset password link has been sent to you");
    },
    onError: (err) => {
      toast.error(err.response.data.message);
    },
  }); 

  return { forgotPassword, isLoading, isError, error };
}
```
 Api layer
 ```js
 export const forgotPassword = async function (email) {
 
  const res = await AxiosInstance({
    url: "/users/forgotPassword",
    method: "post",
    data: {
      email,
    },
  });

  if (res.data.status === "success") {
    toast.success("A email with reset password link has been sent to you");
  }

  return null;
};
 ```

In the backend, server will listen to this request and call functions provided by supabase which will send the email.
```js
export const forgotPassword = catchAsync(async function (req, res, next) {

  const email = req.body.email;

  const { error } = await supabase.auth.resetPasswordForEmail(email, {
    redirectTo: "http://localhost:5173/reset-password", // Must match added redirect URL
  });

  if (error) {
    // Don't reveal if email exists → always 200 OK in production
    return res.status(200).json({
      status: "success",
      message: {
        message: "Reset email sent if account exists",
      },
    });
  } 

  return res.status(200).json({
    status: "success",
    message: {
      message: "Reset email sent if account exists",
    },
  });
});
```
You have to set the `http://localhost:5173/reset-password` in your supabase email section so supabase can send it. You can also define custom templates.

Supabase own SMTP limits is quite low so you have to setup your own custom SMTP which will require you to buy a domain (DNS is needed for sending domains by resend, mailgun and much more). Instead of buying a domain you can use your gmail for the same stuff. Just create SMTP under app password and copy the password you are given. Paste this password, email to supabase custom SMTP dashboard and gmail will handle sending mails for you.

Now comes resetting password.

Resetting password requires you to setup a session. To create a session you need `access_token` and `refresh_token`. When user receives a email for resetting password the URL embedded in the reset button will have all the above things and you have to extract them from URL.

In the frontend page responsible for resetting password
```jsx
const hash = new URLSearchParams(window.location.hash.substring(1));
const access_token = hash.get("access_token");
const refresh_token = hash.get("refresh_token");
```
You have to send the above two tokens to backend. As tanstack can only pass one argument we have to create a object and use them.
```js
await resetPassword({ password, access_token, refresh_token });
```
In the tanstack custom hook.
```js
import { useMutation } from "@tanstack/react-query";
import { resetPassword as resetPasswordApi } from "../../services/apiAuthentication";
import toast from "react-hot-toast";
import { useNavigate } from "react-router";  

export function useResetPassword() {
  const navigate = useNavigate();

  const {
    mutateAsync: resetPassword,
    isLoading,
    isError,
    error,
  } = useMutation({
    mutationFn: resetPasswordApi,
    onSuccess: () => {
      toast.success("Password reset successfully");

      navigate("/login");
    },
    onError: (err) => {
      toast.error(err.response.data.message);
    },
  });
  
  return { resetPassword, isLoading, isError, error };
}
```
In api layer:
```js
export const resetPassword = async function ({
  password,
  access_token,
  refresh_token,
}) {
  const res = await AxiosInstance({
    url: "/users/resetPassword",
    method: "post",
    data: {
      password,
      access_token,
      refresh_token,
    },
  });

  if (res.data.status === "success") {
    return true;
  }
  
  return false;
};
```

In backend, you first need to setup session and than update password.
```js
export const resetPassword = catchAsync(async function (req, res, next) {
  const { password, access_token, refresh_token } = req.body;

  if (!access_token || !refresh_token) {
    return res.status(401).json({
      status: "error",
      message: "Something went wrong. Please try again later.",
    });
  }

  const { error: sessionError } = await supabase.auth.setSession({
    access_token,
    refresh_token,
  });

  if (sessionError) {
    return res.status(401).json({
      status: "error",
      message: "Invalid or expired session. Please log in again.",
    });
  }
  
  const { error } = await supabase.auth.updateUser({
    password,
  });
  
  if (error) {
    return res.status(401).json({
      status: "error",
      message: "Something went wrong. Please try again later.",
    });
  }
  
  res.status(200).json({
    status: "success",
    message: "Password updated successfully",
  });
});
```
