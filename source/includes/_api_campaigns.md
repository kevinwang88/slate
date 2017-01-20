---
nav_title: API Campaigns
platform: REST APIs
---
# API Campaigns
Campaigns sent through the [Messaging API][1] can have the same detailed reporting and retargeting options as campaigns created on the dashboard. This section of the documentation will detail how to generate a `campaign_id` to include in your API calls and take advantage of this feature.

## Step 1: Click "API Campaigns" on the "Campaigns" page
Navigate to the [Appboy Campaigns page][2] in your company dashboard and click "Create Campaign" then click "API Campaigns"

![API Campaigns][3]

## Step 2: Configuring API Campaigns
1. Add a descriptive title so you can find the results on our [campaigns page][2] after you've sent your messages.
2. Click the "Add Message" button and add the messages types which will be included in your API campaign.
3. Optionally add a conversion event to track user conversions on a specific action or campaign goal.
4. Click "Save Campaign" and you're set to begin your API campaign.
5. Include the generated `campaign_ID` fields with your API request where noted in the [Messaging API - Send Endpoint Spec][1]

![Configuring API Campaigns][4]

[1]: {{ site.baseurl }}/REST_API/#send-endpoints "Send Endpoint : Messaging API"
[2]: https://dashboard.appboy.com/engagement/campaigns/ "Campaigns Page"
[3]: {% image_buster /assets/shared/img/api_campaign_create.png %} "Create API Campaign Button"
[4]: {% image_buster /assets/shared/img/api_campaign_keys_new.png %} "API Campaign Keys Page"
