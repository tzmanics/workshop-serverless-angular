# Setup & Deployment

In order to jump into coding as soon as we can, we'll use a template project that uses [Angular Universal](https://angular.io/guide/universal) to pre-render a few pages for us. All of the sections of the workshop will start with a clean, fresh version of this template instead of building off of one another. This makes it so you can use each section individually but feel free to keep building upon the base template in this workshop.

We will then deploy this template to [Netlify](https://ntl.fyi/3fN1SuB). This process will let us see the project live but also setup a git workflow by automatically setting up CI/CD: each time we push code it will trigger a new build and deployment on Netlify. We will also be taking advantage of [Netlify Edge](https://ntl.fyi/3fPWU0f) which is a globally-distributed CDN ([Content Delivery Network](https://jamstack.org/glossary/cdn/)) to make our site more reliable thanks to the redundancy of the redirect knowledge on the CDN nodes.

This is what the page will look like:

![the template live](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617815938/live-template_dbv70w.gif 'The template live')

Let's get it started in here!

## Pre-rendered Template Using Angular Universal

> 📓 There is a [whole blog post on creating this template](https://hubs.ly/H0GZlXP0) too! We'll then use [Netlify]() to deploy the template

The project can either be cloned locally from the command line (passing in the new project name):

`git clone https://github.com/tzmanics/angular-universal-pre-render serverless-angular`

Or we can click the 'Use this template' button from [the repo's homepage](https://github.com/tzmanics/angular-universal-pre-render).

![the template home page](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617821608/Screen_Shot_2021-04-07_at_2.51.41_PM_vpsiqm.jpg 'The template home page')

Whichever approach we take we'll need to install all the dependencies locally with npm. Change into the project directory and run:

`npm install`

### The Name Game

With Angular projects, the name of the project is used all over the place. We don't want to have any confusion or any remnants of the project so we will replace `angular-universal-pre-render` with the new project name (in this case it's `serverless-angular`).

Instances of the project need changed in the following files:

- `angular.json`
- `netlify.toml`
- `package.json`
- `server.ts`
- `server.ts`
- `src/app/app.component.spec.ts`
- `src/app/app.component.ts`
- `e2e/src/app.e2e-spec.ts`
- `karma.conf.js`

The easiest approach is to find all instances of `angular-universal-pre-render` and replace all with your text editor. There will be no bad side effects using this specific search & replace. [Here is a link](https://code.visualstudio.com/docs/editor/codebasics#_find-and-replace) that shows you how to do it with VS Code.

Just to make sure everything is in order it's good to serve the project locally by running the command

`ng serve`

Head over to [`http://localhost:4200/`](http://localhost:4200/) to see the working project locally.

## Deploying the Project to Netlify

We can now hook this site up to Netlify so whenever we push a new commit it will trigger a new deploy (CI/CD). Having it live we can see what our users will experience when they request the site. The template project already has a Netlify configuration file, [`netlify.toml`](https://github.com/tzmanics/angular-sanity/blob/main/netlify.toml). We can just run a few commands using the Netlify CLI.

```bash
npm install netlify-cli -g
netlify login
netlify init
```

First, we install the Netlify CLI globally and log in. The `netlify init` command will ask a few questions and set the command and publish settings with the project's `netlify.toml` file. This command will also trigger a deploy of the project. We can run `netlify open` to get to the project dashboard, to see when the build has been published.

![netlify init output](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617822696/Screen_Shot_2021-04-07_at_3.11.07_PM_v1rjmq.jpg '`netlify init` output')

Once this is set up, we can commit the changes we made when we changed this project name.

```bash
git add .
git commit -m 'changes project name'
git push --set-upstream main
```

This push will also trigger a new build, so we'll have the new information up on GitHub and live, yay!

## Resources for the Road

- [Angular Universal](https://angular.io/guide/universal)
- [Netlify Edge](https://ntl.fyi/3fPWU0f)
- [Pre-rendering with Angular Universal](https://ntl.fyi/3fPZXFz)
