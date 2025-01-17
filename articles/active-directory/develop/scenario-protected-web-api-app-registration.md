---
title: Protected web API app registration | Azure
titleSuffix: Microsoft identity platform
description: Learn how to build a protected web API and the information you need to register the app.
services: active-directory
author: jmprieur
manager: CelesteDG

ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/26/2021
ms.author: jmprieur
ms.custom: aaddev
#Customer intent: As an application developer, I want to know how to write a protected web API using the Microsoft identity platform for developers.
---

# Protected web API: App registration

This article explains the specifics of app registration for a protected web API.

For the common steps to register an app, see [Quickstart: Register an application with the Microsoft identity platform](quickstart-register-app.md).

## Accepted token version

The Microsoft identity platform can issue v1.0 tokens and v2.0 tokens. For more information about these tokens, see [Access tokens](access-tokens.md).

The token version your API may accept depends on your **Supported account types** selection when you create your web API application registration in the Azure portal.

- If the value of **Supported account types** is **Accounts in any organizational directory and personal Microsoft accounts (e.g. Skype, Xbox, Outlook.com)**, the accepted token version must be v2.0.
- Otherwise, the accepted token version can be v1.0.

After you create the application, you can determine or change the accepted token version by following these steps:

1. In the Azure portal, select your app and then select **Manifest**.
1. Find the property **accessTokenAcceptedVersion** in the manifest.
1. The value specifies to Azure Active Directory (Azure AD) which token version the web API accepts.
   - If the value is 2, the web API accepts v2.0 tokens.
   - If the value is **null**, the web API accepts v1.0 tokens.
1. If you changed the token version, select **Save**.

The web API specifies which token version it accepts. When a client requests a token for your web API from the Microsoft identity platform, the client gets a token that indicates which token version the web API accepts.

## No redirect URI

Web APIs don't need to register a redirect URI because no user is interactively signed in.

## Exposed API

Other settings specific to web APIs are the exposed API and the exposed scopes or app roles.

### Application ID URI and scopes

Scopes usually have the form `resourceURI/scopeName`. For Microsoft Graph, the scopes have shortcuts. For example, `User.Read` is a shortcut for `https://graph.microsoft.com/user.read`.

During app registration, define these parameters:

- The resource URI
- One or more scopes
- One or more app roles

By default, the application registration portal recommends that you use the resource URI `api://{clientId}`. This URI is unique but not human readable. If you change the URI, make sure the new value is unique. The application registration portal will ensure that you use a [configured publisher domain](howto-configure-publisher-domain.md).

To client applications, scopes show up as _delegated permissions_ and app roles show up as _application permissions_ for your web API.

Scopes also appear on the consent window that's presented to users of your app. Therefore, provide the corresponding strings that describe the scope:

- As seen by a user.
- As seen by a tenant admin, who can grant admin consent.

App roles cannot be consented to by a user (as they're used by an application that call the web API on behalf of itself). A tenant administrator will need to consent to client applications of your web API exposing app roles. See [Admin consent](v2-admin-consent.md) for details.

### Exposing delegated permissions (scopes)

1. Select **Expose an API** in the application registration.
1. Select **Add a scope**.
1. If prompted, accept the proposed application ID URI (`api://{clientId}`) by selecting **Save and Continue**.
1. Specify these values:
   - Select **Scope name** and enter **access_as_user**.
   - Select **Who can consent** and make sure **Admins and users** is selected.
   - Select **Admin consent display name** and enter **Access TodoListService as a user**.
   - Select **Admin consent description** and enter **Accesses the TodoListService web API as a user**.
   - Select **User consent display name** and enter **Access TodoListService as a user**.
   - Select **User consent description** and enter **Accesses the TodoListService web API as a user**.
   - Keep the **State** value set to **Enabled**.
1. Select **Add scope**.

### If your web API is called by a daemon app

In this section, you learn how to register your protected web API so that daemon apps can securely call it.

- You declare and expose only _application permissions_ because daemon apps don't interact with users. Delegated permissions wouldn't make sense.
- Tenant admins can require Azure AD to issue web API tokens only to applications that have registered to access one of the API's application permissions.

#### Exposing application permissions (app roles)

To expose application permissions, edit the manifest.

1. In the application registration for your application, select **Manifest**.
1. To edit the manifest, find the `appRoles` setting and add application roles. The role definitions are provided in the following sample JSON block.
1. Leave `allowedMemberTypes` set to `"Application"` only.
1. Make sure `id` is a unique GUID.
1. Make sure `displayName` and `value` don't contain spaces.
1. Save the manifest.

The following sample shows the contents of `appRoles`, where the value of `id` can be any unique GUID.

```json
"appRoles": [
  {
    "allowedMemberTypes": [ "Application" ],
    "description": "Accesses the TodoListService-Cert as an application.",
    "displayName": "access_as_application",
    "id": "ccf784a6-fd0c-45f2-9c08-2f9d162a0628",
    "isEnabled": true,
    "lang": null,
    "origin": "Application",
    "value": "access_as_application"
  }
],
```

#### Ensuring that Azure AD issues tokens for your web API to only allowed clients

The web API checks for the app role. This role is a software developer's way to expose application permissions. You can also configure Azure AD to issue API tokens only to apps that the tenant admin approves for API access.

To add this increased security:

1. Go to the app **Overview** page for your app registration.
1. Under **Managed application in local directory**, select the link with the name of your app. The label for this selection might be truncated. For example, you might see **Managed application in ...**

   When you select this link, you go to the **Enterprise Application Overview** page. This page is associated with the service principal for your application in the tenant where you created it. You can go to the app registration page by using the back button of your browser.

1. Select the **Properties** page in the **Manage** section of the Enterprise application pages.
1. If you want Azure AD to allow access to your web API from only certain clients, set **User assignment required?** to **Yes**.

   
   If you set **User assignment required?** to **Yes**, Azure AD checks the app role assignments of a client when it requests a web API access token. If the client isn't assigned to any app roles, Azure AD will return the error message "invalid_client: AADSTS501051: Application \<application name\> isn't assigned to a role for the \<web API\>".
   
   If you keep **User assignment required?** set to **No**, Azure AD won’t check the app role assignments when a client requests an access token for your web API. Any daemon client, meaning any client using the client credentials flow, can get an access token for the API just by specifying its audience. Any application can access the API without having to request permissions for it.
   
   But as explained in the previous section, your web API can always verify that the application has the right role, which is authorized by the tenant admin. The API performs this verification by validating that the access token has a role claim and that the value for this claim is correct. In the previous JSON sample, the value is `access_as_application`.

1. Select **Save**.

## Next steps

Move on to the next article in this scenario,
[App code configuration](scenario-protected-web-api-app-configuration.md).
