# Pet Management Web App

This React single-page application demonstrates customer authentication with Asgardeo and uses the resulting access token to call the Pet Management Service.

This guide covers:

- Running the web application locally at `http://localhost:5173`
- Registering the application in Asgardeo
- Configuring username and password login
- Optionally enabling passwordless login with Magic Link
- Connecting the application to the Pet Management Service

## Prerequisites

- Node.js 18 or later
- npm
- An [Asgardeo account and organization](https://console.asgardeo.io/)
- A non-administrator test user in the Asgardeo organization
- A valid email address on the test user's profile if you plan to use Magic Link
- Optional: a running or deployed Pet Management Service

## 1. Register the application in Asgardeo

1. Sign in to the [Asgardeo Console](https://console.asgardeo.io/).
2. Confirm that the correct organization is selected.
3. Go to **Applications** and select **New Application**.
4. Select **React**. If the React template is not available, select **Single-Page Application**.
5. Enter `PetDesk Web` as the name.
6. Enter `http://localhost:5173` as the authorized redirect URL.
7. Create or register the application.
8. Open the application's **Protocol** tab and copy the **Client ID**.
9. Verify the following protocol settings:

   - `http://localhost:5173` is listed under **Authorized redirect URLs**.
   - `http://localhost:5173` is listed under **Allowed origins**.
   - The application is a public client using the authorization code flow with PKCE.

Do not add a client secret to this frontend application. Browser-based single-page applications are public clients, and the React or SPA template enables PKCE.

For more information, see [Register a React application](https://wso2.com/identity-platform/docs/guides/applications/register-react-app/).

## 2. Configure user attributes

The application requests the `openid`, `profile`, and `email` scopes. The Pet Management Service also reads the `email` claim from the access token.

1. Open the registered application in the Asgardeo Console.
2. Go to the **User Attributes** tab.
3. Select **Email** from the email scope.
4. Select **Full Name** from the profile scope.
5. Mark **Email** as mandatory if every application user must have an email address.
6. Click **Update**.

See [Enable user attributes for OpenID Connect applications](https://wso2.com/identity-platform/docs/guides/authentication/user-attributes/enable-attributes-for-oidc-app/) for additional details.

## 3. Create a test user

An Asgardeo administrator account cannot be used as an end user of the application.

1. In the Asgardeo Console, go to **User Management** and then **Users**.
2. Select **Add User** and then **Single User**.
3. Create a user with a valid email address.
4. Set a password or invite the user to set one.

See [Onboard users](https://wso2.com/identity-platform/docs/guides/users/onboard-users/) for additional options.

## 4. Optional: enable Magic Link login

Asgardeo provides Magic Link as a passwordless sign-in option.

1. Open the `PetDesk Web` application in the Asgardeo Console.
2. Go to the **Login Flow** tab.
3. In the Visual Editor, go to **Predefined Flows**, **Basic Flows**, and **Add Passwordless login**.
4. Select **Magic Link**.
5. Confirm the change and click **Update**.

If you use the Classic Editor, add **Magic Link** as an authentication option in the first authentication step.

The test user must have a valid email address. See [Add Magic Link login](https://wso2.com/identity-platform/docs/guides/authentication/passwordless-login/add-passwordless-login-with-magic-link/) for the current product instructions.

## 5. Configure the local application

Open [`config.js`](./config.js) and replace the sample values:

```javascript
window.config = {
    baseUrl: "https://api.asgardeo.io/t/<organization-name>",
    clientID: "<client-id>",
    signInRedirectURL: "http://localhost:5173",
    signOutRedirectURL: "http://localhost:5173",
    storage: "localStorage",
    resourceServerURL: "http://localhost:9090"
};
```

Configuration properties:

- `baseUrl`: Your Asgardeo organization URL. Replace `<organization-name>` with the organization name shown in the Asgardeo Console.
- `clientID`: The Client ID copied from the application's **Protocol** tab.
- `signInRedirectURL`: The exact URL to which Asgardeo redirects after sign-in. For this local setup, use `http://localhost:5173`.
- `signOutRedirectURL`: The exact URL to which Asgardeo redirects after sign-out. For this local setup, use `http://localhost:5173`.
- `storage`: Keep this as `localStorage` when testing Magic Link with this sample. See the storage note below.
- `resourceServerURL`: The base URL of the Pet Management Service or its API gateway endpoint.

The OIDC scopes are configured in [`src/util/getConfig.tsx`](./src/util/getConfig.tsx) as `openid`, `profile`, and `email`.

### Authentication-only setup

Authentication can be tested without a running Pet Management Service. Keep `resourceServerURL` set to `http://localhost:9090`. Sign-in and sign-out will work, but pet and notification requests will fail until the service is available.

### Full application setup

Set `resourceServerURL` to one of the following:

- A deployed Pet Management Service API or gateway endpoint
- `http://localhost:9090` when running the Ballerina service locally

The API must:

- Allow cross-origin requests from `http://localhost:5173`
- Accept the Asgardeo access token sent in the `Authorization: Bearer <token>` header

The access token must contain the `sub` and `email` claims expected by the service.

For the complete Choreo service and API setup, see the [Pet Management Application User Guide](../README.md).

## 6. Install and run the application

From the repository root:

```bash
cd b2c-apps/pet-care-app/pet-management-webapp
npm ci
npm run dev
```

Open [http://localhost:5173](http://localhost:5173).

Vite may choose another port if `5173` is already in use. Stop the process using port `5173` and restart the application because the browser URL must exactly match the redirect URL and allowed origin registered in Asgardeo.

## 7. Verify authentication

### Username and password

1. Open `http://localhost:5173`.
2. Click **Get Started**.
3. Sign in with the test user's username and password.
4. Confirm that Asgardeo redirects to `http://localhost:5173`.
5. Verify that the application displays the authenticated view and user menu.

### Magic Link

1. Open `http://localhost:5173` in the browser profile where you read the test user's email.
2. Click **Get Started**.
3. Select Magic Link and enter the test user's username.
4. Open the latest sign-in email and click **Sign In**.
5. Confirm that the application displays the authenticated view.

Magic links are one-time links and should be used promptly. Start a new login attempt if a link has expired or has already been used.

### Why this sample uses `localStorage`

Email clients commonly open Magic Link URLs in a new browser tab. The legacy Asgardeo SDK used by this sample stores the OIDC transaction state and PKCE verifier in `sessionStorage` by default, which is isolated to the tab that initiated login. The callback tab would establish an Asgardeo session but could not complete the application's token exchange.

Using `localStorage` shares the pending transaction data across tabs of the same browser profile. Keep the application tab and email tab in the same browser profile. Different browsers and private browsing sessions do not share this storage.

## Troubleshooting

### Asgardeo reports a redirect URL or origin error

- Confirm that the browser is using exactly `http://localhost:5173`.
- Confirm that this exact value is present in both **Authorized redirect URLs** and **Allowed origins**.
- Do not mix `localhost` and `127.0.0.1`.
- Do not mix HTTP and HTTPS.

### Magic Link authenticates with Asgardeo, but the application still shows Get Started

- Confirm that `storage` is set to `localStorage` in `config.js`.
- Confirm that the application and email are open in the same browser profile.
- Clear site data for `http://localhost:5173`, restart the login flow, and use the newly generated email.
- Do not reuse a Magic Link.

### No Magic Link email arrives

- Confirm that Magic Link is present in the application's login flow.
- Confirm that the test user has a valid email address.
- Check the spam or junk folder.
- Start a new login attempt and use only the latest email.

### Authentication works, but pet data does not load

- Confirm that `resourceServerURL` points to the Pet Management Service or API gateway, not the frontend URL.
- Confirm that the API allows the `http://localhost:5173` origin.
- Check the browser's Network panel for `401`, `403`, or CORS errors.
- Confirm that the access token contains the `sub` and `email` claims.

### The application immediately redirects through Asgardeo

This is expected if an active Asgardeo session already exists. The authorization request can complete without prompting for credentials again.

## Security and production considerations

- The Client ID is public and can be included in frontend configuration. Never place a client secret in `config.js`.
- `localStorage` is used here to support the Magic Link cross-tab flow, but data in `localStorage` is accessible to JavaScript running on the same origin. Apply a strong Content Security Policy and review the application's XSS protections.
- This sample currently uses the legacy `@asgardeo/auth-react` SDK. That package is deprecated. For a production implementation, plan a migration to the current [`@asgardeo/react`](https://www.npmjs.com/package/@asgardeo/react) SDK and reevaluate the token-storage architecture.
- Use HTTPS redirect URLs and allowed origins outside local development.

See the current [React quickstart](https://wso2.com/identity-platform/docs/quick-starts/react/) and [frontend token-storage guidance](https://wso2.com/identity-platform/docs/complete-guides/fesecurity/insecure-tokens/) before using this design in production.
