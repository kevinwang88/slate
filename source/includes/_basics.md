# Basics

Appboy provides a high performance REST API to allow you to track users, send messages, export data, and more.

## What is a REST API?

A REST API is a way to programatically transfer information over the web using a predefined schema. Appboy has created many different endpoints with specific requirements that will perform various actions and/or return various data. API access is done using HTTPS web requests to https://api.appboy.com/

## API Definitions

Below is some terminology that you may see in the Appboy REST API documentation and what it means.

### Company Secret Explanation

The `company_secret` was formerly included with all API requests but has been deprecated as of October 2014. This field will be ignored for all future API requests to ensure backwards compatibility.

### App Group Identifier Explanation

The `app_group_id` indicates the app title with which the data in this request is associated. It can be found in the [Developer Console](https://dashboard.appboy.com/app_settings/api_settings/ "Developer Console") section of the Appboy dashboard.

###  App Identifier Explanation

For Custom Events and Revenue, you may want to specify a particular variant of the App in which the event occurred. For example, if you have event data on your servers that you know came from the Android version of your app, you might want to indicate that so that it's reflected on the dashboard. In that case, you will provide the appropriate app identifier. It can be found in the [Developer Console][8] section of the Appboy dashboard.

###  External User ID Explanation

The `external_id` serves as a unique user identifier for whom you are submitting data. This identifier should be the same as the one you set in the Appboy mobile SDK in order to avoid creating multiple profiles for the same user.

###  Appboy User ID Explanation

The `appboy_id` serves as a unique user identifier that is set by Appboy. This identifier can be used to delete users through the REST API in addition to external_ids.

#### For more information see:

- [Setting User IDs - iOS][9]
- [Setting User IDs - Android][10]
- [Setting User IDs - Windows Phone 8][11]
- [Setting User IDs - Windows Universal][12]

##  Server-Side Only Integration

The combination of Appboy's User Data and Messaging APIs enables powerful server-side integration options. However, we do **not** recommend a server-side only implementation.

The SDK supports several cases which are difficult or not possible to replicate solely via the API:

- Transferring data in real-time
- Automatically capturing device and session data
- Recording anonymous user data
- Changing states from an anonymous user to a known user
- Native Message Types (News-Feed, In-App Messaging)
- Triggered In-App Messages
- Easy Push Messaging Handling
- User Feedback capability
- UI Components
- Message Response Analytics
- Local Time Delivery
- Intelligent Delivery
- Uninstall Tracking Helper Methods

Additionally, the native SDKs optimize network traffic, cache data offline and reduce the burden on your servers.

Effectively, this means:

- Additional burden on the engineering team
- Server costs to support the integration
- Incomplete device and session data
- Segments will not update in real-time
- Campaigns may deliver to users who are no longer part of a segment

##  API Limits

- API access for enterprise customers starts at 50,000 requests per hour and can be increased
based on need.
- API access for non-enterprise customers is limited to 50,000 requests per hour. The limit is raised from 100 requests per hour once you upgrade to a paid plan.
- API access for free accounts is limited to 100 requests per hour.
- Any valid request will include in its response headers the current rate limit statuses:

Header Name             | Description
----------------------- | -----------------------
`X-RateLimit-Limit`     | The maximum number of requests that the consumer is permitted to make per hour.
`X-RateLimit-Remaining` | The number of requests remaining in the current rate limit window.
`X-RateLimit-Reset`     | The time at which the current rate limit window resets in UTC epoch seconds.

- If you have questions about API limits please contact your Customer Success Manager or the [Appboy Community](https://community.appboy.com/appboy).

##  API IP Whitelisting

For additional security, Appboy can whitelist IP addresses which are allowed to hit the REST APIs for a given app group. To mark specific IP addresses and subnets as whitelisted, navigate to the "Developer Console" page under App Settings. Fill in the IP addresses and subnets to whitelist in the box marked below and press save.

![Developer Console][16]

[8]: https://dashboard.appboy.com/app_settings/api_settings/ "Developer Console"
[9]: {{ site.baseurl }}/iOS/#setting-user-ids
[10]: {{ site.baseurl }}/Android/#setting-user-ids
[11]: {{ site.baseurl }}/Windows/Phone8/#setting-user-ids
[12]: {{ site.baseurl }}/Windows/Universal/#setting-user-ids
[16]: {% image_buster /assets/shared/img/ip_whitelisting_dashboard.png %} "Developer Console"
