---
title: "Enforcing MFA in authentik for administrators only"
date: 2023-04-23T11:50:50+02:00
draft: false
author: Lea
author_url: https://lea.pet/@lea
---

This will be a short tutorial to set up [authentik](https://goauthentik.io/) to enforce MFA using WebAuthn keys for admins while allowing normal users to use them as single factor.
<!--more-->
The resulting authentication flow will allow users to either sign in using their password (and TOTP, if configured) **or** using a WebAuthn security key. However, for administrators, the WebAuthn 1FA flow will be disabled and signing in with the password will require the WebAuthn device as second factor.

### Creating the policy

> There may be a better way to have a policy check whether a user is an administrator, but using an Expression Policy is my preferred method.

First, create a new [Expression Policy](https://goauthentik.io/docs/policies/expression/) with a name like `pending-user-is-admin` and the following expression:

```py
if context['pending_user'].is_staff:
  return True
return False
```

This policy will later be bound to flow stages. If you didn't know, assigning a policy to a stage will **skip the stage** if the policy is denied. Keep this in mind so you don't make the same mistake as me and accidentally disable authentication completely by assigning a binding to the wrong stage 😉

### Setting up a WebAuthn 1FA flow

Next, let's create a flow that will allow users to sign in only using their WebAuthn device. Call it something like `webauthn-1fa`.

> If you already have such a flow, you most likely only need to add the `deny` stage to it.

Create and add the following stages:

- `1fa-identification` (Identification Stage) \
  **User fields:** Username, Email
- `deny` (Deny Stage) \
  After adding this stage, expand it and bind your `pending-user-is-admin` policy to it. This will deny flow execution if the user is an admin, preventing admins from using WebAuthn devices for single factor authentication.
- `webauthn-1fa` (Authenticator Validation Stage) \
  **Device classes:** *Only* check `WebAuthn Authenticators` \
  **Not configured action:** Deny the user access
- `default-authentication-login` (User Login Stage): You can just bind your existing login stage here

The resulting flow diagram should look like this:

{{< image "authentik-admin-2fa/1fa-flow-diagram.png" >}}

### Modifying the default authentication flow

This next step assumes you're using authentik's default authentication flow, which should look similar to this:

{{< image "authentik-admin-2fa/default-authentication-flow.png" >}}

If you already have a custom authentication flow, you'll need to adapt these instructions to it.

First, edit the `default-authentication-identification` stage. Expand the "Flow settings" section, and under "Passwordless flow", select the `webauthn-1fa` flow you previously created. This will show a "Use security key" button on the login page, which users can use to access that flow.

Next, edit the `default-authentication-mfa-validation` stage and uncheck "WebAuthn Authenticators" from the Device classes list. This will skip 2FA if only a WebAuthn device is configured, but will still prompt for TOTP authenticators if the user has set one of these up.

Create a new Authenticator Validation Stage with a name like `admin-mfa-validation`, which has every Device class checked. Add it to the flow with the same order number as the other MFA stage.

Finally, bind the `pending-user-is-admin` policy to both the `default-authentication-mfa-validation` and `admin-mfa-validation` stages, but check "Negate result" on the binding to the default stage. \
This will make it so the default MFA stage (with WebAuthn removed) executes for normal users, and the admin MFA stage executes only for administrators.

The flow diagram authentik generates doesn't appear to take the "Negate result" option into consideration, meaning the flow diagram isn't accurate ("Policy passed" and "Policy denied" should be swapped after the first Expression Policy, and the second Expression Policy check can be omitted. I'm not sure why the diagram generates like this).

{{< image "authentik-admin-2fa/2fa-flow-diagram.png" >}}

### Testing the new configuration

Finally, you should make sure that the new flows work as expected.

In a private window, open the login screen and click "Use a security key". When entering the username of an **administrator account** or an account that **doesn't have a WebAuthn device** set up, you should be greeted by a "Request has been denied" message. Entering the name of a normal user account should pop up a WebAuthn prompt without asking for a password.

Next, test the normal authentication flow with an administrator account and a normal user account. The admin should be prompted for a WebAuthn device, while the normal user should be able to log in only with a password (and TOTP, if configured).
