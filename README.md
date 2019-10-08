http://bit.ly/okta-dev-workshop

# OAuth 2.0 Exercise

- Before starting, import the "API Access Management (OAuth 2.0)" collection from the Okta [Postman collections](https://developer.okta.com/docs/reference/postman-collections/) page.

## Authorization Code Flow

**In Okta:**

- Create a new "Web" Application
- Go to Applications > Add Application > Web
    - Add a login redirect URI: https://oidcdebugger.com/debug
    - Ensure the "Authorization Code" grant type is selected.
    - Copy the Client ID and Client secret to a handy location.
- Get the Authorization Server information.
    - Go to API > Authorization Servers
    - Copy the Issuer URI for your "default" authorization server to a handy location. It looks something like this: `https://dev-120098.okta.com/oauth2/default`
    - Append `/v1/authorize` and `/v1/token` to the above URL and keep those URLs handy.

**In OIDC Debugger tool:**

- Begin the OAuth flow
    - Go to https://oidcdebugger.com and fill in the form with the following values:
        - Authorize URI: `[your authorization server path]/v1/authorize`
        - Redirect URI: `https://oidcdebugger.com/debug`
        - Client ID: Your app client ID created in step #1 above.
        - Scope: `openid profile email`
        - State: `foo` or any other value
        - Click "Send request"
    - Copy the "authorization code" to a convenient location.

> **NOTE:** this code will expire so if you don’t perform the following steps soon, you may have to repeat the steps above to generate a new code.

## Authorization Code Exchange

**In Postman**

- Import the Okta Dev environment for Postman: http://bit.ly/okta-dev-workshop-postman
- change the following values to reflect your Okta tenant and OIDC app created above:
    - url
    - clientId
    - clientSecret

Now:

- Find the "Get Access Token with Code" request in your Postman Collections.
- In the "Body" tab, replace the "code" value with the authorization code returned in the previous step.
- Click "Send"
- Right-click the access token value from the response and set the the `accessToken` value in your environment.  Also copy it to your clipboard.

## Token Introspection

**In Postman**:

- Find the "Inrospect Token" request in your Postman Collections.
- Click "Send"
- Note the token has been decoded and validated by Okta.
- Note for development purposes you can also inspect the token in a tool like https://jsonwebtoken.io  -- try pasting the token into that tool.

## Custom Scopes

**In Okta:**

- Define a custom scope in your authorization server:
    - Go to API > Authorization Servers > default > Scopes > Add Scope
    - Give the scope a name `messages` and add a description (e.g. "Access messages")
- Repeat the steps above to get an access token, starting with the OIDC Debugger tool, this time add the "messages" scope to "Scopes" in the debugger.
- Inspect the token and notice the scope was returned.

## PKCE Flow

TODO

# Okta Sign-In Widget Exercise

**In Glitch**:

- Create an app in [Glitch](https://glitch.com)
    - Go to New Project > hello-webpage -- you'll get a new project with a new name e.g. https://zinc-frown.glitch.me

**In Okta:**

- Create a new SPA OIDC client
    - Go to Applications > Add Application > Single-Page app
    - Add your Glitch app URL as the login redirect URI: e.g. `https://zinc-frown.glitch.me`
    - Ensure the "Implicit" grant types are selected.
    - Copy the Client ID to a handy location.

**In Glitch**:

- Modify `index.html`

Before `</head>` add:

```html
<script src="https://global.oktacdn.com/okta-signin-widget/2.22.0/js/okta-sign-in.min.js" type="text/javascript"></script>
<link href="https://global.oktacdn.com/okta-signin-widget/2.22.0/css/okta-sign-in.min.css" type="text/css" rel="stylesheet"/>
```

In `<body`> add:

```html
<div id="okta-login-container"></div>
```

Move `script.js` reference down to line above `</body>`:

- Modify `script.js`

```javascript
var oktaSignIn = new OktaSignIn({
  baseUrl: "{ OKTA ORG URL }",
  clientId: "{ OIDC CLIENT ID }",
  logo: '//logo.clearbit.com/okta.com',
  authParams: {
    issuer: "{ OKTA ORG URL }/oauth2/default",
    responseType: ['token', 'id_token'],
    scopes: ['email', 'profile', 'openid'],
    display: 'page'
  }
});
if (oktaSignIn.token.hasTokensInUrl()) {

  oktaSignIn.token.parseTokensFromUrl(
    function success(tokens) {
      // Save the tokens for later use, e.g. if the page gets refreshed:
      // Add the token to tokenManager to automatically renew the token when needed
      tokens.forEach(token => {
        if (token.idToken) {
          oktaSignIn.tokenManager.add('idToken', token);
        }
        if (token.accessToken) {
          oktaSignIn.tokenManager.add('accessToken', token);
        }
      });

      // Say hello to the person who just signed in:
      var idToken = oktaSignIn.tokenManager.get('idToken');
      var accessToken = oktaSignIn.tokenManager.get('accessToken');
      console.log(accessToken);
      document.getElementById('okta-login-container').innerHTML = 'Hello, ' + idToken.claims.email;
      // Remove the tokens from the window location hash
      window.location.hash='';
    },
    function error(err) {
      // handle errors as needed
      console.error(err);
    }
  );
} else {
  oktaSignIn.session.get(function (res) {
    // Session exists, show logged in state.
    if (res.status === 'ACTIVE') {
      document.getElementById('okta-login-container').innerHTML = 'Welcome back, ' + res.login;
      return;
    }
    // No session, show the login form
    oktaSignIn.renderEl(
      { el: '#okta-login-container' },
      function success(res) {
        // Nothing to do in this case, the widget will automatically redirect
        // the user to Okta for authentication, then back to this page if successful
      },
      function error(err) {
        // handle errors as needed
        console.error(err);
      }
    );
  });
}
```

- Go to your new Glitch project site and open up dev tools.
- Login using the Okta sign in widget.
- Notice you've been logged in as your user, but look at the console for the access token; note that tokens have been stored in Developer Tools > Application > Local Storage
- Go to https://www.jsonwebtoken.io/ and paste the access token


# .NET Core + React Sample App

References:

- https://developer.okta.com/docs/
- https://github.com/okta/samples-aspnetcore
- https://github.com/okta/samples-js-react

## Set up .NET app

**In a terminal window:**

```bash
git clone https://github.com/okta/samples-aspnetcore
cd samples-aspnetcore/resource-server
```

- Open your IDE and:
    - change `launchSettings.json` to use `http://localhost:8000`
    - change `appsettings.json` to use your Okta domain: e.g. `https://dev-120098.okta.com`

```bash
cd okta-aspnetcore-webapi-example
dotnet run
```

- Your API service is now running at http://localhost:8000

## Set up React app

**In Okta:**

- Create a new PKCE OIDC client
    - Go to Applications > Add Application > Native
    - Ensure the "Authorization Code" grant type is selected.
    - Ensure redirect URI is: http://localhost:8080/implicit/callback
    - Ensure "Use PKCE" is enabled
    - Copy the Client ID to a handy location.

**In a new terminal window:**

```bash
git clone https://github.com/okta/samples-js-react
cd samples-js-react/custom-login
npm install
```

Create a `.env` file in the root of the project directory with the following variables populated appropriately:

    ISSUER=https://SUBDOMAIN.okta.com/oauth2/default
    CLIENT_ID=YOUR_PKCE_CLIENT_ID

```bash
npm start
```

- Your app is now running at http://localhost:8080
- Login and click on "messages" to see your sending an access token to the API.
