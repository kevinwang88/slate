---
nav_title: Email Sync
---
# Email Sync

Users' email subscription status can be updated and retrieved via Appboy using a RESTful API. You can use the API to setup bi-directional sync between Appboy and other email systems or your own database.

## API Specification

All API requests are made over HTTPS to `https://api.appboy.com/`. Below are the paths for each email sync endpoint:

| URL | HTTP Verb | Functionality |
| ---------------------| --------------- |
| /email/unsubscribes | GET | Retrieving Objects|
| /email/status | POST | Creating Objects |
| /email/bounce/remove | POST | Removing Objects |
| /email/spam/remove | POST | Removing Objects |

## Querying Unsubscribed Emails

`GET https://api.appboy.com/email/unsubscribes`

| Parameter | Required | Data Type | Description |
| ---------------------| --------------- |
| `app_group_id` | Yes | String | see App Group Identifier in Parameter Definitions |
| `start_date` | No | String in YYYY-MM-DD format| Start date of the range to retrieve unsubscribes, must be earlier than end_date. This will be taken as UTC time by the API. |
| `end_date` | No | String in YYYY-MM-DD format | End date of the range to retrieve unsubscribes. This will be taken as UTC time by the API |
| `limit` | No | Integer | Optional field to limit the number of results returned. Defaults to 100 |
| `offset` | No | Integer | Optional beginning point in the list to retrieve from |
| `email` | No | String | If provided, we will return whether or not the user has unsubscribed |

__Note:__ You must provide either an `email` or a `start_date`, and an `end_date`.

## Changing Email Subscription Status

`POST https://api.appboy.com/email/status`

This endpoint allows you to set the email subscription state for your users. Users can be "opted_in", "unsubscribed", or "subscribed" (not specifically opted in or out).

You can set the email subscription state for an email address that is not yet associated with any of your users within Appboy. When that email address is subsequently associated with a user, the email subscription state that you uploaded will be automatically set.


| Parameter | Required | Data Type | Description |
| ---------------------| --------------- |
| `app_group_id` | Yes | String | see App Group Identifier in Parameter Definitions |
| `email` | Yes | String or Array | String email address to modify, or an Array of up to 50 email addresses to modify. |
| `subscription_state` | Yes | String | Either “subscribed”, “unsubscribed”, or “opted_in”. |

## Removing Hard Bounces

`POST https://api.appboy.com/email/bounce/remove`

This endpoint allows you to remove email addresses from your Appboy bounce list. We will also remove them from the bounce list maintained by your email provider.

| Parameter | Required | Data Type | Description |
| ---------------------| --------------- |
| `app_group_id` | Yes | String | see App Group Identifier in Parameter Definitions |
| `email` | Yes | String or Array | String email address to modify, or an Array of up to 50 email addresses to modify. |

## Removing Spam

This endpoint allows you to remove email addresses from your Appboy spam list. We will also remove them from the spam list maintained by your email provider.

`POST https://api.appboy.com/email/spam/remove`

| Parameter | Required | Data Type | Description |
| ---------------------| --------------- |
| `app_group_id` | Yes | String | see App Group Identifier in Parameter Definitions |
| `email` | Yes | String or Array | String email address to modify, or an Array of up to 50 email addresses to modify. |

### Example Unsubscribe CURL

The following example CURL demonstrates how to unsubscribe a user from receiving email via the Appboy APIs:

```
curl -X POST -H "Content-Type: application/json" -d '{"app_group_id":"YOUR_APP_GROUP_ID","email":"EMAIL_TO_UNSUBSCRIBE","subscription_state":"unsubscribed"}' https://api.appboy.com/email/status
```

[4]: {{ site.baseurl }}/REST_API/#User-Data
