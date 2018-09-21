# Notifications in Calling

> **Important:** APIs for Calling in Microsoft Graph are in preview and are subject to change. Use of these APIs in production applications is not supported.

Refer to [Calls Api Overview](../api-reference/beta/resources/calls-api-overview.md) on how to register the callback URL. This callback is used for all incoming calls to the application.

## Protocol determination
The incoming notification is provided in legacy format for compatibility with the previous [protocol](https://docs.microsoft.com/en-us/azure/bot-service/dotnet/bot-builder-dotnet-real-time-media-concepts?view=azure-bot-service-3.0). In order to convert the call to the Microsoft Graph protocol, the bot must determine the notification is in legacy format and reply with:

```http
HTTP/1.1 204 No Content
```

The application will again receive the notification but this time it will be in the Microsoft Graph protocol.

You may configure the protocol your application supports and avoid receiving the initial callback in legacy format. The setting is available as a configuration option in the Skype Channel.

## Redirects for region affinity

We will invoke your callback from the data-center hosting the call. The call may start in any data-center and does not take into account region affinities. The notification will be sent to your deployment depending on the GeoDNS resolution. If your application determines, by inspecting the initial notification payload or otherwise, that it needs to run in a different deployment, the application may reply with:

```http
HTTP/1.1 302 Found
Location: your-new-location
```

You may decide to pickup the call and [answer](../api-reference/beta/api/call_answer.md). You can specify the callback URL to handle this particular call. This is useful for _stateful_ instances where your call is handled by a particular partition and you want to embed this information on the callback URL for routing to the right instance.

## Authenticating the callback

Application should inspect the token passed by on the notification to validate the request. Whenever the API raises a web hook event, the API gets an OAUTH token from AAD, with audience as the application's App ID and adds it in the Authorization header as a Bearer token.

![Azure-application-properties](./images/azure-application-properties.png)

The application is expected to validate this token before accepting the callback request.

```http
POST https://bot.contoso.com/api/calls
Content-Type: application/json
Authentication: Bearer <TOKEN>

"value": [
    "subscriptionId": "2887CEE8344B47C291F1AF628599A93C",
    "subscriptionExpirationDateTime": "2016-11-20T18:23:45.9356913Z",
    "changeType": "updated",
    "resource": "/app/calls/8A934F51F25B4EE19613D4049491857B",
    "resourceData": {
        "@odata.type": "#microsoft.graph.call",
        "state": "Established"
    }
]
```

The OAUTH token would have values like the following, and will be signed by AAD.

```json
{
    "aud": "8A34A46B-3D17-4ADC-8DCE-DC4E7D572698",
    "iss": "https://sts.windows.net/31537af4-6d77-4bb9-a681-d2394888ea26/",
    "iat": 1466741440,
    "nbf": 1466741440,
    "exp": 1466745340,
    "appid": "00000004-0000-0ff1-ce00-000000000000",
    "appidacr": "2",
    "oid": "2d452913-80c9-4b56-8419-43a7da179822",
    "sub": "2d452913-80c9-4b56-8419-43a7da179822",
    "tid": "31537af4-6d77-4bb9-a681-d2394888ea26",
    "ver": "1.0"
}
```

* **aud** audience is the App ID specified for the application.
* **tid** is the tenant id for contoso
* **iss** is the token issuer, `https://sts.windows.net/{tenantId}/`

The listener interface on the web hook URL can validate the OAUTH token, ensure it has not expired, checking whether AAD issued and signed the token. We recommend you check whether audience matches your App ID before accepting the callback request.

AuthenticationProvider.cs under [Sample](https://github.com/microsoftgraph/microsoft-graph-comms-samples) shows how to validate inbound requests.

## Additional information

You can read more about [AAD token validation](http://www.cloudidentity.com/blog/2014/03/03/principles-of-token-validation/) in Vittorio Bertocci's **Cloud Identity** blog.
