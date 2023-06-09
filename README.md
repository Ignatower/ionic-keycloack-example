# Keycloak Ionic (Capacitor) Example
Showing how to connect your Ionic app with a Keycloak instance.

![Demo](https://github.com/JohannesBauer97/keycloak-ionic-example/blob/main/demo.gif?raw=true)

# Machete Nacho

* build android app for the first time:
```
ionic capacitor add android
ionic capacitor build android
ionic capacitor sync android
ionic capacitor open
```

* build android app for next times:
```
ionic capacitor sync android
ionic capacitor open
```

* how to open chrome devtools? https://stackoverflow.com/questions/38744809/ionic-2-how-can-i-get-console-messages-from-android-device 
run on chrome: chrome://inspect/#devices and then choose "inspect".

* Bug we have:
 ERR_CLEARTEXT_NOT_PERMITTED
 ```
 Wt{type: 'discovery_document_load_error', reason: j0, params: null}
params: null
reason: j0
error: ProgressEvent{isTrusted: true, lengthComputable: false, loaded: 0, total: 0, type: 'error',…}
headers: Dn
headers: Map(0){size: 0}
lazyUpdate: null
normalizedNames: Map(0){size: 0}
[[Prototype]]: Object
message: "Http failure response for http://localhost:8080/realms/master/.well-known/openid-configuration: 0 Unknown Error"
name: "HttpErrorResponse"
ok: false
status: 0
statusText: "Unknown Error"
url: "http://localhost:8080/realms/master/.well-known/openid-configuration"
[[Prototype]]: Lp
type: "discovery_document_load_error"
```
Fired by this.oauthService.loadDiscoveryDocument()

# Initial Project Setup
## Requirements
* [NodeJS & NPM](https://nodejs.org/en/)
* [Docker](https://www.docker.com/)

## Install Ionic
[Official Documentation](https://ionicframework.com/)
```shellsession
npm i -g @ionic/cli
```

## Create empty project
[Official Documentation](https://ionicframework.com/docs/cli/commands/start)
```shellsession
ionic start example blank
```

## Start local Keycloak instance using Docker
[Official Documentation](https://www.keycloak.org/getting-started/getting-started-docker)
````shellsession
docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:20.0.1 start-dev
````
This will start Keycloak exposed on the local port `8080`. It will also create an initial admin user with username `admin` and password `admin`.

## Keycloak configuration
This example uses the default `master` realm and `admin` user.
Follow these instructions careful or import the json file at the end to have the same settings.

### Create the example client
1. Create the client with id `example-ionic-app` on `master` realm (default settings in Keycloak 20)
2. Add `http://localhost:8100` to Valid redirect URIs, Valid post logout redirect URIs and Web origins (depending on your platform - see below)
3. Add the predefined token mapper "realm roles" to the client. Navigate to clients -> open example-ionic-app -> open tab client scopes -> open example-ionic-app-dedicated -> add predefined mapper -> search & add "realm roles" mapper -> open realm roles mapper -> edit token claim name from "realm_access.roles" to "realm_roles" -> save

**Step 2** is dependent on the target Platform (iOS, Android, Web) and the deployment type (Development or Production). For production you'll need to configure your domain instead localhost. For iOS deployment you'll need to configure your universal link / url schema as (post logout) redirect url. For Android you'll do the same. The exported example-ionic-app client below is configured for web & iOS development deployment.

__Why do we need the token mapper?__
When you get a JWT access/id token from Keycloak with the default settings, and inspect the token with [jwt.io](https://jwt.io/) you'll see that realm roles are actually already part of the token. But every provider (Keycloak, IdentityServer, Auth0,...) might use a different naming for the fields where the roles are listed. This cannot be handled by a generic OAuth library. But we can configure Keycloak to add the list of realm roles to any token claim we want. And this is what we've done. After adding the token mapper, a new claim "realm_roles" is added which contains a list of the assigned user roles. If you're working with client roles, you can add the predefined mapper "client roles". Make sure that you use no dots in the token claim name, this cannot be handled by the generic OAuth library.

*Exported example-ionic-app client*
```json
{
  "clientId": "example-ionic-app",
  "name": "example-ionic-app",
  "description": "",
  "rootUrl": "",
  "adminUrl": "",
  "baseUrl": "",
  "surrogateAuthRequired": false,
  "enabled": true,
  "alwaysDisplayInConsole": false,
  "clientAuthenticatorType": "client-secret",
  "redirectUris": [
    "myschema://login",
    "http://localhost:8100"
  ],
  "webOrigins": [
    "capacitor://localhost",
    "http://localhost:8100"
  ],
  "notBefore": 0,
  "bearerOnly": false,
  "consentRequired": false,
  "standardFlowEnabled": true,
  "implicitFlowEnabled": false,
  "directAccessGrantsEnabled": true,
  "serviceAccountsEnabled": false,
  "publicClient": true,
  "frontchannelLogout": true,
  "protocol": "openid-connect",
  "attributes": {
    "oidc.ciba.grant.enabled": "false",
    "backchannel.logout.session.required": "true",
    "post.logout.redirect.uris": "http://localhost:8100##myschema://login",
    "oauth2.device.authorization.grant.enabled": "false",
    "display.on.consent.screen": "false",
    "backchannel.logout.revoke.offline.tokens": "false"
  },
  "authenticationFlowBindingOverrides": {},
  "fullScopeAllowed": true,
  "nodeReRegistrationTimeout": -1,
  "protocolMappers": [
    {
      "name": "realm roles",
      "protocol": "openid-connect",
      "protocolMapper": "oidc-usermodel-realm-role-mapper",
      "consentRequired": false,
      "config": {
        "multivalued": "true",
        "userinfo.token.claim": "true",
        "id.token.claim": "true",
        "access.token.claim": "true",
        "claim.name": "realm_roles",
        "jsonType.label": "String"
      }
    }
  ],
  "defaultClientScopes": [
    "web-origins",
    "acr",
    "roles",
    "profile",
    "email"
  ],
  "optionalClientScopes": [
    "address",
    "phone",
    "offline_access",
    "microprofile-jwt"
  ],
  "access": {
    "view": true,
    "configure": true,
    "manage": true
  }
}
```
## Install angular-oauth2-oidc library
* NPMJS: https://www.npmjs.com/package/angular-oauth2-oidc
* Sources and Sample: https://github.com/manfredsteyer/angular-oauth2-oidc
* Source Code Documentation: https://manfredsteyer.github.io/angular-oauth2-oidc/docs
* Community-provided sample implementation: https://github.com/jeroenheijmans/sample-angular-oauth2-oidc-with-auth-guards/

```shellsession
npm i angular-oauth2-oidc --save
```
## Remove the home module
You can see in the example code, that there is no home module. To keep the example simple and small, everything is implemented in the app module/component.

## Add Capacitor for iOS
[Official Documentation](https://ionicframework.com/docs/developing/ios)
```shellsession
ionic capacitor add ios
```

# Connecting Keycloak with Ionic
## Setup app.module.ts
1. Add `HttpClientModule` to imports
2. Add `OAuthModule.forRoot()` to imports

## Use and configure OAuthService (web)
Most of the configuration is self explaining, you can find the URLs for your Keycloak instance in `Realm settings -> Endpoints -> OpenID Endpoint Configuration`. If you're new to OAuth2 you should read into the concept and different authorization flows.

`redirectUri` must be changed depending if the app is running as web, Android or iOS app. This is the URL which Keycloak uses to redirect the user back to your application after successful login, with tokens. Make sure to use the IDENTICAL redirectUri in you Keycloak client config already a missing slash will give you the error "invalid redirect_uri".
* For web: use a web url
* For iOS: use [universal links](https://developer.apple.com/ios/universal-links/) or [url schemas](https://developer.apple.com/documentation/xcode/defining-a-custom-url-scheme-for-your-app)
* For Android: use [app links](https://developer.android.com/training/app-links)

*Auth Config for desktop web applications*
```typescript
let authConfig: AuthConfig = {
      issuer: "http://localhost:8080/realms/master",
      redirectUri: "http://localhost:8100",
      clientId: 'example-ionic-app',
      responseType: 'code',
      scope: 'openid profile email offline_access',
      // Revocation Endpoint must be set manually when using Keycloak
      // See: https://github.com/manfredsteyer/angular-oauth2-oidc/issues/794
      revocationEndpoint: "http://localhost:8080/realms/master/protocol/openid-connect/revoke",
      showDebugInformation: true,
      requireHttps: false
    }
```
## Use and configure OAuthService (iOS)
For iOS this example uses [url schemas](https://developer.apple.com/documentation/xcode/defining-a-custom-url-scheme-for-your-app) and not universal links (because it requires a domain + webserver). Whether you use universal links or url schemas, the concept is the same.

1. Open the XCode project and configure an url schema, this example used `my.identifier` as identifier and `myschema` as schema with role `Editor`
2. Add `myschema://login` as redirect uri and post logout uri in the Keycloak client config
3. Use `myschema://login` as new redirect uri in your Angular project
```typescript
let authConfig: AuthConfig = {
      issuer: "http://localhost:8080/realms/master",
      redirectUri: "myschema://login", // needs to be a working universal link / url schema (setup in xcode)
      clientId: 'example-ionic-app',
      responseType: 'code',
      scope: 'openid profile email offline_access',
      // Revocation Endpoint must be set manually when using Keycloak
      // See: https://github.com/manfredsteyer/angular-oauth2-oidc/issues/794
      revocationEndpoint: "http://localhost:8080/realms/master/protocol/openid-connect/revoke",
      showDebugInformation: true,
      requireHttps: false
    }
```
4. Listen when the App is opened by a URL. But we have to detect if it's the Keycloak redirect, for that we used `/login` in our schema.
```typescript
    App.addListener('appUrlOpen', (event: URLOpenListenerEvent) => {
      let url = new URL(event.url);
      if(url.host != "login"){
        // Only interested in redirects to myschema://login
        return;
      }
      
    });
```
5. After the user is redirected back to the app, parse the query parameters and append them to our current active route. Afterwards trigger `tryLogin`.
```typescript
App.addListener('appUrlOpen', (event: URLOpenListenerEvent) => {
      let url = new URL(event.url);
      if(url.host != "login"){
        // Only interested in redirects to myschema://login
        return;
      }

      this.zone.run(() => {

        // Building a query param object for Angular Router
        const queryParams: Params = {};
        for (const [key, value] of url.searchParams.entries()) {
          queryParams[key] = value;
        }

        // Add query params to current route
        this.router.navigate(
          [],
          {
            relativeTo: this.activatedRoute,
            queryParams: queryParams,
            queryParamsHandling: 'merge', // remove to replace all query params by provided
          })
          .then(navigateResult => {
            // After updating the route, trigger login in oauthlib and
            this.oauthService.tryLogin().then(tryLoginResult => {
              console.log("tryLogin", tryLoginResult);
              if (this.hasValidAccessToken){
                this.loadUserProfile();
                this.realmRoles = this.getRealmRoles();
              }
            })
          })
          .catch(error => console.error(error));

      });
    });
```
6. The OAuth library detects the query params and continues the login flow.

## Use and configure OAuthService (Android)
The example is not having an Android project attached, but the concept for iOS and Android are the same.

[Android uses app links](https://developer.android.com/training/app-links) instead universal links/url schemas like in iOS.

## Setup the app start
When a user enters the app, we want to check if there is a valid access token or if the user needs to log in.

*app.component.ts*
```typescript
export class AppComponent implements OnInit{
  public hasValidAccessToken = false;
  public realmRoles: string[] = [];

  ngOnInit(): void {
    /**
     * Load discovery document when the app inits
     */
    this.oauthService.loadDiscoveryDocument()
      .then(loadDiscoveryDocumentResult => {
        console.log("loadDiscoveryDocument", loadDiscoveryDocumentResult);

        /**
         * Do we have a valid access token? -> User does not need to log in
         */
        this.hasValidAccessToken = this.oauthService.hasValidAccessToken();

        /**
         * Always call tryLogin after the app and discovery document loaded, because we could come back from Keycloak login page.
         * The library needs this as a trigger to parse the query parameters we got from Keycloak.
         */
        this.oauthService.tryLogin().then(tryLoginResult => {
          console.log("tryLogin", tryLoginResult);
          if (this.hasValidAccessToken){
            this.loadUserProfile();
            this.realmRoles = this.getRealmRoles();
          }
        });

      })
      .catch(error => {
        console.error("loadDiscoveryDocument", error);
      });

    /**
     * The library offers a bunch of events.
     * It would be better to filter out the events which are unrelated to access token - trying to keep this example small.
     */
    this.oauthService.events.subscribe(eventResult => {
      console.debug("LibEvent", eventResult);
      this.hasValidAccessToken = this.oauthService.hasValidAccessToken();
    })
  }
````
## Login
Here we can use the library methods.

*app.component.ts*
```typescript
  /**
   * Calls the library loadDiscoveryDocumentAndLogin() method.
   */
  public login(): void {
    this.oauthService.loadDiscoveryDocumentAndLogin()
      .then(loadDiscoveryDocumentAndLoginResult => {
        console.log("loadDiscoveryDocumentAndLogin", loadDiscoveryDocumentAndLoginResult);
      })
      .catch(error => {
        console.error("loadDiscoveryDocumentAndLogin", error);
      });
  }
````

## Logout
Here we can use the library methods.

*app.component.ts*
```typescript
  /**
   * Calls the library revokeTokenAndLogout() method.
   */
  public logout(): void {
    this.oauthService.revokeTokenAndLogout()
      .then(revokeTokenAndLogoutResult => {
        console.log("revokeTokenAndLogout", revokeTokenAndLogoutResult);
      })
      .catch(error => {
        console.error("revokeTokenAndLogout", error);
      });
  }
````

## Load Userprofile
Here we can use the library methods.

*app.component.ts*
```typescript
  /**
   * Calls the library loadUserProfile() method and sets the result in this.userProfile.
   */
  public loadUserProfile(): void {
    this.oauthService.loadUserProfile()
      .then(loadUserProfileResult => {
        console.log("loadUserProfile", loadUserProfileResult);
        this.userProfile = loadUserProfileResult;
      })
      .catch(error => {
        console.error("loadUserProfile", error);
      });
  }
````
## Get Realm Roles
In the earlier chapters we configured the client in Keycloak that the realm roles are added as claims to the tokens.
This method shows how to access them now.

*app.component.ts*
```typescript
  /**
   *  Use this method only when an id token is available.
   *  This requires a specific mapper setup in Keycloak. (See README file)
   *
   *  Parses realm roles from identity claims.
   */
  public getRealmRoles(): string[] {
    let idClaims = this.oauthService.getIdentityClaims()
    if (!idClaims){
      console.error("Couldn't get identity claims, make sure the user is signed in.")
      return [];
    }
    if (!idClaims.hasOwnProperty("realm_roles")){
      console.error("Keycloak didn't provide realm_roles in the token. Have you configured the predefined mapper realm roles correct?")
      return [];
    }

    let realmRoles = idClaims["realm_roles"]
    return realmRoles ?? [];
  }
````
