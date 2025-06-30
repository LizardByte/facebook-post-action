# facebook-post-action

> [!WARNING]
> This action is deprecated and will not be maintained anymore.
> Please use our [LizardByte/actions](https://github.com/LizardByte/actions) monorepo instead.

[![GitHub Workflow Status (CI)](https://img.shields.io/github/actions/workflow/status/lizardbyte/facebook-post-action/ci.yml.svg?branch=master&label=CI%20build&logo=github&style=for-the-badge)](https://github.com/LizardByte/facebook-post-action/actions/workflows/ci.yml?query=branch%3Amaster)
[![Codecov](https://img.shields.io/codecov/c/gh/LizardByte/facebook-post-action.svg?token=LiDWK90cNO&style=for-the-badge&logo=codecov&label=codecov)](https://app.codecov.io/gh/LizardByte/facebook-post-action)

GitHub Action for posting to a facebook page or group.

## 🎒 Prep Work
1. Get a facebook permanent access token (explained below) using a facebook account that owns the page where you want to post messages.
2. Find the ID of the page or group that you want to post messages in (explained below).

## 🖥 Workflow example
```yaml
name: Facebook Post Action

on:
  release:
    types: [published]
jobs:
  notify:
    runs-on: ubuntu-latest

    steps:
      - name: facebook-post-action
        uses: LizardByte/facebook-post-action@master
        with:
          access_token: ${{ secrets.FACEBOOK_ACCESS_TOKEN }}
          fail_on_eror: True
          message: |
            ${{ github.event.repository.name }} ${{ github.ref }} Released

            ${{ github.event.release.body }}
          page_id: ${{ secrets.FACEBOOK_PAGE_ID }}
          url: ${{ github.event.release.html_url }}
```

## 🤫 Inputs

- **ACCESS_TOKEN**: The permanent facebook access token
- **FAIL_ON_ERROR**: Fail the workflow on error.
  Group posts will fail if the facebook app is not installed to the group; however, the message will be posted,
  setting this to False will allow the workflow to be successful.
- **MESSAGE**: The content to post
- **PAGE_ID**: The page ID where you want to post
- **URL**: The url to embed with the post (optional)

## 👥 How to get a Facebook permanent access token

Following the instructions laid out in Facebook's [extending page tokens documentation][2] I was able to get a page
access token that does not expire.

I suggest using the [Graph API Explorer][3] for all of these steps except where otherwise stated.

### 1. Create Facebook App

**If you already have an app**, skip to the next step.

1. Go to [My Apps][4].
2. Click "+ Add a New App".
3. Set up a website app.

You don't need to change its permissions or anything. You just need an app that won't go away before you're done with
your access token.

### 2. Get User Short-Lived Access Token

1. Go to the [Graph API Explorer][3].
2. Select the application you want to get the access token for (in the "Meta App" drop-down menu).
3. In the "Add a Permission" drop-down, search and check "pages_manage_posts", "pages_show_list",
   and "publish_to_groups". Publishing to groups requires an approved app.
4. Click "Generate Access Token".
5. Grant access from a Facebook account that has access to manage the target page. Note that if this user loses access,
   the final, never-expiring, access token will likely stop working.

The token that appears in the "Access Token" field is your short-lived access token.

### 3. Generate Long-Lived Access Token

Following [these instructions][5] from the Facebook docs, make a GET request to

```
https://graph.facebook.com/oauth/access_token?grant_type=fb_exchange_token&client_id=**{app_id}**&client_secret=**{app_secret}**&fb_exchange_token=**{short_lived_token}**
```

entering in your app's ID and secret and the short-lived token generated in the previous step.

You **cannot use the Graph API Explorer**. For some reason it gets stuck on this request.
I think it's because the response isn't JSON, but a query string. Since it's a GET request,
you can just go to the URL in your browser.

The response should look like this:

```json
{"access_token":"**ABC123**","token_type":"bearer","expires_in":5183791}
```

"ABC123" will be your long-lived access token. You can put it into the [Access Token Debugger][7] to verify.
Under "Expires" it should have something like "2 months". If it says "Never", you can skip the rest of the steps.

### 4. Get User ID

Using the long-lived access token, make a GET request to

```
https://graph.facebook.com/me?access_token=**{long_lived_access_token}**
```

The `id` field is your account ID. You'll need it for the next step.

### 5. Get Permanent Page Access Token

Make a GET request to

```
https://graph.facebook.com/**{account_id}**/accounts?access_token=**{long_lived_access_token}**
```

The JSON response should have a `data` field under which is an array of items the user has access to.
Find the item for the page you want the permanent access token from. The `access_token` field should have your
permanent access token. Copy it and test it in the [Access Token Debugger][7]. Under "Expires" it should say "Never".

[2]:https://developers.facebook.com/docs/facebook-login/access-tokens#extendingpagetokens
[3]:https://developers.facebook.com/tools/explorer
[4]:https://developers.facebook.com/apps/
[5]:https://developers.facebook.com/docs/facebook-login/access-tokens#extending
[6]:https://luckymarmot.com/paw
[7]:https://developers.facebook.com/tools/debug/accesstoken

## 👥 How to get a Facebook page ID

To find your Page ID:

1. From News Feed, click Pages on the left side menu.
2. Click your Page's name to go to your Page.
3. Select the About tab. If you don't see the About tab, click the More dropdown.
4. Find your Page ID in the Page Transparency section.
