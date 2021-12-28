---
layout: post
title: "Using Google Forms for custom form responses"
tags: [Websites, Forms, Google Forms]
---

Most websites end up needing some sort of form submission. It might be a mailing list sign-up, a contract form, or submitting bug reports/feedback. For a static website (or if you just don't prefer to build any backend), there are quite a few services out there to help "collect" form responses. However, the free services I've found tend to be quite limited.

Google Forms is usually my go to option for creating any kind of form, but sometimes it's nice being able to create something a little more custom. Well, it turns out you can pretty easily design your own custom Google Forms, or even post submissions from your own backend to Google Forms. It's completely free and there are no limitations!

I created a [helper tool](/tools/extract-google-form-fields/) to easily extract the field names.

## Example: Custom newsletter sign-up form
Here's an example of how I added a mailing list sign-up form to my blog using Google Forms.

### 1) Create a new Google Form
Let's start by creating a new Google Form with two questions, one for the user's email address, and another field for the page they are currently on (we will populate this field automatically). The title and field names don't matter, since we will be creating a custom form.

*Important: If you add any multiple choice or dropdown question then you must submit the matching values, otherwise validation will fail!*

![](/assets/img/forms/form.png)

### 2) Prepare your form submission
To submit a response to a Google Form you need to send a HTTP POST request. The URL is simply your form's URL, but with `viewform` replaced with `formResponse`:

```
https://docs.google.com/forms/d/e/.../viewform
```

Replace `viewform` with `formResponse`:

```
https://docs.google.com/forms/d/e/.../formResponse
```

The POST body should be a standard URL-encoded form (`Content-Type: application/x-www-form-urlencoded`) with the corresponding field entries.

To find the field names, you can either inspect the Google Form page source code manually, or you can use this [helper tool](/tools/extract-google-form-fields/) I created to automatically extract the field names.

![](/assets/img/forms/extract.png)

### 3) Sending form submissions

Next choose how you want to submit your custom form responses.

#### HTML Form
The easiest method is to simply create a regular HTML form. This works great for static websites. All you need to do is to set the correct action URL and name your fields accordingly. In this example we will also inject the current page path into the page field (using Jekyll in my case).

```html
<form method="post" action="https://docs.google.com/forms/d/e/.../formResponse">
  <label>Email</label>
  <input type="email" name="entry.1074208666" placeholder="Email Address" />

  <!-- Let's include a hidden field containing our current path -->
  <input type="hidden" name="entry.1363556558" value="{% raw %}{{ page.url }}{% endraw %}" />

  <button type="submit">Subscribe</button>
</form>
```

![](/assets/img/forms/example.png)

Tip: For form submissions you'll be redirected to the Google Form confirmation page. You can customize the message (and perhaps include a link back to your website) under `Settings > Presentation > Confirmation message`

![](/assets/img/forms/custom-message.png)


#### Server-side
For a completely custom submission flow (including the confirmation page), or if you need to do any custom post-processing, you can also send the form submission from your own backend. Here's an example of how to do that using node.js:

```js
import got from 'got';

app.post('/subscribe', (req, res) => {
  const formAction = 'https://docs.google.com/forms/d/e/.../formResponse';
  const form = { 'entry.1074208666': req.body.email };
  got.post(formAction, { form });
});
```

#### *What about AJAX?*
*Note: Unfortunately a client-side AJAX submission (e.g. using the browser's `fetch` API) directly to the Google Form won't work due to the CORS policy.*

## Email notifications

Google Forms even supports email notifications for new responses. This setting is located under `Responses > ... > Get email notifications for new responses`:

![](/assets/img/forms/email-notifications.png)

You could also hook up something like Zapier to your Google Form to automate notifications or other workflows.

## Bonus: Pre-filling

Another useful feature is that you can generate a pre-filled link to a Google Form. Simply append the field entries as a query string to your form URL.

```
https://docs.google.com/forms/d/e/.../viewform?entry.1074208666=foo@bar.com
```
