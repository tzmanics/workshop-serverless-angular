# Setup & Deployment

In order to jump into coding as soon as we can, we'll use a template project that uses [Scully](https://scully.io) to pre-render a few pages for us. All of the sections of the workshop will start with a clean, fresh version of this template instead of building off of one another. This makes it so you can use each section individually but feel free to keep building upon the base template in this workshop.

We will then deploy this template to [Netlify](https://ntl.fyi/3fN1SuB). This process will let us see the project live but also setup a git workflow by automatically setting up CI/CD: each time we push code it will trigger a new build and deployment on Netlify. We will also be taking advantage of [Netlify Edge](https://ntl.fyi/3fPWU0f) which is a globally-distributed CDN ([Content Delivery Network](https://jamstack.org/glossary/cdn/)) to make our site more reliable thanks to the redundancy of the redirect knowledge on the CDN nodes.

This is what the page will look like:

![the template live](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617815938/live-template_dbv70w.gif 'The template live')

> 🧠 We can also pre-render using Angular Universal, I have a whole [blog post on that too!](https://ntl.fyi/3fPZXFz)

Let's get it started in here!

## Pre-rendered Template Using Scully

The project can either be cloned locally from the command line (passing in the new project name):

`git clone https://github.com/tzmanics/serverless-angular-starter serverless-angular`

Or we can click the 'Use this template' button from [the repo's homepage](https://github.com/tzmanics/serverless-angular-starter) and then clone it locally.

![the template home page](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618153440/Screen_Shot_2021-04-11_at_11.01.21_AM_1_b7ggu3.jpg 'The template home page')

Whichever approach we take we'll need to install all the dependencies locally with npm. Change into the project directory and run:

`npm install`

### The Name Game

With Angular projects, the name of the project is used all over the place. We don't want to have any confusion or any remnants of the project so we will replace `serverless-angular-starter` with the new project name (in this case it's `serverless-angular`).

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

The easiest approach is to find all instances of `serverless-angular-starter` and replace all with your text editor. There will be no bad side effects using this specific search & replace. [Here is a link](https://code.visualstudio.com/docs/editor/codebasics#_find-and-replace) that shows you how to do it with VS Code.

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

![netlify init output](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618156458/Screen_Shot_2021-04-11_at_11.46.22_AM_w1ytbi.jpg '`netlify init` output')

Once this is set up, we can commit the changes we made when we changed this project name.

```bash
git add .
git commit -m 'changes project name'
git push --set-upstream main
```

This push will also trigger a new build, so we'll have the new information up on GitHub and live, yay! To check out the live version we can use the Netlify CLI again by using the `open` command. This will open the browser to the project's dashboard.

`ntl open`

![project dashboard](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1618157040/Screen_Shot_2021-04-11_at_12.00.22_PM_1_faz604.jpg 'screenshot of the project dashboard')

We'll be coming back to this dashboard a lot through the workshop. Right now, we can find the live site's url at the top of the page once the project is done building. If the url isn't there yet, we can check the 'Deploys' tab to see how the build is progressing.

When the live site is ready we can roam around the template site to see what exists now. This will be the jump-off point for all the other sections of the workshop, but again, feel free to continuously add the new features to this base site.

## Resources for the Road

- [Scully.io, Angular's Static Site Generator](https://scully.io)
- [Netlify Edge](https://ntl.fyi/3fPWU0f)
- [Pre-rendering with Angular Universal](https://ntl.fyi/3fPZXFz)
- [Angular Universal](https://angular.io/guide/universal)
