# workshop-serverless-angular

٩꒰｡•‿•｡꒱۶ Creating an Angular eCommerce site utilizing serverless deployment and functions.

> 💡 To get the most out of the workshop time, please complete the [setup section](#the-setup) before joining the workshop.

# Serverless Angular: Deployment, Dynamic Data, Auth, and More!

With our sites requiring more data and our users requiring faster speeds, we need a different approach to our app development without compromising our development process. As developers, we want to be able to focus on evolving and creating the features of our apps but not be consumed by managing infrastructure. Moving to serverless deployment and utilizing serverless functions can help deliver our apps and data fast while working in an easy to manage and maintain workflow. Here’s how! In this workshop we’ll begin with a pre-rendered eCommerce template made with Angular 11. Then work together to add:

- a git workflow that sets up continuous deployment to a globally distributed Content Delivery Network (CDN)
- stateless auth with JSON Web Tokens (JWTs) & serverless functions to allow for login
- setting up a headless CMS to input data
- a database that we use serverless functions to store and retrieve data
- & the ability to take payments by setting up serverless functions to talk to the Stripe API

## This Is How We Do It ヾ(^ ^ゞ

This repo doesn't need to be cloned, it's purpose is to guide you through each section of the workshop. My hope is that you'll be able to use this repo as a resource even when you're not in the workshop with me. In the workshop we'll be going in this order:

0. [Introduction](/introduction/README.md) (EDT 12:00 - 12:30)
1. [Project Setup & Deployment](/setup-and-deployment/README.md) (EDT 12:30 - 13:00)
2. [User Login using Netlify Identity](/user-login/README.md) (EDT 13:00 - 13:30)
3. [Building a Products List with Sanity.io Headless CMS](product-list/README.md) (EDT 13:30 - 15:00)
4. BREAK TIME ƪƪ’▿’) (‘▿’ʃʃ (EDT 15:00 - 16:00)
5. [Storing & Retrieving User Data with Hasura](user-data/README.md) (EDT 16:00 - 17:30)
6. [Taking Payments using Stripe](payments/README.md) (EDT 17:30 - 18:30)
7. Wrap Up & Questions (EDT 18:30 - 19:00)

## The Setup

In order to get the most out of this workshop we need to hit the ground coding, so I've made a prep sheet for you. Feel free to reach out to me via email at tara [@] netlify [dot] com 👍.

## Coding Set Up

You are free to use whatever code editor you prefer. Using [VS Code](https://code.visualstudio.com/download) will allow us to code together if needed, but again, use the editor that's most comfortable.

- [Link for VS Code Install](https://code.visualstudio.com/download)

### Angular Requirements

A few things to create our example will work best with the Angular v11 CLI. If you don't have the Angular CLI installed already when you do install it will give you the latest version. If not, you can check your version by running the `ng --version` command in your command line to see if you need to update.

- To install the Angular CLI run

```bash
npm install -g @angular/cli
```

- [Link on updating to v11.](https://update.angular.io/)

Once you have the Angular CLI installed and up to date, you may want to run the project generation command `ng new` to make sure everything works ok.

### Version Control

I know _you never_ make mistakes BUT just in case, it's always good to commit and commit often. If you don't already have a git account somewhere, may I recommend signing up for GitHub.

- [Link to make a GitHub account.](https://github.com/join)

### Hosting

For this workshop we'll be using [Netlify](https://www.netlify.com/?utm_source=github-repo&utm_medium=angular-workshop_tzm&utm_campaign=devex) to throw our site online. If you don't already have an account you can sign up for a free one. It will then hook up to your git account and allow us to make quick work of our [CI/CD](https://www.netlify.com/products/build/?utm_source=github-repo&utm_medium=angular-workshop_tzm&utm_campaign=devex) setup.

- [Link to setup Netlify account.](https://app.netlify.com/signup?utm_source=github-repo&utm_medium=angular-workshop_tzm&utm_campaign=devex)

> Heads up, I work for Netlify. Even before I did I always recommended using them because I LOVE their developer experience. Which, in turn, is why I happily work on their developer experience team. Just wanted to be transparent 👍.

### Other Services

We'll also be using a few other services, so it will speed up the process if you make accounts with them as well. They are all free to start so you won't have to worry about adding a credit card with any.

- [Sanity.io](https://manage.sanity.io/) headless CMS
- [Hasura](https://cloud.hasura.io/signup) GraphQL-based database
- [Stripe](https://dashboard.stripe.com/register) account

## Checklist

Here's a little tldr; of what to have before coming to the workshop. Again, this helps us to really get the most out of the workshop.

- [ ] A code editor like VS Code, Sublime, sick vim setup, etc.
- [ ] Angular CLI v9
- [ ] A git account
- [ ] A Netlify account
- [ ] Other service accounts
- [ ] A good knock-knock joke (just in case)

## Pre-Workshop Resources

If you want to do some learning before we get started in the workshop, here are some great resources on the Jamstack.

- [What's Angular in the Jamstack? It Sounds Delicious!](https://www.netlify.com/blog/2019/10/30/whats-angular-in-the-jamstack-it-sounds-delicious/?utm_source=github-repo&utm_medium=angular-workshop_tzm&utm_campaign=devex)
- [What is the Jamstack](https://dev.to/shortdiv/what-is-the-jamstack-15i2)
- [FREE BOOK: Modern Web Development on the Jamstack](https://www.netlify.com/oreilly-jamstack/#download)
- [Jamstack.org](https://jamstack.org/)
