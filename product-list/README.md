# Building a Product List with Sanity.io Headless CMS

A headless Content Management System (CMS) separates where the data is displayed (the "head") from where it stored and entered (the "body"). This approach makes it so data can be entered without a commitment to how it will get displayed. It also allows for teams of varying technical expertise to work seamlessly together. For instance, if the content creation team can stick to entering data in a form version that they would be used to but then the project engineers are free to manipulate and use the entered data in their favorite code editors.

To get a handle on how this works and how it is setup, we'll be creating a Sanity.io instance for data entry. Then we will display all the items on a page using Netlify Functions to grab the data, Angular to display the data, and webhooks to refresh the page when new data is entered.

This is what the page will look like:
![final product page](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617852205/template_sv0io2.jpg 'the final product page')

\</talking-time>

\<coding-time>

## Setting Up Sanity

It's time to set up the Sanity.io instance locally. We can do this with the Sanity.io CLI first by installing the CLI tool then initializing a Sanity.io project:

```bash
npm install -g @sanity/cli
sanity init
```

When we run `sanity init` it will make sure we're logged in (and have an account) then it will step through prompts to set up a new project.

![sanity init output](https://cdn.netlify.com/ba702acad9a0a7363e5ce2948297489f237a5fa3/6ef9d/img/blog/sanity-init.jpg '`sanity init` ouput')

We'll create a new project and name it `backend`, say yes to the defaults, but use the `Clean project with no redefined schemas`. Using the clean product will allow us to write custom schemas without too much overhead that may be confusing.

Change into the Sanity.io instance `backend` folder and run `sanity start` to start the UI up locally.

```bash
cd backend
sanity start
```

Then head to [`http://localhost:3333/`](http://localhost:3333/) to see what we have to work with. It's nothing. This is right because we haven't added any schemas yet!

![results at localhost:3333](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617853159/empty-schema_iq6jkf.jpg 'localhost:3333 output')

### Deploying the Sanity.io Desk

Although we can host our Sanity.io instance on Netlify, I want to show the built-in way. From the terminal, we can run `sanity deploy` and it will prompt for a name then set the live instance at https://<project name>.sanity.desk. Once made, this information can be found at the top of the project page at [sanity.io](https://sanity.io).

![sanity.io dashboard](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617853196/sanity-deploy_hczci1.jpg 'Sanity.io dashboard')

Anytime we want to update the deployed version of the Sanity.io instance, we'll need to run the `sanity deploy` command again.

> 🐙 This is a good place to push the new code up, BUT make sure to add `backend/node_modules` & `backend/dist` to the project's [`.gitignore` file](https://github.com/tzmanics/angular-sanity/blob/main/.gitignore) in the root directory!

## Coding Out the Schemas

It's time to make the custom schemas and update the main schema file. Sanity.io takes this information and uses it to model the information and also make the UI for the CMS.

> 📚 Learn more about Sanity.io schemas [from their guides](https://www.sanity.io/guides/how-to-configure-schemas) and [their documentation](https://www.sanity.io/docs/schema-types).

For now, we will look at the code we need to add and why we're adding each different schema.

### Block Content

We want to be able to have a section for each product where we can write more about the product. The block content will allow for the person entering information to edit text, add bulleted lists, headers, and some format.

[`backend/schemas/blockContent.js`](https://github.com/tzmanics/angular-sanity/blob/main/backend/schemas/blockContent.js)

```javascript
export default {
  title: 'Block Content',
  name: 'blockContent',
  type: 'array',
  of: [
    {
      title: 'Block',
      type: 'block',
      styles: [
        { title: 'Normal', value: 'normal' },
        { title: 'H1', value: 'h1' },
        { title: 'H2', value: 'h2' },
        { title: 'H3', value: 'h3' },
        { title: 'H4', value: 'h4' },
        { title: 'Quote', value: 'blockQuote' },
      ],
      lists: [{ title: 'Bullet', value: 'bullet' }],
      marks: {
        decorators: [
          { title: 'Strong', value: 'strong' },
          { title: 'Emphasis', value: 'em' },
        ],
        annotations: [
          {
            title: 'URL',
            name: 'link',
            type: 'object',
            fields: [
              {
                title: 'URL',
                name: 'href',
                type: 'url',
              },
            ],
          },
        ],
      },
    },
    {
      type: 'image',
      options: { hotspot: true },
    },
  ],
};
```

### Category

Each product will have a category and also a parent category. This means that we can have a parent category like 'Furniture' but be able to drill in further with a category under 'Furniture' like 'Chair'.

[`backend/schemas/category.js`](https://github.com/tzmanics/angular-sanity/blob/main/backend/schemas/category.js)

```javascript
export default {
  name: 'category',
  title: 'Category',
  type: 'document',
  fields: [
    {
      name: 'title',
      title: 'Title',
      type: 'string',
    },
    {
      name: 'slug',
      title: 'Slug',
      type: 'slug',
      options: {
        source: 'title',
        maxLength: 96,
      },
    },
    {
      name: 'description',
      title: 'Description',
      type: 'text',
    },
    {
      name: 'parents',
      title: 'Parents',
      type: 'array',
      of: [
        {
          type: 'reference',
          to: [{ type: 'category' }],
        },
      ],
    },
  ],
};
```

### Product Variants

The product variant will allow us to add more detailed information to each product.

[`backend/schemas/productVariant.js`](https://github.com/tzmanics/angular-sanity/blob/main/backend/schemas/productVariant.js)

```javascript
export default {
  name: 'productVariant',
  title: 'Product Variant',
  type: 'object',
  fields: [
    {
      name: 'title',
      title: 'Title',
      type: 'string',
    },
    {
      name: 'price',
      title: 'Price',
      type: 'number',
    },
    {
      name: 'sku',
      title: 'SKU',
      type: 'string',
    },
    {
      name: 'images',
      title: 'Images',
      type: 'array',
      of: [{ type: 'image' }],
    },
  ],
};
```

### Product

This is the main schema for the products we're adding and it will reference other schemas we've made like category and product variant. Knowing that we can have this type of schema inception adds for a lot of possibilities for customizing how we enter data.

[`backend/schemas/product.js`](https://github.com/tzmanics/angular-sanity/blob/main/backend/schemas/product.js)

```javascript
export default {
  name: 'product',
  title: 'Product',
  type: 'document',
  fields: [
    {
      name: 'title',
      title: 'Title',
      type: 'string',
    },
    {
      name: 'slug',
      title: 'Slug',
      type: 'slug',
      options: {
        source: 'title',
        maxLength: 96,
      },
    },
    {
      name: 'defaultProductVariant',
      title: 'Default variant',
      type: 'productVariant',
    },
    {
      name: 'tags',
      title: 'Tags',
      type: 'array',
      of: [{ type: 'string' }],
    },
    {
      name: 'blurb',
      title: 'Blurb',
      type: 'string',
    },
    {
      name: 'categories',
      title: 'Categories',
      type: 'array',
      of: [
        {
          type: 'reference',
          to: { type: 'category' },
        },
      ],
    },
    {
      name: 'body',
      title: 'Body',
      type: 'array',
      of: [{ type: 'block' }],
    },
  ],
  preview: {
    select: {
      title: 'title',
    },
  },
};
```

One other thing to note about this schema is the `preview` section at the bottom. This is what we will see in the Sanity.io UI when it showcases the list of products. Right now, we have the title but we can add an image, price, or any other information from \`product\` that would be handy to see.

### Schema

This is where it all comes together. We'll want to import all the new schemas we created and add them to the schema types.

[`/backend/schemas/schema.js`](https://github.com/tzmanics/angular-sanity/blob/main/backend/schemas/schema.js)

```diff-javascript
  // First, we must import the schema creator
  import createSchema from 'part:@sanity/base/schema-creator'

  // Then import schema types from any plugins that might expose them
  import schemaTypes from 'all:part:@sanity/base/schema-type'

+ import blockContent from "./blockContent";
+ import category from "./category";
+ import product from "./product";
+ import productVariant from "./productVariant";

  // Then we give our schema to the builder and provide the result to Sanity
  export default createSchema({
    // We name our schema
    name: 'default',
    // Then proceed to concatenate our document type
    // to the ones provided by any plugins that are installed
+   types: schemaTypes.concat([blockContent, category, product, productVariant]),
  })
```

We have all the schemas we need now! If we run `sanity start` now we can see the UI that has been created and even enter some products of our own.

### Importing Existing Data (optional)

We can add products now through the UI but we can also import data. I have data that I have entered, the only catch is the images are a part of the dataset in Sanity, so, unfortunately, they don't import well. Please, feel free to add images to replace these. Image troubles aside, here is the command we can use to bring in exported Sanity.io data that matches the schemas.

In the terminal, we can run the command `sanity dataset import`, pass in the URL to where the exported data lives and the name of the dataset we want to add it to. Because of the image importing situation, we'll need to add `--allow-assets-in-different-dataset` to the end of the command so it ignores that we're adding images from another dataset.

`sanity dataset import https://4cj83dm6.api.sanity.io/v1/data/export/production production --allow-assets-in-different-dataset`

We can run `sanity start` again to see the data added.

## Adding Sanity.io Environment Variables

To connect to the CMS, we'll need to use some project credentials: the project id, dataset, and a token. We can grab this information from the project's dashboard. First, the project id is at the top of the page and we know we are using the 'production' data set. To get the Sanity.io token we'll go to Settings/API/Tokens and click the 'Add New Token' button.

![Sanity.io project dashboard](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617853293/sanity-project-dashboard_qq6m5y.jpg 'Sanity.io project dashboard')

Then, we'll name the token 'functions' and give it rights to write (which includes read, write, and delete data). When we click the 'Add New Token' button, we'll get a token to copy.

![creating a new token](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617853393/create-token_yu6rxo.jpg 'Creating a new token')

![the new token](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617853436/new-token_eefwnw.jpg 'Sanity.io token')

Next, we'll head back over to the project's Netlify dashboard to enter these values. Go to Site settings/Build & deploy/Environment and click the `Edit variables` button. We will add the following variables:

- `SANITY_TOKEN` = (the token we just created and copied)
- `SANITY_DATASET` = production
- `SANITY_PROJECT_ID` = <your project id (e.g. kgu5d2ud)>

![Netlify environment variables entry](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617853535/env-vars_nmfhhv.jpg 'Environment variables')

## Finally, the Function!

We have everything in place to actually make a serverless function to grab the CMS data. Woohoo! Let's get coding.

### Function Setup & Test

Let's start with a test function and see how we can test it locally. We will create a folder in the project's root directory, then make a new file in there named `getProducts.js`.

`mkdir functions && touch getProducts.js`

[`functions/getProducts.js`](https://github.com/tzmanics/angular-sanity/blob/main/functions/getProducts.js)

```javascript
exports.handler = async () => {
  return {
    statusCode: 200.
    body: 'ok',
  };
};
```

### Testing Locally

Through the Netlify CLI we can start up a local development environment by using the `netlify dev` command. Before we run that, let's set local build settings for `command` and `publish` for the dev environment in the `netlify.toml` configuration file.

[`netlify.toml`](https://github.com/tzmanics/angular-sanity/blob/main/netlify.toml)

```toml
[build]
  command = "npm run prerender"
  functions = "./functions"
  publish = "dist/angular-sanity/browser"
[dev]
  command = "npm run start"
  functions = "./functions"
  publish = "src"
```

Now when we run `netlify dev`, we can head to [`http://localhost:8888/.netlify/functions/getProduct`](http://localhost:8888/.netlify/functions/getProduct) to see the output from the test function: 'ok'. Riveting, I know.

![test function output](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617853550/test-output_w13yml.jpg 'Test function output')

> 📓 Here's [a blog post on getting started with Netlify Functions](https://hubs.ly/H0H2h--0) if you want to learn more about what's going on here.

### Fetching Sanity.io Data

We need a few libraries from Sanity.io to make sure the data is configured correctly.

`npm install @sanity/client @sanity/image-url @sanity/block-content-to-html`

Now we can add all the logic we need to the Netlify Function.

[`functions/getProducts.js`](https://github.com/tzmanics/angular-sanity/blob/main/functions/getProducts.js)

```javascript
const sanityClient = require('@sanity/client');
const imageUrlBuilder = require('@sanity/image-url');
const blocksToHtml = require('@sanity/block-content-to-html');

// passing the env vars to Sanity.io
const sanity = sanityClient({
  projectId: process.env.SANITY_PROJECT_ID,
  dataset: process.env.SANITY_DATASET,
  useCdn: true,
});

exports.handler = async () => {
  // this query asks for all products in order of title ascending
  const query = '*[_type=="product"] | order(title asc)';
  const products = await sanity.fetch(query).then((results) => {
    // then it iterates over each product
    const allProducts = results.map((product) => {
      // & assigns its properties to output
      const output = {
        id: product.slug.current,
        name: product.title,
        url: `${process.env.URL}/.netlify/functions/getProducts`,
        price: product.defaultProductVariant.price,
        description: product.blurb,
        // this is where we use the Sanity.io library to make the text HTML
        body: blocksToHtml({ blocks: product.body }),
      };

      // we want to make sure an image exists before we assign it
      const image =
        product.defaultProductVariant.images &&
        product.defaultProductVariant.images.length > 0
          ? product.defaultProductVariant.images[0].asset._ref
          : null;

      if (image) {
        // this is where we use the library to make a URL from the image records
        output.image = imageUrlBuilder(sanity).image(image).url();
      }
      return output;
    });
    // this log lets us see that we're getting the projects
    // we can delete this once we know it works
    console.log(allProducts);

    // now it will return all of the products and the properties requested
    return allProducts;
  });

  return {
    statusCode: 200,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(products),
  };
};
```

### Live Logs & Results

Once we have our function live, where do we find the logs? Netlify has a dedicated section for the functions to make them easy to keep track of and manage. From the project dashboard, there is a 'Functions' tab where we can see all the functions listed that the project uses.

![screenshot of functions dashboard](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617853727/function-dashboard_nqg7g4.jpg 'Functions Dashboard')

To deploy the updated function we can git add, commit, and push the changes. The new build will be triggered and if we go to [`https://angular-sanity.netlify.app/.netlify/functions/getProducts`](https://angular-sanity.netlify.app/.netlify/functions/getProducts) we can see the output on the page and in our logs.

![screenshot of function logs](https://res.cloudinary.com/dzkoxrsdj/image/upload/v1617853744/function-logs_ydsgbr.jpg 'Function Logs')

## Functional Function Finita

We have a template site set up, a customized instance of Sanity.io, and a function that's grabbing our CMS data! We are so skilled! In part two of this tutorial, we'll use add an Angular service and component to integrate this data into the site. Then we'll set up a Build Hook so that we re-deploy the site with the newest data as soon as it's entered. I hope to see you there and until then, happy coding 👩🏻‍💻

## Making the Angular Service

In the first part of the series we [added a Netlify Function to grab the data we needed from Sanity.io](https://github.com/tzmanics/angular-sanity/blob/main/functions/getProducts.js). Now, we will use an [Angular service](https://angular.io/guide/architecture-services) to access that data.

### Adding the Product Model

One thing that we'll need before anything else is an interface of the product data we're receiving from Sanity.io. An [interface](https://www.typescriptlang.org/docs/handbook/interfaces.html) is a TypeScript syntactical contract defining types so they can be checked. The `Product` interface is very complicated...everything is type `string` 🙃. Rest assured you can assign other types in interfaces.

[`src/app/models/Product.ts`](https://github.com/tzmanics/angular-sanity/blob/main/src/app/models/Product.ts)

```typescript
export interface Product {
  id: string;
  name: string;
  price: string;
  url: string;
  image: string;
  description: string;
}
```

### HTTP For You and Me

Another thing we'll need for the service is the [HTTP Client](https://angular.io/guide/http). This is an injectable class with methods to perform HTTP requests. We'll import that in the app's main module file.

[`src/app/app.module.ts`](https://github.com/tzmanics/angular-sanity/blob/main/src/app/app.module.ts)

```diff-typescript
import { BrowserModule } from '@angular/platform-browser';
+ import { HttpClientModule } from '@angular/common/http';
import { NgModule } from '@angular/core';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { NavigationComponent } from './navigation/navigation.component';

@NgModule({
  declarations: [AppComponent, NavigationComponent],
  imports: [
    BrowserModule.withServerTransition({ appId: 'serverApp' }),
    AppRoutingModule,
+    HttpClientModule,
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

### On To the Actual Service

To make the service we can use the handy [`generate` command from the Angular CLI](https://angular.io/cli/generate) to create the files we need.

`ng generate service services/Products`

> 💡 We're using the full command here but we could also use the shortened version `ng g s services/Product`.

Once we have the generated files (`product.service.ts` & `product.service.spec.ts`), we can import the HTTP client and the Product interface. We'll also import `Observable` from the [RxJS](https://rxjs.dev/guide/overview) library since the products will be coming through as an observable sequence.

Inside the `ProductService` class we'll use [dependency injection](https://angular.io/guide/dependency-injection) to use the HTTP client in the service. Finally, we'll make a function, `getProduct` that will be returning `Observable<Product[]>` from its HTTP GET call to where the Netlify Function lives on the live site: `/.netlify/functions/getProducts`. The last thing we'll add to the function is an object that passes the GET call a `headers` object that sets the Content-Type to application/json.

Put it all together and what we get:

[`src/app/services/products.service.ts`](https://github.com/tzmanics/angular-sanity/blob/main/src/app/services/products.service.ts)

```diff-typescript
import { Injectable } from '@angular/core';
+ import { HttpClient } from '@angular/common/http';
+ import { Observable } from 'rxjs';

+ import { Product } from '../models/Product';

@Injectable({
  providedIn: 'root',
})
export class ProductsService {
+ constructor(private http: HttpClient) {}

+ getProducts(): Observable<Product[]> {
+   return this.http.get<Product[]>('/.netlify/functions/getProducts', {
+     headers: {
+       'Content-Type': 'application/json',
+     },
+   });
+ }
}
```

## Adding & Updating the Components

Now it's time to really see the work we've done, literally. We'll make a new component to hold the product information. Then we'll update the existing `Product` component with the newly created component.

### Product List Component

To make a component that lists the product, we'll use the Angular CLI generate command again. This time we'll use it to make a component as so:

`ng g component components/ProductList --module products.module`

We want to pass it the `products.module` module (using `--module`) so that it becomes a child component of the main products component and they can share data.

#### The TypeScript Part

Inside the `ProductList` component's TypeScript file we will:

- import the [`Input` directive](https://angular.io/guide/inputs-outputs) in order to share data from parent to child component
- import the `Product` interface we made in the first step
- inside the `ProductListComponent` declare and instantiate the product data we're sharing from the parent Products component: `@Input() product: Product`.

All of that looks like this:

[`src/app/components/product-list/product-list.component.ts`](https://github.com/tzmanics/angular-sanity/blob/main/src/app/components/product-list/product-list.component.ts)

```diff-typescript
+import { Component, OnInit, Input } from '@angular/core';
+import { Product } from 'src/app/models/Product';

@Component({
  selector: 'app-product-list',
  templateUrl: './product-list.component.html',
  styleUrls: ['./product-list.component.css'],
})
+export class ProductListComponent implements OnInit {
+  @Input() product: Product = {
+    name: 'test',
+    id: '1',
+    price: '1.23',
+    url: 'https://netlify.com',
+    image: 'https://bit.ly/20o7MGL',
+    description: 'test',
+  };

  constructor() {}

  ngOnInit(): void {}
}
```

> 💡 We instantiate the `product` data with default data because TypeScript is set to strict mode and will throw an error if it doesn't have matching types for the Interface, Product. I will show a work around for this further into the post in case this is too cumbersome.

#### The HTML Template Part

Now we get to the actual showing of stuff! To do this we're going to use the `product` data we're passing in and use dot notation to grab the parts of the product data we want to use.

Like so:

[`src/app/components/product-list/product-list.component.html`](https://github.com/tzmanics/angular-sanity/blob/main/src/app/components/product-list/product-list.component.html)

```diff-html
-<p>product-list works!</p>
+<div class="product-item">
+  <img [src]="product.image" />
+  <div class="product-item-details">
+    <h3 class="product-name">{{ product.name }}</h3>
+  </div>
+</div>
```

#### The CSS Styling Part

Finally, the super complicated CSS goes into the component's CSS file. I'm kidding, we're just going to center-align the text.

[`src/app/components/product-list/product-list.component.css`](https://github.com/tzmanics/angular-sanity/blob/main/src/app/components/product-list/product-list.component.css)

```diff-css
+.product-item-details {
+  text-align: center;
+}
```

Phew, we've got the component that shows all the products! BUT it's still not being rendered because we haven't passed this component to the Product template. Guess what we're going to do next!

### Updating the Product Component

From the [template project](https://github.com/tzmanics/angular-sanity) we have a Products component to list all of the products. This is where we'll make the changes to insert all the components for each product.

#### The Module Part

We've actually already changed one of its files without even opening it! When we used the generate command to make the Product List component we referenced this Products' module. Doing so imported the Product List component in the Products' module TypeScript file.

[`src/app/products/products.module.ts`](https://github.com/tzmanics/angular-sanity/blob/main/src/app/products/products.module.ts)

```diff-typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

import { ProductsRoutingModule } from './products-routing.module';
import { ProductsComponent } from './products.component';
+import { ProductListComponent } from '../components/product-list/product-list.component';

@NgModule({
+  declarations: [ProductsComponent, ProductListComponent],
  imports: [CommonModule, ProductsRoutingModule],
})
export class ProductsModule {}
```

#### The TypeScript Part

The Product component is the parent component that will be sending the product data to the Product List component. Like with the Product List component, we'll need to import `Observable` and the `Product` interface. We also need to import `ProductsService`, the service we created earlier that calls the `getProducts` Netlify Function.

Inside of the class is where we do all the work:

- assign `products` to an observable array of the interface we made, Product
- inside the constructor we use data injection to use the service, `ProductsService` and assign it to `productsService`
- when the lifecycle [`ngOnInit`](https://angular.io/api/core/OnInit) is called (after Angular has initialized all data-bound properties of a directive), we assign `products` to what is returned from `getProducts` inside the service `productsService`

[`src/app/products/products.component.ts`](https://github.com/tzmanics/angular-sanity/blob/main/src/app/products/products.component.ts)

```diff-typescript
import { Component, OnInit } from '@angular/core';
+import { Observable } from 'rxjs';

+import { ProductsService } from 'src/app/services/products.service';
+import { Product } from 'src/app/models/Product';

@Component({
  selector: 'app-products',
  templateUrl: './products.component.html',
  styleUrls: ['./products.component.css'],
})
export class ProductsComponent implements OnInit {
+  products!: Observable<Product[]>;

+  constructor(private productsService: ProductsService) {}

  ngOnInit(): void {
+    this.products = this.productsService.getProducts();
  }
}
```

> 💡 When `products` is declared at the beginning of the `ProductsComponent` class I use the `!`, [the TypeScript non-null assertion operator](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html#non-null-assertion-operator). This is the workaround I mentioned earlier that can be used instead of initializing with default data as we did in the Product List component. I don't recommend this but thought it was a good alternative to show instead of turning TypeScript's strict mode off.

Okay, we have the data! Let's get going.

#### The HTML Template Part

We're going to change a few things inside of the Products HTML template:

- add a section for the Product List component
- apply an [`*ngIf`](https://angular.io/api/common/NgIf) conditional directive to say if we have the products data use an [async pipe](https://angular.io/api/common/AsyncPipe) assigning the data to `products`, else displaying a loading block [Angular template](https://angular.io/guide/structural-directives)
- then we pass in the Product List component (`app-product-list`)
- we also pass a few parameters to the Product list component including an [`*ngFor`](https://angular.io/api/common/NgForOf) to loop through the products and assign each one to `product`
- inside the component we'll use [Angular's text interpolation](https://angular.io/guide/interpolation) to display the product's name (`{{ product.name }}`)
- finally, we take the existing 'loading' text and place it inside the conditional `ng-template` named `#loadingBlock`

That was a lot of words but let's take a look below to see what the code actually looks like.

[`src/app/products/products.component.html`](https://github.com/tzmanics/angular-sanity/blob/main/src/app/products/products.component.html)

```diff-html
<div class="products">
+  <main
+    class="products-list"
+    *ngIf="products | async as products; else loadingBlock"
+  >
+    <app-product-list
+      class="product-item"
+      *ngFor="let product of products"
+      [product]="product"
+    >
+      {{ product.name }}
+    </app-product-list>
+  </main>
+  <ng-template #loadingBlock>
+    <p>Loading products...</p>
+  </ng-template>
</div>
```

#### The CSS Styling Part

Let's make sure the products look good on the page. We'll use `inline-flex` for the display, use `wrap` for the `flex-wrap`, and set the `justify-content` to `space-evenly`. We're just going to focus on how this looks on the desktop for now. With that in mind, we'll set `margin` to 15px, `min-width` to 350px, and the width of each product `div` to 20%.

[`src/app/products/products.component.css`](https://github.com/tzmanics/angular-sanity/blob/main/src/app/products/products.component.css)

```diff-css
.products {
  margin: 0 auto;
  width: 80%;
}

.products p {
  font-family: 'Poiret One', sans-serif;
  font-size: 40px;
  margin: 50px;
  text-align: center;
}

+.product-item {
+  display: inline-flex;
+  flex-wrap: wrap;
+  justify-content: space-evenly;
+  margin: 15px;
+  min-width: 350px;
+  width: 20%;
+}
```

### Eye on the Product

Get excited, we can finally see how this looks! To do so locally, we can run `ntl dev` in the root directory of the project. Then we'll head to [`localhost:8888/products`](localhost:8888/products).

![final project locally](/img/blog/project-local.jpg 'The final project locally')

Now if we've either already hooked this project up to Netlify using the CLI `ntl init` command, or followed the steps in [part 1 of this series](https://hubs.ly/H0Hks4C0), we have CI/CD set up. That means we can git commit the code changes we made and it will trigger a new build of the project. Then we can head to the project dashboard or run the command `ntl open` to have Netlify take us right to the dashboard. Once, the new build is published we should see the same products we saw locally.

## Hook it Up

The data from Sanity.io will be added to the site whenever it is built. To make sure we grab data right when it's entered into the CMS, we can set up some hooks.

### Adding a Netlify Build Hook

Netlify provides an easy interface for setting up [Build Hooks](https://hubs.ly/H0HlCyf0) through the project's dashboard. We'll head to Site settings > Build & deploy > Continuous deployment > Build hooks and click the 'Add build hook' button. Today we'll just set the name to 'angular-sanity', for identification, and use the 'main' branch.

![add hook](/img/blog/add-hook.jpg 'Creating a hook')

Once we have the build hook information saved, we'll get a unique URL to copy for the next step. To trigger this hook, we need to send a POST request to that URL. Thankfully, Sanity.io has a quick process to set up that POST request.

![hook link](/img/blog/hook-link.jpg 'Unique link for a build hook')

### Creating a Sanity Hook

In the terminal, we need to make sure we're in the `backend` directory where our Sanity.io instance lives. We can then use the Sanity.io CLI to create a hook with just one command.

`sanity hook create`

Through the prompts, we'll name the hook `netlify`, set the dataset to `production`, and paste the link we just received from Netlify when we made the Build Hook.

![sanity hook](/img/blog/sanity-hook.jpg 'Creating a Sanity.io hook')

### [Hook](https://www.youtube.com/watch?v=pdz5kCaCRFM&ab_channel=BluesTravelerVEVO) Test

Does this work? Well, let's add a new product and see about that. We can head back to our deployed Sanity.io instance. The address for this can be found at the top of the project dashboard beside 'Studio'. Check out [the section on Sanity deploying in the first part of this series](https://hubs.ly/H0HhzY30) to see where this link lives.

![adding new data](/img/blog/add-data.jpg 'Adding new data')

After hitting 'Publish' on the new data, we can look at the Netlify Production deploy logs (on the project's main dashboard) and see that a build has been triggered by the Build Hook.

![hook triggered](/img/blog/hook-triggered.jpg 'Hook triggered deploy')

Once that deploy is published we just head back to our site and, voila, a new product!

![new product added](/img/blog/product-added.jpg 'New product added')

## C'est Fini!

We've accomplished what we've come here to do. We now have a pre-rendered Angular site, deployed to a CDN, that is grabbing information from a headless CMS (Sanity.io) using a Netlify Function, and updating with a build hook for dynamic data. Congrats! I hope this helps you on your Jamstack "Jamgular" future. Happy coding 👩🏻‍💻!

## Resources for the Road

- [Headless CMS explained in 2 minutes](https://www.sanity.io/blog/headless-cms-explained)
- [Sanity.io Documentation](https://www.sanity.io/docs)
- [Netlify Functions](https://hubs.ly/H0H0wmQ0)
- [Netlify Build Hooks](https://hubs.ly/H0H0wKK0)
