<h1 align="center"><a href="https://egghead.io/courses/build-content-rich-progressive-web-apps-with-gatsby-and-contentful">Build Content Rich Progressive Web Apps with Gatsby and Contentful</a></h1>

<p align="center"><img src="https://d2eip9sf3oo6c2.cloudfront.net/series/square_covers/000/000/308/full/EGH_GatsbyContentful_Final.png" width="200"></p>

These notes are intended to be used and studied in tandem with [Khaled Garbaya](https://twitter.com/khaled_garbaya)'s [Build Content Rich Progressive Web Apps with Gatsby and Contentful](https://egghead.io/courses/build-content-rich-progressive-web-apps-with-gatsby-and-contentful) course.

Right below is the intended outcomes of the course, these are the skills and knowledge you will learn from the course.

## Outcomes

```
- Data Modeling with Contentful
- Starting a Gatsby project
- Using GraphQL to query data in Gatsby
- Deploying a static site with Netlify
- Set up automatic redeployment
```

## Prerequisites

- [Build a Blog with React and Markdown using Gatsby](https://egghead.io/courses/build-a-blog-with-react-and-markdown-using-gatsby)
- Know React and GraphQL basics

## Contribute

These are community notes that I hope everyone who studies benefits from. If you notice areas that could be improved please feel free to open a PR!

## Table of Contents

- [Model Content in the Contentful Web App](#1-model-content-in-the-contentful-web-app)
- [Model Content programmatically using the contentful-migration tool](#2-model-content-programmatically-using-the-contentful-migration-tool)
- [Add Contentful as a data source for Gatsby](#3-add-contentful-as-a-data-source-for-gatsby)
- [List data entries from Contentful in Gatsby](#4-list-data-entries-from-contentful-in-gatsby)
- [Programmatically create Gatsby pages from Contentful data](#5-programmatically-create-gatsby-pages-from-contentful-data)
- [Render Contentful rich text in Gatsby](#6-render-contentful-rich-text-in-gatsby)
- [Use Graphql backreference to avoid circular dependencies between Content model](#7-use-graphql-backreference-to-avoid-circular-dependencies-between-content-model)
- [Deploy a Gatsby website on Netlify](#8-deploy-a-gatsby-website-on-netlify)
- [Trigger Netlify Builds when content changes in Contentful](#9-trigger-netlify-builds-when-content-changes-in-contentful)

## 1. Model Content in the Contentful Web App

[**Video**](https://egghead.io/lessons/javascript-model-content-in-the-contentful-web-app)

[Contentful](https://www.contentful.com) organizes content into spaces that allows you to group all the related resources for a project together. This includes content entries, media assets, and settings for localizing content into different languages.

- Each space contains 1 or more content types that define the structure of your content.
- Each content type can have up to 50 fields that you define this is called content modeling.

## 2. Model Content programmatically using the contentful-migration tool

[**Video**](https://egghead.io/lessons/javascript-model-content-programmatically-using-the-contentful-migration-tool)

Contentful migration tool you could potentially start anywhere with your content model, then refine it as you learn what you truly need.

You need to have the contentful-cli installed and then login. This will open up a browser window and authenticate you to Contentful.

```bash
npm install -g contentful-cli
```

```bash
contentful login
```

Export the function. Inside of this function, we'll have access to the migration object that will allow us to do any sort of manipulations to our content type. First thing we need to do is create the content type, then we need to define its fields

```js
module.exports = function(migration) {
  // create the content type
  const instructor = migration
    .createContentType("intructor")
    .name("Instructor")
    .description("")
    .displayField("fullName");

  //fields
  instructor
    .createField("fullName")
    .name("Full Name")
    .type("Symbol");

  // fields
  instructor
    .createField("fullName")
    .name("Full Name")
    .type("Symbol");
  instructor
    .createField("slug")
    .name("Slug")
    .type("Symbol");
  instructor
    .createField("bio")
    .name("Bio")
    .type("Symbol");
  instructor
    .createField("website")
    .name("website")
    .type("Symbol");
  instructor
    .createField("twitter")
    .name("Twitter")
    .type("Symbol");
  instructor
    .createField("github")
    .name("Github")
    .type("Symbol");

  instructor
    .createField("avatar")
    .name("Avatar")
    .type("Link")
    .linkType("Asset");

  // appearances
  instructor.changeEditorInterface("slug", "slugEditor", {});
  instructor.changeEditorInterface("website", "urlEditor", {});
  instructor.changeEditorInterface("twitter", "urlEditor", {});
  instructor.changeEditorInterface("github", "urlEditor", {});
};
```

We call the `.changeEditorInterface` on the content type, and we give it the field ID and then the ID of the widget. We can do the same for the website field and other similar fields.

Now we call the migration on the space that we have (we pass it the file that we want to run, basically our migration code.).

```
ontentful space migration --space-id=lkb87t4toc0t instructor.js
```

The space-id, we can get from Contentful. It's basically the URL.

We can follow the same steps for other content types.

```js
// seo.js
module.exports = function(migration) {
  const seo = migration
    .createContentType("seo")
    .name("SEO")
    .description("")
    .displayField("title");
  seo
    .createField("title")
    .name("Title")
    .type("Symbol");
  seo
    .createField("description")
    .name("Description")
    .type("Symbol");
  seo
    .createField("keywords")
    .name("Keywords")
    .type("Symbol");
};
```

```js
// lesson.js
module.exports = function(migration) {
  const lesson = migration
    .createContentType("lesson")
    .name("Lesson")
    .description("")
    .displayField("title")
  lesson
    .createField("title")
    .name("Title")
    .type("Symbol")
  lesson
    .createField("slug")
    .name("Slug")
    .type("Symbol")
  lesson
    .createField("body")
    .name("Body")
    .type("RichText")

  lesson
    .createField("instructor")
    .name("Instructor")
    .type("Link")
    .validations([
      {
        linkContentType: ["instructor"],
      },
    ])
    .linkType("Entry")

  lesson
    .createField("image")
    .name("Image")
    .type("Link")
    .linkType("Asset")

  lesson
    .createField("seo")
    .name("SEO")
    .type("Link")
    .validations([
      {
        linkContentType: ["seo"],
      },
    ])
    .linkType("Entry")
```

This code should live next to your website in the repository, so another developer can bootstrap a separate space. For example, for testing.

## 3. Add Contentful as a data source for Gatsby

[**Video**](https://egghead.io/lessons/gatsby-add-contentful-as-a-data-source-for-gatsby)

Run `npx gatsby new` and give it the name of the website.

```bash
npx gatsby new jamstacktutorials
```

Now let's `cd` to this directory. Let's run `npm run develop`. Now let's add some Contentful dependency to it. Install the `gatsby-source-contentful` plugin:

```bash
npm i gatsby-source-contentful
```

We will need to provide two options -- the `spaceId` and the `accessToken`. To get this, we need to go to Contentful and Settings, API Keys. We will click on the first entry here, copy the `spaceId` and copy the Delivery `accessToken`.

```js
    resolve: `gatsby-source-contentful`,
    options: {
        spaceId: `u2hjug1nowzr`,
        accessToken: `sJJaBCxUdA4BFqtfR_f5y4m9lvmyOHa3siR8iEETKEc`,
    }
```

Now go and add content. To do that, we can go to Content, click on Add entry. Once you have all your content created, you can test if this works by going to the GraphQL server that's provided by Gatsby. And querying for our content.

## 4. List data entries from Contentful in Gatsby

[**Video**](https://egghead.io/lessons/gatsby-list-data-entries-from-contentful-in-gatsby)

In this lesson, you will learn how to get a list of entries from Contentful and render it in a Gatsby website.

First, define the GraphQL query that gets all the lessons. Type `allContentfulLesson` and find the types that you need. For example:

![](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1562190183/transcript-images/egghead-list-data-entries-from-contentful-in-gatsby-url-return.png)

We will paste the query that we already tested into Gatsby:

```js
export const query = graphql`
  {
    allContentfulLesson {
      edges {
        node {
          title
          slug
          image {
            file {
              url
            }
          }
        }
      }
    }
  }
```

Let's `import { graphql } from "gatsby"`. Once the query is done, Gatsby will pass in the `data` in the props. Here, we can extract data. Inside of data, we will have `allContentfulLesson`.

Once you have all the data import it, now you can render it. Example:

```js
import Card from "../components/card";

<div className="flex flex-wrap -mx-3">
  {allContentfulLesson.edges.map(({ node }) => (
    <Card node={{ ...node, slug: `/lessons/${node.slug}` }} key={node.id} />
  ))}
</div>;
```

## 5. Programmatically create Gatsby pages from Contentful data

[**Video**](https://egghead.io/lessons/gatsby-programmatically-create-gatsby-pages-from-contentful-data)

We need to go to the `gatsby-node.js` and write some code here to create the pages. First, we need to export a `createPages` function.

We can then extract the `createPage` from the actions object. To be able to create a page, we need a path to a template, which is basically a React component.

We can use GraphQL to query the Gatsby data for all the Contentful lessons.

```js
// gatsby-node.js
const path = require(`path`);

exports.createPages = ({ graphql, actions }) => {
  const { createPage } = actions;
  const lessonTemplate = path.resolve(`src/templates/lesson.js`);
  const instructorTemplate = path.resolve(`src/templates/instructor.js`);
  return graphql(`
    {
      allContentfulLesson {
        edges {
          node {
            slug
          }
        }
      }
    }
  `).then(result => {
    if (result.errors) {
      throw result.errors;
    }

    result.data.allContentfulLesson.edges.forEach(edge => {
      createPage({
        path: `/lessons/${edge.node.slug}`,
        component: lessonTemplate,
        context: {
          slug: edge.node.slug
        }
      });
    });
  });
};
```

The GraphQL function returns a promise that will contain our `result`, and inside of the `result`, we need to check for errors. If so, we throw result.error.

The `createPage` will accept the path the component, and a context for additional data.

Let's save this and create our lesson template.

```js
// lesson.js template
import React from "react";
import { graphql } from "gatsby";
import Layout from "../components/layout";
import SEO from "../components/seo";

export const query = graphql`
  query lessonQuery($slug: String!) {
    contentfulLesson(slug: { eq: $slug }) {
      title
      body {
        json
      }
      seo {
        title
        description
      }
    }
  }
`;

function Lesson({ data }) {
  return (
    <Layout>
      <SEO
        title={data.contentfulLesson.seo.title}
        description={data.contentfulLesson.seo.description}
      />
      <div className="lesson__details">
        <h2 className="text-4xl">{data.contentfulLesson.title}</h2>
        {documentToReactComponents(data.contentfulLesson.body.json, {
          renderNode: {
            [BLOCKS.HEADING_2]: (node, children) => (
              <h2 className="text-4xl">{children}</h2>
            ),
            [BLOCKS.EMBEDDED_ASSET]: (node, children) => (
              <img src={node.data.target.fields.file["en-US"].url} />
            )
          }
        })}
      </div>
    </Layout>
  );
}

export default Lesson;
```

Let's hit save, and we need to restart our server. Now, if we refresh, you will be able to see the content that was generated.

## 6. Render Contentful rich text in Gatsby

[**Video**](https://egghead.io/lessons/gatsby-render-contentful-rich-text-in-gatsby)

Let's take a look first at how this data is sent to us. If we go to the GraphiQL, and then request the body, inside of the body, we can see JSON. In the result here, we have all our nodes, and you can see the type and all the content.

![](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1562190183/transcript-images/gatsby-render-contentful-rich-text-in-gatsby-output.png)

We need a way to parse this JSON to React components:

```js
npm i @contentful/rich-text-react-renderer @contentful/rich-text-types
```

In the lesson.js here, we need to add the body to the query. We require the json data from the body.

```js
export const query = graphql`
  query lessonQuery($slug: String!) {
    contentfulLesson(slug: { eq: $slug }) {
      title
      body {
        json
      }
      seo {
        title
        description
      }
    }
  }
```

We need to import the `documentToReactComponents` function from the rich text React renderer.

```
import { documentToReactComponents } from "@contentful/rich-text-react-renderer"
```

We take this function, and in the markup here, we give it the data from the body. It will be `data.contentfulLesson.body.json`.

```js
{
  documentToReactComponents(data.contentfulLesson.body.json, {
    renderNode: {
      [BLOCKS.HEADING_2]: (node, children) => (
        <h2 className="text-4xl">{children}</h2>
      ),
      [BLOCKS.EMBEDDED_ASSET]: (node, children) => (
        <img src={node.data.target.fields.file["en-US"].url} />
      )
    }
  });
}
```

If we save now and run our server again, and once we refresh, we can see indeed here we have the content.

And we add here a second argument, an object configuration. For every h2, we will receive this callback, and then we can return how it will show up.

```
 [BLOCKS.HEADING_2]: (node, children) => (
              <h2 className="text-4xl">{children}</h2>
            ),
```

To render an image, let's go to the options here, and then add the BLOCKS.EMBEDDED_ASSET, and return an image with the correct URL. Let's save this.

```
 [BLOCKS.EMBEDDED_ASSET]: (node, children) => (
  <img src={node.data.target.fields.file["en-US"].url} />
),
```

You should now see rich text data from Contentful into Gatsby. If you find an error, remove the cache and public folder, and then run again.

## 7. Use Graphql backreference to avoid circular dependencies between Content model

[**Video**](https://egghead.io/lessons/gatsby-use-graphql-backreference-to-avoid-circular-dependencies-between-content-model)

We have a lesson that references an instructor but the instructor content type does not have any data about the lessons that are assigned to it. Usually, you would create a reference back in the instructor content type to the lesson but this can cause a circular reference that's hard to maintain. Fortunately in the Gatsby data layer, GraphQL, we can query the parent node to get its data from its child. That's called backreference.

If we type lesson, you can see here that we have access to the lesson, even if it's not linked directly to the instructor in Contentful. Here, we can grab stuff like the title, the image, and so on. Let's do the same in our code and render a list of lessons.

![](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1562190182/transcript-images/egghead-use-graphql-backreference-to-avoid-circular-dependencies-between-content-model-output.png)

First thing we need to do is to update this query that we already exported:

```
    bio
    website
    lesson {
      id
      title
      slug
      image {
        file {
          url
        }
      }
    }
  }
}
```

Now that's available for us, let's render it. For that, we need to check first if the lesson is not null. Otherwise, we look through that lesson array and we render it. We will use the same Card component that we use in our index page.

```
    {data.contentfulInstructor.lesson && (
    <div className="border-t my-6">
        <h2 className="text-4xl text-grey-dark my-6">Lessons</h2>
        {data.contentfulInstructor.lesson.map(node => (
        <Card
            node={{ ...node, slug: `/lessons/${node.slug}` }}
            key={node.id}
        />
        ))}
    </div>
    )}
```

## 8. Deploy a Gatsby website on Netlify

[**Video**](https://egghead.io/lessons/gatsby-deploy-a-gatsby-website-on-netlify)

Go to the command line. Here, we will paste this. Hit enter:

```
git remote add origin git@github.com: Khaledgarbaya/jamstacktutorials.git
```

Now that we have the remote added, let's add everything and commit.

Once we have pushed our code to Github, let's go to Netlify. After you log in, this is your main dashboard. You click new site from Git. Here we click GitHub.

![](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1562190183/transcript-images/gatsby-deploy-a-gatsby-website-on-netlify-dashboard.png)

This is our repository. You can see here that it's a Gatsby project. It will run this Gatsby build command from my server. Let's deploy the site.

You can see the logs. This is our website being built.

![](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1562190183/transcript-images/gatsby-deploy-a-gatsby-website-on-netlify-logs.png)

Now, our site is live. We can click on the preview button. You can see here this is our website.

## 9. Trigger Netlify Builds when content changes in Contentful

[**Video**](https://egghead.io/lessons/netlify-trigger-netlify-builds-when-content-changes-in-contentful)

Netlify will rebuild our website whenever we push new code, but we also want to trigger the rebuild whenever we do content changes. We can do that using what's called webhooks. Let's go to deploy settings, and in build hooks, you can click add build hook. Let's call this `Contentful` and hit save.

![](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1562190182/transcript-images/egghead-trigger-netlify-builds-when-content-changes-in-contentful-save-hook.png)

We grab this URL and go to Contentful. In there, we go into settings, webhooks, and we add webhook. This one, we'll call it `Netlify`. We will paste our URL in here, and we select specific events that will trigger this rebuild.

![](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1562190183/transcript-images/egghead-trigger-netlify-builds-when-content-changes-in-contentful-netlify-hook.png)

Let's change something and we hit publish. Now, when we go back to Netlify, and go to deploys, you can see that here, it's triggered by `Contentful`.

![](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1562190184/transcript-images/egghead-trigger-netlify-builds-when-content-changes-in-contentful-rebuilding.png)
