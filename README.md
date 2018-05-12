# gatsby-remark-source-name

Add a custom `sourceName` field to Remark nodes to filter them easily.

## Install

`yarn add gatsby-remark-source-name`

## How to use

First add the plugin to your `gatsby-config.js`.

```javascript
plugins: [`gatsby-remark-source-name`];
```

Next you define a name for the group of markdown files in the filesystem source plugin:

```javascript
{
  resolve: `gatsby-source-filesystem`,
  options: {
    path: `${__dirname}/src/_posts`,
    name: 'blog', // -> name of the group
  }
}
```

You can then query the source name in GraphQL:

```js
// example of gatsby-node.js

exports.createPages = ({ boundActionCreators, graphql }) => {
  const { createPage } = boundActionCreators;

   // create pages
  const PostTemplate = path.resolve('src/templates/post.js');


  const query = graphql(`
    query {
      allMarkdownRemark() {
        edges {
          node {
            excerpt(pruneLength: 250)
            html
            fields {
              sourceName
            }
          }
        }
      }
    }
  `);

  return query.then(result => {
    if (result.errors) {
      return Promise.reject(result.errors);
    }

    // filter by source instance name
    const posts = result.data.allMarkdownRemark.edges.filter(
      single => single.node.fields.sourceName === 'blog' // we use the source name to filter nodes
    );

    posts.forEach(({ node }) => {
      createPage({
        path: `/blog/${node.frontmatter.slug}`,
        component: PostTemplate,
        context: {
          slug: node.frontmatter.slug,
        },
      });
    });
  });
});
```
