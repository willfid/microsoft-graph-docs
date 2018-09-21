# Working with the calls REST API in Microsoft Graph

> **Important:** APIs under the /beta version in Microsoft Graph are in preview and are subject to change. Use of these APIs in production applications is not supported.

The calls API enables you to create and receive calls from users in Skype and Microsoft Teams. This API supports he following scenarios:

- Ad-hoc two-party calling
- Ad-hoc multiparty calling
- Meet now
- Calling into private meetings
- Calling into Microsoft Teams meetings
- IVR scenarios

Before you get started with the calls API, it is important to understand your media processing requirements.

## MediaPaaS

Media processing is managed through Microsoft Media Platform (MediaPaaS). MediaPaaS helps bots engage in Skype/Microsoft Teams audio/video calls and conferences. It allows real-time bots to participate in both 1:1 and group/multiparty calls.

- Direct (**application media**) media bot calls - You can build real-time bots using the SmartAgents API and have access to direct media I/O. Also known as the [Bots Media Library](https://docs.microsoft.com/en-us/bot-framework/dotnet/bot-builder-dotnet-real-time-media-concepts), it helps you build rich real-time media calling bots. You host the smart agents library and media processor.
- Remote media (**service media**) bot calls: developers can manage the workflow but offload media hosting to MediaPaaS/IVR.

## SDK
Calling APIs may be used directly by the **service media** bot. In these cases, the Bot is usually _Stateless_ and does not process media locally.

[Graph Calling SDK](https://graphcallingsdk-docs.azurewebsites.net/index.html) is provided to simplify the creation of bots. The SDK provides functionality to manage the states of the resources in memory and to pull in bot developers' media stack. The Media Extension allows the bot developers to host the media locally and gain access to the low level audio-video sockets.

# Registering a Calling Bot

> **Important:** APIs for Calling in Microsoft Graph are in preview and are subject to change. Use of these APIs in production applications is not supported.

In this topic, learn how to register a new Calling Bot.

## Register your bot in the Azure Bot Service

Complete the following steps:
1. Register a bot through [Azure Bot Channel Registration](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration).
1. Once you complete the registration, take a note of the registered config values (Bot Name, Application Id and Application Secret).  You will need these values later in the code samples.
1. Enable the Skype channel and configure the Calling tab settings to enable calling.  Fill in the Webhook (for calling) where you will receive incoming notifications. E.g. `https://{your domain}/api/calls`. Refer to [Connect a bot to channels](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels) for more information on how to configure channels.
   More about receiving incoming notification can be found here [Calling Notification](../../../concepts/calling-notifications.md)
1. Enable the Microsoft Teams channel.

## Permissions

### Add Microsoft Graph Permissions for Calling to your Bot

Microsoft Graph exposes granular permissions that control the access that apps have to resources. As a developer, you decide which permissions for Microsoft Graph your app requests.  The Microsoft Graph Calling APIs support Application permissions, which are used by apps that run without a signed-in user present; for example, apps that run as background services or bots.  Application permissions can only be consented by an administrator.  Calling bots and applications have some capabilties that will need permission consent.  Below is a list of those permissions:

|Permission|Display String|Description|Admin Consent Required|
|---| ------------- |---|--|
|Calls.Initiate.All|Initiate outgoing 1:1 calls from the app (preview)|Allows the app to place outbound calls to a single user and transfer calls to users in your organization’s directory, without a signed-in user.|Yes|
|Calls.InitiateGroupCall.All|Initiate outgoing group calls from the app (preview)|Allows the app to place outbound calls to multiple users and add participants to meetings in your organization, without a signed-in user.|Yes|
|Calls.JoinGroupCall.All|Join Group Calls and Meetings as an app (preview)|Allows the app to join group calls and scheduled meetings in your organization, without a signed-in user.  The app will be joined with the privileges of a directory user to meetings in your tenant.|Yes|
|Calls.JoinGroupCallasGuest.All|Join Group Calls and Meetings as a guest (preview)|Allows the app to anonymously join group calls and scheduled meetings in your organization, without a signed-in user.  The app will be joined as a guest to meetings in your tenant.|Yes|
|Calls.AccessMedia.All ^^see note^^|Access media streams in a call as an app (preview)|Allows the app to get direct access to participant media streams in a call, without a signed-in user.|Yes|

> **Note:** You may not use the Microsoft.Graph.Calls.Media API to record or otherwise persist media content from calls or meetings that your bot accesses.

### Assigning Permissions

You pre-configure the application permissions your app needs when you register your app.  To add Permissions from the Azure Bot Registration Portal, do the following:

* From the **Settings** blade, click **Manage**. This is the link appearing by the **Microsoft App ID**. This link will open a window where you can scroll down to add Microsoft Graph Permissions: under **Microsoft Graph**, choose **Add** next to **Application Permissions** and then select the permissions your app requires in the **Select Permissions** dialog. <br/>
  ![Manage link in Settings blade](./images/registration-settings-manage-link.png)

  You can also add permissions by accessing your app through the [Microsoft App Registration Portal](https://apps.dev.microsoft.com/).

### Getting Administrator Consent

An administrator can either consent to these permissions using the [Azure portal](https://portal.azure.com) when your app is installed in their organization, or you can provide a sign-up experience in your app through which administrators can consent to the permissions you configured. Once administrator consent is recorded by Azure AD, your app can request tokens without having to request consent again.

You can rely on an administrator to grant the permissions your app needs at the [Azure portal](https://portal.azure.com); however, often, a better option is to provide a sign-up experience for administrators by using the Azure AD v2.0 `/adminconsent` endpoint.  Please refer to the [instructions on constructing an Admin Consent URL](https://developer.microsoft.com/en-us/graph/docs/concepts/auth_v2_service#3-get-administrator-consent) for more detail.

> **Note**: Constructing the Tenant Admin Consent URL requires a configured Redirect URI/Reply URL in the [App Registration Portal](https://apps.dev.microsoft.com/). To add reply URLs for your bot, access your bot registration, choose Advanced Options > Edit Application Manifest.  Add your Redirect URI to the field replyURLs.

> **Important**: Any time you make a change to the configured permissions, you must also repeat the Admin Consent process. Changes made in the app registration portal will not be reflected until consent has been reapplied by the tenant's administrator.

# Calls
[Application](./application.md) is used for creating a new call by posting to the calls collection.
[Call](./call.md) provides APIs to manage a call.

# Samples

Samples are hosted in [VSTS](https://sampleapps-microsoftteams.visualstudio.com/_git/CVIBot) and you can get started by reading the [README](https://sampleapps-microsoftteams.visualstudio.com/_git/CVIBot?path=%2FLocalMediaSampleBot%2FREADME.md&version=GBmaster) file.

# Testing

> **Important:** APIs for Calling in Microsoft Graph are in preview and are subject to change. Use of these APIs in production applications is not supported.

This document describes how to setup the Bot for testing locally by tunneling the traffic (media and signaling) through ngrok to your local machine. 

## Prerequisites
The testing setup requires ngrok to create tunnels to localhost. Go to [ngrok](https://ngrok.com) and sign up for a free account. Once you signed up, go to the [dashboard](https://dashboard.ngrok.com) and get your authtoken.

Create an ngrok configuration file `ngrok.yml` with the following data
```
authtoken: <Your-AuthToken>
```


> **TIP**: Free ngrok account does not provide static tunnels. Tunnels change everytime a tunnel is created. So, if using free account, it is recommended to not close ngrok until it's use is completed.

> **TIP**: Ngrok does not require sign up if you do not use TCP tunnels.

## Setting up Signaling

In order for the platform to talk to your bot, the bot needs to be reached over the internet. So, an ngrok tunnel is created in http mode with an address pointing to a port on your localhost. Add the following lines to your ngrok config

```
tunnels:
    signaling:
        addr: 12345
        proto: http
```

## Setting up Local Media

> **NOTE**: This section is only required for Local Media bots and can be skipped if you do not host media yourself.

Local Media uses certificates and TCP tunnels to properly work. The following steps are required in order for proper media establishment.

- Ngrok's public TCP endpoints have fixed urls. They are `0.tcp.ngrok.io`, `1.tcp.ngrok.io`, etc. You should have a dns CNAME entry for your service that points to these urls. In this example, let's say `0.bot.contoso.com` is pointing to `0.tcp.ngrok.io`, and similarly for other urls.
- Now you require an SSL certificate for the url you own. To make it easy, use an SSL certificate issued to a wild card domain. In this case, it would be `*.bot.contoso.com`. This ssl certificate is validated by Media flow so should match your media flow's public url. Note down the thumbprint and install the thumbprint in your machine certificates.
- Now, we setup a TCP tunnel to forward the traffic to localhost. Write the following lines into your ngrok config.
    ```
    media:
        addr: 8445
        proto: tcp
    ```

## Start Ngrok

Now that ngrok configuration is ready, start it up. Download the ngrok executable and run the following command

```
ngrok.exe start -all -config <Path to your ngrok.yml>
```

This would start ngrok and provide you the public urls which provide the tunnels to your localhost. The output looks like the following
```
Forwarding  http://signal.ngrok.io -> localhost:12345
Forwarding  https://signal.ngrok.io -> localhost:12345
Forwarding  tcp://1.tcp.ngrok.io:12332 -> localhost:8445
```

Here, `12345` is my signaling port, `8445` is the local media port and `12332` is the remote media port exposed by ngrok. Note that we have a forwarding from `1.bot.contoso.com` to `1.tcp.ngrok.io`. This will be used as the media url for bot. These ports are just suggestive and you can use any available port.

### Update Code

Once ngrok is up and running, we update the code to use the config we just setup.

#### Update Signaling

- In the builder, change the `NotificationUrl` to the signaling url provided by ngrok.

```
statefulClientBuilder.SetNotificationUrl(
    new Uri("https://signal.ngrok.io/notificationEndpoint"))
```

> **IMPORTANT**: Replace signal with the one provided by ngrok and the `NotificationEndpoint` with the controller path that receives notification.

> **IMPORTANT**: The url in `SetNotificationUrl` must be HTTPS.

> **IMPORTANT**: Your local instance must be listening to http traffic on the signaling port. The requests made by Platform will reach the bot as localhost http traffic when End to End encryption is not setup.

#### Update Media

Update your `MediaPlatformSettings` to the following.
```
var mediaPlatform = new MediaPlatformSettings 
{
    ApplicationId = <Your application id>
    MediaPlatformInstanceSettings = new MediaPlatformInstanceSettings
    {
        CertificateThumbprint = <Your SSL Cert thumbprint>,
        InstanceInternalPort = <Localhost media port>,
        InstancePublicPort = <Ngrok exposed remote media port>,
        InstancePublicIPAddress = new IPAddress(0x0),
        ServiceFqdn = <Media url for bot (eg: 1.bot.contoso.com)>,
    },
}
```

> **IMPORTANT**: The Certificate Thumbprint provided above should match the Service FQDN. That is why the DNS entries are required.

## Next Steps

Your bot can now run locally and all the flows work from your localhost.

## Caveats

- The free accounts of ngrok do **NOT** provide End to End encryption. The HTTPS data ends at the ngrok url and the data flows unencrypted from ngrok to localhost. You require paid ngrok account and configuration update to use End to End encryption. See [ngrok docs](http://ngrok.com/docs) for steps on setting up secure E2E tunnels.

- Because the bot callback url is dynamic, incoming call scenarios won't work as they are part of bot registration and they are static. One way to fix this is to use a paid ngrok account which provides fixed subdomains to which you can point your bot and the platform.