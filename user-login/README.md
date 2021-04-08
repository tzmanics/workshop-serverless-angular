# User Login

Using serverless functions and Jason Web Tokens ([JWT]()) we're going to make it so users can log in with their emails or using an external provider to authenticate. We'll use [Netlify Identity]() to do the heavy lifting.

## The Starting Gate

We'll be using this [project template]() that uses Angular Universal to pre-render the page. If you don't already have this set up you can learn how to do it from [this part of the workshop]() or [this blog post]().

We will also be using the [Netlify Identity]() service which is free for 1k users per month. There's a ton that we can do with authentication just using the Identity dashboard and that's what we'll cover in this section. We can do even more when we set up webhooks and other services, like [manage subscriptions and gate content](https://ntl.fyi/3cW3akX), but we'll cover that later.
