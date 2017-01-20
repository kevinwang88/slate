---
nav_title: Messaging
---
{% raw %}

# Messaging

## Overview

The Appboy messaging API provides you with two distinct options for sending messages to your users. You can provide the message contents and configuration in the API request with the `/messages/send` and `/messages/schedule` endpoints. Alternatively, you can manage the details of your message with an API-Triggered Delivery campaign in the dashboard and just control when and to whom it is sent with the `campaigns/trigger/send` and `campaigns/trigger/schedule` endpoints. The following sections will detail the request specification for both methods.

__Note__: Similarly to other campaigns, you can limit the number of times a particular user can receive a Messaging API campaign by configuring [re-eligibility settings][40] in the Appboy Dashboard. Appboy will not deliver API messages to users that haven't become re-eligible for the campaign regardless of how many API requests are sent.

##  Send Endpoints

The send endpoint allows you to send immediate, ad-hoc messages to designated users. If you are targeting a segment, a record of your request will be stored in the [Developer Console][41].

### Sending Messages Immediately via API Only

```json
POST https://api.appboy.com/messages/send
Content-Type: application/json
{
   "app_group_id": (required, string) see App Group Identifier below,
   // You will need to include at least one of 'segment_id' and 'external_user_ids'
   // Including 'segment_id' will send to members of that segment
   // Including 'external_user_ids' will send to those users
   // Including both will send to the provided user ids if they are in the segment
   "external_user_ids": (optional, array of strings) see External User ID,
   "segment_id": (optional, string) see Segment Identifier,
   "audience": (optional, Audience Object) see Audience,
   "campaign_id": (optional, string) see Campaign Identifier,
   "override_frequency_capping": (optional, bool) ignore frequency_capping for campaigns, defaults to false,
   "recipient_subscription_state": (optional, string) use this to send messages to only users who have opted in ('opted_in'), only users who have subscribed or are opted in ('subscribed') or to all users, including unsubscribed users ('all'), the latter being useful for transactional email messaging. Defaults to 'subscribed',
   "messages": {
     "apple_push": (optional, Apple Push Object),
     "android_push": (optional, Android Push Object),
     "windows_phone8_push": (optional, Windows Phone 8 Push Object),
     "windows_universal_push": (optional, Windows Universal Push Object),
     "kindle_push": (optional, Kindle/FireOS Push Object),
     "web_push": (optional, Web Push Object),
     "in_app_message": (optional, In-App Message Object),
     "email": (optional, Email Object)
   }
 }
```

### Sending Messages via API Triggered Delivery

API Triggered Delivery allows you to house message content inside of the Appboy dashboard, while dictating when a message is sent, and to whom via your API. Please see this section of [Appboy Academy for further details][39].

```json
POST https://api.appboy.com/campaigns/trigger/send
Content-Type: application/json
{
  "app_group_id": (required, string) see App Group Identifier below,
  "campaign_id": (required, string) see Campaign Identifier,
  "trigger_properties": (optional, object) personalization key/value pairs that will apply to all users in this request
  "recipients": (optional, array) [
    {
      "external_user_id": (required, string) External Id of user to receive message,
      "trigger_properties": (optional, object) personalization key/value pairs that will apply to this user (these key/value pairs will override any keys that conflict with trigger_properties above)
    },
    ...
  ]
}
```

__Note__: The `recipients` array may contain up to 50 objects, with each object containing a single `external_user_id` string and `trigger_properties` object.

__Note__: Customers using the API for server-to-server calls may need to whitelist `api.appboy.com` if they're behind a firewall.

##  Schedule Endpoints

The schedule endpoints allow you to send messages at a designated time and modify or cancel messages that you have already scheduled.

###  Create Schedule Endpoint

The create schedule endpoint allows you to schedule a message to go out at a designated time and provides you with an identifier to reference that message for updates. If you are targeting a segment, a record of your request will be stored in the [Developer Console][41] after all scheduled messages have been sent.

#### Scheduling Messages

```json
POST https://api.appboy.com/messages/schedule/create
Content-Type: application/json
{
  "app_group_id": (required, string) see App Group Identifier,
  // You will need to include at least one of 'segment_id' and 'external_user_ids'
  // Including 'segment_id' will send to members of that segment
  // Including 'external_user_ids' will send to those users
  // Including both will send to the provided user ids if they are in the segment
  "external_user_ids": (optional, array of strings) see External User ID,
  "segment_id": (optional, string) see Segment Identifier,
  "campaign_id": (optional, string) see Campaign Identifier,
  "override_messaging_limits": (optional, bool) ignore global rate limits for campaigns, defaults to false,
  "recipient_subscription_state": (optional, string) use this to send messages to only users who have opted in ('opted_in'), only users who have subscribed or are opted in ('subscribed') or to all users, including unsubscribed users ('all'), the latter being useful for transactional email messaging. Defaults to 'subscribed',
  "schedule": {
    "time": (required, datetime as ISO 8601 string) time to send the message,
    "in_local_time": (optional, bool),
    "at_optimal_time": (optional, bool),
  },
  "messages": {
    "apple_push": (optional, Apple Push Object),
    "android_push": (optional, Android Push Object),
    "windows_push": (optional, Windows Phone 8 Push Object),
    "windows8_push": (optional, Windows Universal Push Object),
    "kindle_push": (optional, Kindle/FireOS Push Object),
    "web_push": (optional, Web Push Object),
    "in_app_message" : (optional, In-App Message Object)
    "email": (optional, Email object)
    "webhook": (optional, Webhook object)
  }
}
```

#### Schedule API Triggered Campaign

```json
POST https://api.appboy.com/campaigns/trigger/schedule/create
Content-Type: application/json
{
  "app_group_id": (required, string) see App Group Identifier,
  "campaign_id": (required, string) see Campaign Identifier,
  // Including 'recipients' will send only to the provided user ids if they are in the campaign's segment
  "recipients": (optional, Array of Recipient Object),
  // for any keys that conflict between these trigger properties and those in a Recipient Object, the value from the
  // Recipient Object will be used
  "trigger_properties": (optional, object) personalization key/value pairs for all users in this send; see Trigger Properties,
  "schedule": {
    "time": (required, datetime as ISO 8601 string) time to send the message,
    "in_local_time": (optional, bool),
    "at_optimal_time": (optional, bool),
  }
}
```

The parameters for the create schedule endpoint mirror those of the send endpoint and add the `schedule` parameter, which allows you to specify when you want your targeted users to receive your message. If you include only the `time` parameter in the `schedule` object, all of your users will be messaged at that time. If you set `in_local_time` to be true, your users will receive the message at the designated date and time in their respective timezones. If `in_local_time`is true, you will get an error response if the `time` parameter has passed in your company's time zone. If you set `at_optimal_time` to be true, your users will receive the message at the designated date at the [optimal time][33] for them (regardless of the time you provide). When using local or optimal time sending, do not provide time zone designators in the value of the time parameter (e.g. just give us `"2015-02-20T13:14:47"` instead of `"2015-02-20T13:14:47-05:00"`).

The response will provide you with a `schedule_id` that you should save in case you later need to cancel or update the message you schedule.

#### Create Schedule Response

```json
Content-Type: application/json
{
  "schedule_id" : (required, string) identifier for the scheduled message that was created
}
```

__Note__: Customers using the API for server-to-server calls may need to whitelist `api.appboy.com` if they're behind a firewall.

###  Update Schedule Endpoint

The update schedule endpoint allows you to change the schedule or message contents of a scheduled message you previously created.

#### Update Message Schedule

```json
POST https://api.appboy.com/messages/schedule/update
Content-Type: application/json
{
  "app_group_id": (required, string) see App Group Identifier below,
  "schedule_id": (required, string) the schedule_id to update (obtained from the response to create schedule),
  "schedule": {
    // optional, see create schedule documentation
  },
  "messages": {
    // optional, see create schedule documentation
  }
}
```

The messages update schedule endpoint accepts updates to either the `schedule` or `messages` parameter or both. Your request must contain at least one of those two keys.

#### Update API Triggered Campaign Schedule

```json
POST https://api.appboy.com/campaigns/trigger/schedule/update
Content-Type: application/json
{
  "app_group_id": (required, string) see App Group Identifier below,
  "campaign_id": (required, string) see Campaign Identifier,
  "schedule_id": (required, string) the schedule_id to update (obtained from the response to create schedule),
  "schedule": {
    // required, see create schedule documentation
  }
}
```

Any schedule or messages object that you provide will completely overwrite the one that you provided in the create schedule request or in previous update schedule requests (so if you originally provide `"schedule" : {"time" : "2015-02-20T13:14:47", "in_local_time" : true}` and then in your update you provide `"schedule" : {"time" : "2015-02-20T14:14:47"}`, your message will now be sent at the provided time in UTC, not in the user's local time). Scheduled messages or triggers that are updated very close to or during the time they were supposed to be sent will be updated with best efforts, so last second changes could be applied to all, some, or none of your targeted users.

###  Delete Schedule Endpoint

The delete schedule endpoint allows you to cancel a message that you previously scheduled before it has been sent.

#### Delete Message Schedule

```json
POST https://api.appboy.com/messages/schedule/delete
Content-Type: application/json
{
  "app_group_id": (required, string) see App Group Identifier below,
  "schedule_id": (required, string) the schedule_id to delete (obtained from the response to create schedule)
}
```

#### Delete Scheduled API Trigger Campaign

```json
POST https://api.appboy.com/campaigns/trigger/schedule/delete
Content-Type: application/json
{
  "app_group_id": (required, string) see App Group Identifier below,
  "campaign_id": (required, string) see Campaign Identifier,
  "schedule_id": (required, string) the schedule_id to delete (obtained from the response to create schedule)
}
```

Scheduled messages or triggers that are deleted very close to or during the time they were supposed to be sent will be updated with best efforts, so last second deletions could be applied to all, some, or none of your targeted users.

##  Parameter Definitions

###  App Group Identifier

The `app_group_id` indicates the app title with which the data in this request is associated and authenticates the requester as someone who is allowed to send messages to the app. It must be included with every request. It can be found in the [Developer Console][18] section of the Appboy dashboard.

###  App Identifier

If you want to send push to a set of device tokens (instead of users), you need to indicate on behalf of which specific app you are messaging. In that case, you will provide the appropriate App Identifier in a Tokens Object. It can be found in the [Developer Console][18] section of the Appboy dashboard.

### External User ID

A unique identifier for sending a message to specific users. This identifier should be the same as the one you [set in the Appboy mobile SDK][19]. You can only target users for messaging who have already been identified through the mobile SDK or the Users API. If you need to send messages to specific users who have not yet been identified to Appboy, consider attaching a Tokens Object (explained below) to your message. A maximum of 50 External User IDs are allowed in a request.

For campaign trigger endpoints, if you provide this field, the criteria will be layered with the campaign's segments and only users who are in the list of External User IDs _and_ the campaign's segment will receive the message.

### Segment Identifier

The `segment_id` indicates the segment to which the message should be sent. A Segment Identifier for each of the segments you have created can be found in the [Developer Console][18] section of the Appboy dashboard. For message endpoints, if you provide both a Segment Identifier _and_ a list of External User IDs in a single messaging request, the criteria will be layered and only users who are in the list of External User IDs _and_ the provided segment will receive the message.

### Campaign Identifier

For messaging endpoints, the `campaign_id` indicates the [API Campaign][26] under which the analytics for a message should be tracked. A Campaign Identifier for each of the campaigns you have created can be found in the [Developer Console][18] section of the Appboy dashboard. If you provide a Campaign Identifier in the request body, you must provide a `message_variation_id` in each of the message objects indicating the represented variant of your campaign.

For campaign trigger endpoints, the `campaign_id` indicates the API ID of the campaign to be triggered. This field is required for all trigger endpoint requests.

### Trigger Properties

When using one of the endpoints for sending a campaign with API Triggered Delivery, you have the option to provide a map of message template keys and values which can then be used to customize your message for this delivery. If you make an API request that contains an object in `"trigger_properties"`, the values in that object can then be referenced in your message template under the `api_trigger_properties` namespace. For example, a request with `"trigger_properties" : {"product_name" : "shoes", "product_price" : 79.99}` could add the word "shoes" to the message by adding `{{api_trigger_properties.${product_name}}}` to the template.

### Recipient Object
```json
{
  "external_user_id": (required, string) see External User Id,
  "trigger_properties": (optional, object) personalization key/value pairs for this user; see Trigger Properties
}
```

### Connected Audience Object

A Connected Audience is a selector that identifies the audience to send the message to. It is composed of either a single Connected Audience Filter, or several Connected Audience Filters in a logical expression using either "AND" or "OR" operators.

Multiple filter example:

```json
{
  "AND":
    [
      Connected Audience Filter,
      {
        "OR" :
          [
            Connected Audience Filter,
            Connected Audience Filter
          ]
      },
      Connected Audience Filter
    ]
}
```

### Connected Audience Filter

These filters are used to create an Connected Audience Object.

#### Custom Attribute Filter

This filter allows you to segment based on a user's custom attribute. These filters contain up to three fields:

```json
{
  "custom_attribute":
    {
      "custom_attribute_name": (String) the name of the custom attribute to filter on,
      "comparison": (String) one of the allowed comparisons to make against the provided value,
      "value": (String, Numeric, Boolean) the value to be compared using the provided comparison
    }
}
```

The custom attribute's type determines the comparisons that are valid for a given filter.

| Custom Attribute Type | Allowed Comparisons |
| ---------------------| --------------- |
| String | `equals`, `not_equal`, `matches_regex`, `does_not_match_regex`, `exists`, `does_not_exist` |
| Array | `includes_value`, `does_not_include_value`, `exists`, `does_not_exist` |
| Numeric | `equals`, `not_equal`, `greater_than`, `greater_than_or_equal_to`, `less_than`, `less_than_or_equal_to`, `exists`, `does_not_exist` |
| Boolean | `equals`, `does_not_equal`, `exists`, `does_not_exist` |
| Time | `less_than_x_days_ago`, `greater_than_x_days_ago`, `less_than_x_days_in_the_future`, `greater_than_x_days_in_the_future`, `after`, `before`, `exists`, `does_not_exist`

__Note__: `value` is not required when using the `exists` or `does_not_exist` comparisons. `value` must be an ISO 8601 DateTime string when using the `before` and `after` comparisons.

Examples:

```json
{
  "custom_attribute":
    {
      "custom_attribute_name": "eye_color",
      "comparison": "equals",
      "value": "blue"
    }
}

{
  "custom_attribute":
  {
    "custom_attribute_name": "favorite_foods",
    "comparison": "includes_value",
    "value": "pizza"
  }
}

{
  "custom_attribute":
  {
    "custom_attribute_name": "last_purchase_time",
    "comparison": "less_than_x_days_ago",
    "value": 2
  }
}
```

#### Push Subscription Filter

This filter allows you to segment based on a user's push subscription status. These filters contain two fields:

```json
{
  "push_subscription_status":
  {
    "comparison": (String) one of the two allowed comparisons listed below,
    "value": (String) one of the three allowed values listed below
  }
}
```

| Allowed Comparisons | Allowed Values |
| ---------------------| --------------- |
| `is`, `is_not` | `opted_in`, `subscribed`, `unsubscribed` |

#### Email Subscription Filter

This filter allows you to segment based on a user's email subscription status. These filters contain two fields:

```json
{
  "email_subscription_status":
  {
    "comparison": (String) one of the two allowed comparisons listed below,
    "value": (String) one of the three allowed values listed below
  }
}
```

| Allowed Comparisons | Allowed Values |
| ---------------------| --------------- |
| `is`, `is_not` | `opted_in`, `subscribed`, `unsubscribed` |

### Apple Push Object

```json
{
   "badge": (optional, int) the badge count after this message,
   "alert": (required unless content-available is true, string or Apple Push Alert Object) the notification message,
   // Specifying "default" in the sound field will play the standard notification sound
   "sound": (optional, string) the location of a custom notification sound within the app,
   "extra": (optional, object) additional keys and values to be sent,
   "content-available": (optional, boolean) if set, Appboy will send down the "content-available" flag with the push token,
   "category": (optional, string) define a type of notification which contains actions a user can perform in response,
   "expiry": (optional, ISO 8601 date string) if set, push messages will expire at the specified datetime,
   "custom_uri": (optional, string) a web URL, or Deep Link URI,
   "message_variation_id": (optional, string) used when providing a campaign_id to specify which message variation this message should be tracked under (must be an iOS Push Message),
   "asset_url": (optional, string) content URL for rich notifications for devices using iOS 10 or higher,
   "asset_file_type": (required if asset_url is present, string) file type of the asset - one of "aif", "gif", "jpg", "m4a", "mp3", "mp4", "png", or "wav"
}
```

-You must include an Apple Push Object in `messages` if you want users you have targeted to receive a push on their iOS Devices. The total number of bytes in your `alert` string, `extra` object, and other optional parameters should not exceed 1912. The Messaging API will return an error if you exceed the message size allowed by Apple. Messages that include the keys `ab` or `aps` in the `extra` object will be rejected.

### Apple Push Alert Object

__Note__: In most cases, `alert` can just be specified in an `apple_push` object as a string. You should specify `alert` as an object only in cases where you need specific localization or Apple Watch customization.

```json
{
   "body": (required unless content-available is true in the Apple Push Object, string) the text of the alert message,
   "title": (optional, string) a short string describing the purpose of the notification, displayed as part of the Apple Watch notification interface,
   "title_loc_key": (optional, string) the key to a title string in the `Localizable.strings` file for the current localization,
   "title_loc_args": (optional, array of strings) variable string values to appear in place of the format specifiers in title_loc_key,
   "action_loc_key": (optional, string) if a string is specified, the system displays an alert that includes the Close and View buttons, the string is used as a key to get a localized string in the current localization to use for the right button’s title instead of “View",
   "loc_key": (optional, string) a key to an alert-message string in a Localizable.strings file for the current localization,
   "loc_args": (optional, array of strings) variable string values to appear in place of the format specifiers in loc_key
}
```

###  Android Push Object

```json
{
   "alert": (required, string) the notification message,
   "title": (required, string) the title that appears in the notification drawer,
   "extra": (optional, object) additional keys and values to be sent in the push,
   "message_variation_id": (optional, string) used when providing a campaign_id to specify which message variation this message should be tracked under (must be an Android Push Message),
   "priority": (optional, integer) the notification priority value,
   "send_to_sync": (optional, if set to True we will throw an error if "alert" or "title" is set),
   "collapse_key": (optional, string) the collapse key for this message,
   // Specifying "default" in the sound field will play the standard notification sound
   "sound": (optional, string) the location of a custom notification sound within the app,
   "custom_uri": (optional, string) a web URL, or Deep Link URI,
   "summary_text": (optional, string),
   "delay_while_idle": (optional, boolean),
   "time_to_live": (optional, integer (seconds)),
   "notification_id": (optional, integer),
   "push_icon_image_url": (optional, string) an image URL for the large icon,
   "accent_color": (optional, integer) accent color to be applied by the standard Style templates when presenting this notification, an RGB integer value
}
```

__Note__: You can send "Big Picture" notifications by specifying the key `appboy_image_url` in the `extra` object. The value for `appboy_image_url` should be a URL that links to where your image is hosted. Images need to be cropped to a 2:1 aspect ratio and should be at least 600x300. Images used for notifications will only display on devices running Jelly Bean (Android 4.1) or higher.

__Note__: `priority` will accept values from -2 to 2, where -2 represents "MIN" priority and 2 represents "MAX". 0 is the "DEFAULT" value. Any values sent that outside of that integer range will default to 0. For more information on which priority level to use, please see our section on [Android Notification Priority][29].

__Note__: The value for the large icon `push_icon_image_url` should be a URL that links to where your image is hosted. Images need to be cropped to a 1:1 aspect ratio and should be at least 40x40. Images used for custom notification icons will only display on devices running Honeycomb MR1 (Android 3.1) or higher.

For more information on collapsing notifications using the `collapse_key` please see the [Android Developer Docs][35]

For more information on `send_to_sync` messages please see our section on ["Silent Android Notifications"][28]

You must include an Android Push Object in `messages` if you want users you have targeted to receive a push on their Android devices. The total number of bytes in your `alert` string and `extra` object should not exceed 4000. The Messaging API will return an error if you exceed the message size allowed by Google.

### Kindle/FireOS Push Object

```json
{
   "alert": (required, string) the notification message,
   "title": (required, string) the title that appears in the notification drawer,
   "extra": (optional, object) additional keys and values to be sent in the push,
   "message_variation_id": (optional, string) used when providing a campaign_id to specify which message variation this message should be tracked under (must be an Kindle/FireOS Push Message),
   "priority": (optional, integer) the notification priority value,
   "collapse_key": (optional, string) the collapse key for this message,
   // Specifying "default" in the sound field will play the standard notification sound
   "sound": (optional, string) the location of a custom notification sound within the app,
   "custom_uri": (optional, string) a web URL, or Deep Link URI
}
```

__Note__: `priority` will accept values from -2 to 2, where -2 represents "MIN" priority and 2 represents "MAX". 0 is the "DEFAULT" value. Any values sent that outside of that integer range will default to 0.

### Web Push Object

```json
{
   "alert": (required, string) the notification message,
   "title": (required, string) the title that appears in the notification drawer,
   "extra": (optional, object) additional keys and values to be sent in the push,
   "message_variation_id": (optional, string) used when providing a campaign_id to specify which message variation this message should be tracked under (must be an Kindle/FireOS Push Message),
   "custom_uri": (optional, string) a web URL,
   "image_url": (optional, string) url for image to show,
   "time_to_live": (optional, integer (seconds))
}
```

__Note__: The value for `image_url` should be a URL that links to where your image is hosted. Images need to be cropped to a 1:1 aspect ratio.

### Windows Phone 8 Push Object

```json
{
   "push_type": (optional, string) must be "toast",
   "toast_title": (optional, string) the notification title,
   "toast_content": (required, string) the notification message,
   "toast_navigation_uri": (optional, string) page uri to send user to,
   "toast_hash": (optional, object) additional keys and values to send,
   "message_variation_id": (optional, string) used when providing a campaign_id to specify which message variation this message should be tracked under (must be a Windows Phone 8 Push Message)
}
```

###  Windows Universal Push Object

See the Windows Universal [toast template catalog][32] for details on the options for `push_type` below.

```json
{
   "push_type": (required, string) one of: "toast_text_01", "toast_text_02", "toast_text_03", "toast_text_04", "toast_image_and_text_01", "toast_image_and_text_02", "toast_image_and_text_03", or "toast_image_and_text_04",
   "toast_text1": (required, string) the first line of text in the template,
   "toast_text2": (optional, string) the second line of text (for templates with > 1 line of text),
   "toast_text3": (optional, string) the third line of text (for the *_04 templates),
   "toast_text_img_name": (optional, string) the path for the image for the templates that include an image,
   "message_variation_id": (optional, string) used when providing a campaign_id to specify which message variation this message should be tracked under (must be a Windows Universal Push Message),
   "extra_launch_string": (optional, string) used to add deep linking functionality by passing extra values to the launch string
}
```

__Note__: For more information on using the `extra_launch_string` parameter for deep linking, see [Deep Linking with Windows Universal.] [37] For information regarding what a deep link is, please see our [FAQ Section][38].

### In-App Message Object Specification

See our [Guide to Best Practices for In-App Messages][36] for the details on the different types of in-app messages available.

```json
{
   "type" : (optional, string) one of "SLIDEUP" OR "MODAL" or "FULL", defaults to "SLIDEUP",
   // For "SLIDEUP" messages, old versions of the SDK only support some of the features, so set this to false to avoid sending messages that would not make sense as an older version of the slideups (i.e. includes an image that is referenced by the message). This is not applicable for "MODAL" or "FULL" messages, which are always only sent to new versions
   "also_send_to_slideup_only_versions" : (optional, boolean), defaults to true,
   "message" : (required, string) 140 characters max,
   // all colors should be 8-digit hex strings plus a leading "0x", for example "0xFF00AA88"
   // see http://developer.android.com/reference/android/graphics/Color.html for specifications
   "message_text_color" : (optional, string) hex value for colors, defaults to black or white depending on message type,
   "header" : (optional, string) the header shown, not shown if excluded,
   "header_text_color" : (optional, string) hex value for colors, defaults to black or white depending on message type,
   "background_color" : (optional, string) hex value for colors, defaults to black or white depending on message type,
   "close_button_color" : (optional, string) hex value for colors, defaults to black
   "slide_from" : (optional, string) "TOP" OR "BOTTOM" for "STANDARD" messages, defaults to "BOTTOM",
   "message_close" : (optional, string) "SWIPE" OR "AUTO_DISMISS", defaults to "AUTO_DISMISS",
   // icon should be 4-digit hex string without the leading "0x" from http://fortawesome.github.io/Font-Awesome/cheatsheet/
   // for example, "f042" for the first icon, fa-adjust [&#xf042;]
   // if both image_url and icon are present, image_url will be used
   "icon": (optional, string) Font Awesome icon hex value,
   "icon_color" : (optional, string) hex value for colors, defaults to white,
   "icon_background_color" : (optional, string) hex value for colors, defaults to blue,
   "image_url" : (optional, string) url for image to show when type is "FULL", overrides "icon" if both are present,
   "buttons" : (optional, Array of Button Objects) buttons to show, at most 2 allowed,
   "extras" : (optional, valid Key Value Hash), extra hash,
   // click actions and uri are not applicable if there are buttons and type is MODAL or FULL
   "ios_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "android_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "windows_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "windows8_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "kindle_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "android_china_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "web_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "ios_uri" : (optional, string) valid http or protocol uri,
   "android_uri" : (optional, string) valid http or protocol uri,
   "windows_uri" : (optional, string) valid http or protocol uri,
   "windows8_uri" : (optional, string) valid http or protocol uri,
   "kindle_uri" : (optional, string) valid http or protocol uri,
   "android_china_uri" : (optional, string) valid http or protocol uri,
   "web_uri" : (optional, string) valid http or protocol uri,
   "message_variation_id": (optional, string) used when providing a campaign_id to specify which message variation this message should be tracked under
}
```

The In-App Message Object should be included in `messages` when you would like to also deliver an in-app message with the given content to the targeted users.

####  Button Object

```json
{
   "text": (required, string) text shown on button,
   "text_color": (optional, string) hex value for colors, defaults to white,
   "background_color": (optional, string) hex value for colors, defaults to blue,
   "ios_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "android_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "windows_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "windows8_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "kindle_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "android_china_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "web_click_action" : (optional, string) "NONE" OR "NEWS_FEED" OR "URI", defaults to "NONE",
   "ios_uri" : (optional, string) valid http or protocol uri,
   "android_uri" : (optional, string) valid http or protocol uri,
   "windows_uri" : (optional, string) valid http or protocol uri,
   "windows8_uri" : (optional, string) valid http or protocol uri,
   "kindle_uri" : (optional, string) valid http or protocol uri,
   "android_china_uri" : (optional, string) valid http or protocol uri,
   "web_uri" : (optional, string) valid http or protocol uri,
}
```

### Email Object Specification

```json
{
  "app_id": (required, string) see App Identifier above,
  "subject": (optional, string),
  "from": (required, valid email address in the format "Display Name <email@address.com>"),
  "reply_to": (optional, valid email address in the format "email@address.com" - defaults to your app group's default reply to if not set),
  "body": (required unless email_template_id is given, valid HTML),
  "plaintext_body": (optional, valid plaintext, defaults to autogenerating plaintext from "body" when this is not set),
  "email_template_id": (optional, string) If provided, we will use the subject/body values from the given email template UNLESS they are specified here, in which case we will override the provided template,
  "message_variation_id": (optional, string) used when providing a campaign_id to specify which message variation this message should be tracked under,
  "extras": (optional, valid Key Value Hash), extra hash - for SendGrid customers, this will be passed to SendGrid as Unique Arguments,
  "headers": (optional, valid Key Value Hash), hash of custom extensions headers. Currently, only supported for SendGrid customers
}
```

__Note:__ An `email_template_id` can be retrieved from the bottom of any Email Template created within the dashboard. Below is an example of what this ID looks like:

![Email Template ID][31]

### Webhook Object Specification

```json
{
  "url": (required, string),
  "request_method": (required, string) one of "POST", "PUT", "DELETE", or "GET",
  "request_headers": (optional, Hash) key/value pairs to use as request headers,
  "body": (optional, string) if you want to include a JSON object, make sure to escape quotes and backslashes,
  "message_variation_id": (optional, string) used when providing a campaign_id to specify which message variation this message should be tracked under
}
```

##  Server Responses

If your POST payload was accepted by our servers, then successful messages will be met with the following response:

```json
{
  "message" : "success"
}
```

Note that success only means that the RESTful API payload was correctly formed and passed onto our push notification or email or other messaging services. It does not mean that the messages were actually delivered, as additional factors could prevent the message from being delivered (e.g., a device could be offline, the push token could be rejected by Apple's servers, you may have provided an unknown user ID, etc.)

If your message is successful but has non-fatal errors you will receive the following response:

```json
{
  "message" : "success", "errors" : [<minor error message>]
}
```

In the case of a success, any messages that were not affected by an error in the `errors` array will still be delivered. If your message has a fatal error you will receive the following response:

```json
{
  "message" : <fatal error message>, "errors" : [<minor error message>]
}
```

### Queued Responses {#messaging-queued}

During times of maintenance, Appboy might pause real-time processing of the API. In these situations, the server will return an HTTP Accepted 202 response code and the following body, which indicates that we have received and queued the API call but have not immediately processed it. All scheduled maintenance will be posted to [http://status.appboy.com](http://status.appboy.com) ahead of time.

```json
{
  "message" : "queued"
}
```

### Fatal Errors

The following status codes and associated error messages will be returned if your request encounters a fatal error. Any of these error codes indicate that no messages will be sent.

- 400 Bad Request - Bad syntax.
- 400 No Recipients - There are no external IDs or segment IDs or no push tokens in the request
- 400 Invalid Campaign ID - No Messaging API Campaign was found for the campaign ID you provided
- 400 Message Variant Unspecified - You provide a campaign ID but no message variation ID
- 400 Invalid Message Variant - You provided a valid campaign ID, but the message variation ID doesn't match any of that campaign's messages
- 400 Mismatched Message Type - You provided a message variation of the wrong message type for at least one of your messages
- 400 Invalid Extra Push Payload - You provide the "extra" key for either "apple_push" or "android_push" but it is not a dictionary
- 400 Max input length exceeded - Caused by:
  - More than 50 external ids
- 400 No message to send - No payload is specified for the message
- 400 Slideup Message Length Exceeded - Slideup message > 140 characters
- 400 Apple Push Length Exceeded - JSON payload > 1912 bytes
- 400 Android Push Length Exceeded - JSON payload > 4000 bytes
- 400 Bad Request - Cannot parse send_at datetime
- 400 Bad Request - `in_local_time` is true but `time` has passed in your company’s time zone
- 401 Unauthorized - Unknown or missing app group id
- 403 Forbidden - Rate plan doesn't support or account is otherwise inactivated
- 404 Not Found - Unknown App Group ID
- 429 Rate limited - Over rate limit
- 5XX - Internal server error, you should retry your request with exponential backoff

##  API Limits {#messaging-limits}

- You can post up to 50 events per request
- API access for enterprise customers is unlimited.  With the understanding that you can get whatever throughput your use case requires and you are able to pay for.
- API access for non-enterprise customers is limited to 100 requests per hour.
- If you have questions about API limits please contact your Customer Success manager.


## Sample Requests in Various Languages {#samples-messaging}

### Python {#messaging-python}

__Note__: This request can alternatively be completed using the external library [Requests][22].

```python
# Necessary built-in imports to be utilized later
import json
import urllib2

# Define your static variables (app group ID, request url)
request_url = 'https://api.appboy.com/messages/send'
app_group_id = 'your app group id'

# Determine which users you want to message
external_user_ids = ['external user id 1', 'external user id 2']

# Define the content type as a dictionary
headers_params = {'Content-Type':'application/json'}

# Define the contents of your message(s)
android_noti = {"alert": "your message",
                "title": "your message title"}
apple_noti = {"alert": "your message",
              "badge": 'remaining badge count (as an integer)'}
# Instantiate your messages variable as a dictionary
messages = {"android_push" : android_noti,
            "apple_push": apple_noti}

# Store the request data as a dictionary
data = {'app_group_id': app_group_id,
        'external_user_ids': external_user_ids,
        'messages' : messages}

# CAMPAIGN TRIGGER ENDPOINT VARIABLES ONLY

request_url = 'https://api.appboy.com/campaigns/trigger/send'
campaign_id = 'your campaign id'

data = {'app_group_id': app_group_id,
        'campaign_id': campaign_id,
        'external_user_ids': external_user_ids}

# END ENDPOINT-SPECIFIC VARIABLES

# Convert the data into JSON format
JSONdata = json.dumps(data)
# Create the request
req = urllib2.Request(request_url, JSONdata, headers_params)
# Open the request
f = urllib2.urlopen(req)
# Get the response code
response = f.read()
# Close the opened request
f.close()
# Check that the request worked correctly
print response
```

### Ruby (using REST Client & MultiJSON): {#ruby-messaging}

__Note__: This post request requires the downloading of the external gems [Rest Client][24] and [MultiJSON][23]

```ruby
# Required libraries to import
require 'rest-client'
require 'multi_json'

app_group_id = 'your app group id'

# Decide which users you wish to target
external_user_ids = ['external user id 1', 'external user id 2']

# Define the content type as a hash
headers_params = {'Content-Type'=>'application/json'}

# Define the contents of your messages
android_noti = {:alert => 'your message',
                :title => 'your message title'}
apple_noti = {:alert => 'your message',
              :badge => 'remaining badge count (as an integer)'}
# Instantiate the messages array        
messages = {'android_push' => android_noti,
            'apple_push' => apple_noti}

# Organize the data to send to the API as a hash
# comprised of your previously defined variables.
data = {:app_group_id => app_group_id,
        :external_user_ids => external_user_ids,
        :messages => messages}

# CAMPAIGN TRIGGER ENDPOINT VARIABLES ONLY

request_url = 'https://api.appboy.com/campaigns/trigger/send'
campaign_id = 'your campaign id'

# Organize the data to send to the API as a hash
# comprised of your previously defined variables.
data = {:app_group_id => app_group_id,
        :campaign_id => campaign_id,
        :external_user_ids => external_user_ids}

# END ENDPOINT-SPECIFIC VARIABLES

# Convert the data into JSON format
JSONdata = MultiJson.encode(data)
# Send and check the POST request
puts RestClient.post(request_url, JSONdata, headers_params)

```

### PHP {#php-messaging}

```php
<?php
$app_group_id = 'your app group ID';

// Determine the users you plan to message
$external_user_ids = array('external user id 1', 'external user id 2');

// Establish the contents of your messages array
$android_noti = array('alert' => 'your message',
                      'title' => 'your message title');
$apple_noti = array('alert' => 'your message',
                    'badge' => 'remaining badge count (as an integer)');
// Instantiate the messages array
$messages = array('android_push' => $android_noti,
                  'apple_push' => $apple_noti);

// Organize the data to send to the API as another map
// comprised of your previously defined variables.
$postData = array(
  'app_group_id' => $app_group_id,
  'external_user_ids' => $external_user_ids,
  'messages' => $messages,
);

// CAMPAIGN TRIGGER ENDPOINT VARIABLES ONLY

$request_url = 'https://api.appboy.com/campaigns/trigger/send';
$campaign_id = 'your campaign id';

// Organize the data to send to the API as another map
// comprised of your previously defined variables.
$postData = array(
  'app_group_id' => $app_group_id,
  'campaign_id' => $campaign_id,
  'external_user_ids' => $external_user_ids,
);

// END ENDPOINT-SPECIFIC VARIABLES

// Create the context for the request
$context = stream_context_create(array(
    'http' => array(
        'method' => 'POST',
        'header' => "Content-Type: application/json\r\n",
        'content' => json_encode($postData)
    )
));

// Send the request
$response = file_get_contents($request_url, FALSE, $context);

// Print the response to ensure a successful request
echo $response;

?>
```

{% endraw %}

[18]: https://dashboard.appboy.com/app_settings/api_settings/
[19]: {{ site.baseurl }}/iOS/#setting-user-ids
[22]: http://docs.python-requests.org/en/latest/ "Requests"
[23]: https://rubygems.org/gems/multi_json "multiJSON"
[24]: https://rubygems.org/gems/rest-client "Rest Client"
[25]: #email-object
[26]: {{ site.baseurl }}/REST_API/#api-campaigns
[27]: #sample-requests
[28]: {{ site.baseurl }}/Android/#silent-push-notifications
[29]: {{ site.baseurl }}/Android/#advanced-use-cases
[31]: {% image_buster /assets/shared/img/email_template_id.png %}
[32]: https://msdn.microsoft.com/en-us/library/windows/apps/hh761494.aspx
[33]: /academy/Scheduling_and_Organizing_Campaigns/#option-3-intelligent-delivery
[34]: #campaign-id
[35]: http://developer.android.com/training/cloudsync/gcm.html#collapse "Collapse Key Documentation"
[36]: /academy/Best_Practices/#in-app-message-types
[37]: {{ site.baseurl }}/Windows/Universal/#step-4-deep-linking-from-push-into-your-app "Windows Universal Deep Linking Integration"
[38]: {{ site.baseurl }}/FAQs/#what-is-deep-linking "What is Deep Linking?"
[39]: /academy/Scheduling_and_Organizing_Campaigns/#api-triggered-campaigns-server-triggered-campaigns
[40]: /academy/Scheduling_and_Organizing_Campaigns/#re-eligibility-with-api-triggered-campaigns
[41]: https://dashboard.appboy.com/app_settings/developer_console/activitylog/
