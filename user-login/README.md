# User Login

Using serverless functions and Jason Web Tokens ([JWT](https://jwt.io/introduction)) we're going to make it so users can log in with their emails or using an external provider to authenticate. We'll use [Netlify Identity](https://ntl.fyi/3mzlPXf) to do the heavy lifting.

Here are some resources to help you along the way:

- 🐙 [Project repo: Serverless Angular Auth](https://github.com/tzmanics/serverless-angular-auth)
- ▶️ This button will automiatically deploy ⬆️ repo to Netlify:
  [![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/tzmanics/serverless-angular-auth&utm_source=github&utm_medium=serverless-angular-workshop&utm_campaign=devex-tzm)

## The Starting Gate

We'll be using this [project template](https://github.com/tzmanics/serverless-angular-starter) that uses Angular Universal to pre-render the page. If you don't already have this set up you can learn how to do it from [this part of the workshop](https://github.com/tzmanics/workshop-serverless-angular/blob/main/setup-and-deployment/README.md).

We will also be using the [Netlify Identity](https://ntl.fyi/3mzlPXf) service which is free for 1k users per month. There's a ton that we can do with authentication just using the Identity dashboard and that's what we'll cover in this section. We can do even more when we set up webhooks and other services, like [manage subscriptions and gate content](https://ntl.fyi/3cW3akX), but we'll cover that later.

## Adding a Log In Button

To add log in and log out functionality we only need to add two lines of code and click a button. Let's start with the code.

In the `index.html` file we need to add a script tag linking to the Netlify Identity widget

`src/index.html`

```html
...
<head>
  <meta charset="utf-8" />
  <title>Angular Universal Pre-Render</title>
  <base href="/" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <link rel="icon" type="image/x-icon" href="favicon.ico" />
  <link rel="preconnect" href="https://fonts.gstatic.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" />
  <link
    href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;1,300&family=Poiret+One&display=swap"
    rel="stylesheet"
  />
  <!-- new code starts here -->
  <script
    type="text/javascript"
    src="https://identity.netlify.com/v1/netlify-identity-widget.js"
  ></script>
</head>
...
```

Next, we need to add the button to the UI. We're going to replace current content on the account dashboard page with the log in button. For now that is all that will be there. Later we will populate the page with user data once the user is logged in.

`src/app/user-dashboard/user-dashboard.component.html`

```html
<div class="user-dashboard">
  <div class="login" data-netlify-identity-button></div>
</div
```

### Styling

We have a functional button but let's make it look nice as well. When adding this button I realized a lot of the buttons on the page have a lot of the same attributes, so I added the common `a` tag styles to the main style sheet:

`src/style.css`

```css
... a {
  color: black;
  display: inline-block;
  font-family: 'Poiret One', sans-serif;
  font-size: 25px;
  position: relative;
  text-decoration: none;
  text-transform: lowercase;
}
a:hover {
  background-color: rgba(143, 188, 143, 0.2);
}
a.active {
  background-color: linen;
}
a:visited {
  color: black;
}
```

[🐙 This commit](https://github.com/tzmanics/serverless-angular-auth/commit/8c2215a38482082bd489e1ce067d862133e29231) shows where I removed the unneeded styles from the other style sheets. I also added some styling to align the text in the center of the dashboard page.

`src/app/user-dashboard/user-dashboard.component.css`

```css
.user-dashboard {
  font-size: 40px;
  margin: 0 auto;
  text-align: center;
  width: 80%;
}
...
```

The log in button in the dashboard now looks like this:

![log in button](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618187747/Screen_Shot_2021-04-11_at_8.35.04_PM_nw37zr.jpg 'log in button in dashboard')

That's great and all but if we click it now, we'll get a pop-up with an error. This is because we still need to enable Netlify Identity. We can do this in out project's Identity dashboard.

## Enabling Netlify Identity & Customization

To find the Identity settings we can go to the project dashboard (`ntl open` from the terminal) and click the 'Identity' tab, then select 'Enable Identity'.

![identity dashboard](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618188848/Screen_Shot_2021-04-11_at_12.00.22_PM_2_tstfcq.jpg 'identity dashboard and enable button')

Now we're able to click the button and get a pop-up window allowing users to sign up and log in. There is already email validation hooked up too.

![defualt sign up and log in settings](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618190622/2021-04-11_21-09-14.2021-04-11_21_23_17_tppg6x.gif 'a gif of default log in and sign up settings')

Currently a user could sign up using an email address and Netlify Identity would send an email with the app name asking the user to verify their email (we can turn the email validation off if we don't need it). We want to give the user more options to log in and we also want to customize the email that gets sent to the user. To the Identity dashboard we go!

### External Authentication Providers

To start out we can leverage external providers to authenticate users. If we go to the Identity dashboard and click 'Settings and usage' then scrool down, we will find a section labeled 'External providers'.

![external providers](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618192270/Screen_Shot_2021-04-11_at_9.02.37_PM_g9fped.jpg 'a screenshot of the external providers section')

We can use BitBucket, GitHub, GitLab, and Google. For this project we'll just add GitHub and Google. There is an option to use custom OAuth credentials but we'll just use the default settings for now. We can change this in the future too by clicking the 'Options' button that shows up once we add an external provider to the project.

![default external provider settings](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618192449/Screen_Shot_2021-04-11_at_9.02.47_PM_yef0yw.jpg 'default external provider settings')

Now we can see the user has an option to either sign up or login with their email address or by authenticating with GitHub or Google.

![external provider log in](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618199959/Screen_Shot_2021-04-11_at_11.54.28_PM_urwfeh.jpg 'screenshot of external provider log in ui')

## Managing Users

To test out what we can manually do with users, we can add some test users. We can do this by either using our own emails with the '+' format so we can have duplicates (yourEmail+randomWord@gmail.com). To make it easier we could also turn off the email verification under Identity dashboard > Settings & Usage > Confirmation template > 'Allow users to sign up without verifying their email address'.

![canceling email verificationi](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618202925/Screen_Shot_2021-04-12_at_12.46.30_AM_nbk0ce.jpg 'canceling email verification')

Even if the email isn't verified the users will still show up on our list of users. 'But where is this list?', you may ask. It's on the home page of the Identity tab for the project.

![user list](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618203141/Screen_Shot_2021-04-12_at_12.50.33_AM_1_t8ap4r.jpg 'screenshot of the identity dashboard where users are displayed')

If we click on the arrow at the end of each user's row, we can drill down into each user and even manually add or change roles. We can use these later to set permissions and even gate content.

![changing user roles](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618203565/Screen_Shot_2021-04-12_at_12.55.00_AM_au1djb.jpg 'the ui for changing user roles')

### Other Settings

There are a few other settings that we have the ability to customize but won't get into today:

- email customization with template (including invitation, confirmation, password recovery, email change emails)
- webhooks that send a POST on validate, sign up, or login
- and moar!

## Logging Out

Ok, we can log out of this section (get it?) because we have sign up and log in workflow working with serverless functions and stateless JWT authentication. In the [user data](https://github.com/tzmanics/workshop-serverless-angular/blob/main/user-data/README.md) section we'll look at how to take advantage of the Netlify Idenitiy webhooks to get and use user data. See you there!
