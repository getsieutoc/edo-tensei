# Edo Tensei

Built on top of `cms-wordpress` [example](https://github.com/vercel/next.js/tree/canary/examples/cms-wordpress)

## Key features:

- `robots.ts`: This automatically gets the robots.txt of the API route and serves it on the `/robots.txt` route.
- `sitemap.ts`: This automatically gets all paths from the API and generates a sitemap to serve on the `/sitemap.xml` route.
- `middleware.ts`: This contains a middleware function that checks the users path for stored redirects, and redirects the user if a match is found.
- `[[...slug]]`: This is the catch-all route that is used to render all pages. It is important that this route is not removed, as it is used to render all pages. It fetches the ContentType and renders the corresponding
- `not-found.tsx`: This page is used for dynamic 404 handling - adjust the database id to match your decired WordPress page, and make sure the WordPress slug is "not-found", your 404 page will then be editable from your CMS.
- `codegen.ts`: Automatic type generation for your WordPress installation
- `Draft Mode`: Seamless Preview / Draft Preview support, using authentication through WPGraphQL JWT Authentication and Next.js Draft Mode
- `On Demand Cache Revalidation`: Including a bare minimum WordPress theme that implements cache revalidation, WordPress link rewrites and other utils for integrating with Next.js

## How to use

### Clone the project

```
pnpm dlx tiged getsieutoc/edo-tensei your-project
```

## Configuration

### WordPress

1. Set `Site Address (URL)` to your frontend URL, e.g. `https://localhost:3000` in Settings -> General
2. Make sure Permalinks are set to `Post name` in Settings -> Permalinks
3. Set `Sample page` as `Static page` in Settings -> Reading
4. Create a new page called `404 not found` ensuring the slug is `404-not-found`
5. Install and activate following plugins:
   - Add WPGraphQL SEO
   - Classic Editor
   - Redirection
   - WPGraphQL
   - [WPGraphQL JWT Authentication](https://github.com/wp-graphql/wp-graphql-jwt-authentication/releases)
   - Yoast SEO
   - [Advanced Custom Fields PRO](https://www.advancedcustomfields.com/pro/) (optional)
   - WPGraphQL for ACF (optional)
6. Do first-time install of Redirection. Recommended to enable monitor of changes
7. Configure Yoast SEO with:

   - Disable XML Sitemaps under Yoast SEO -> Settings
   - If you did not change the `Site Address (URL)` before installing Yoast, it will ask you to run optimize SEO data after changing permalinks, do so
   - Generate a robots.txt file under Yoast SEO -> Tools -> File Editor
   - Modify robots.txt sitemap reference from `wp-sitemap.xml` to `sitemap.xml`

8. `Enable Public Introspection` under GraphQL -> Settings
9. Add following constants to `wp-config.php`
   ```php
   define('HEADLESS_SECRET', 'INSERT_RANDOM_SECRET_KEY');
   define('HEADLESS_URL', 'INSERT_LOCAL_DEVELOPMENT_URL'); // http://localhost:3000 for local development
   define('GRAPHQL_JWT_AUTH_SECRET_KEY', 'INSERT_RANDOM_SECRET_KEY');
   define('GRAPHQL_JWT_AUTH_CORS_ENABLE', true);
   ```
10. Create a bare minimum custom WordPress theme, consisting of only 2 files:

- [style.css](https://developer.wordpress.org/themes/basics/main-stylesheet-style-css/#basic-structure)
- functions.php (see the bottom of this README)

### Next.js

1. Clone the repository
2. Run `npm install` to install dependencies
3. Create `.env` file in the root directory and add the following variables:

| Name                                 | Value                                                                   | Example                  | Description                                                                                                                                                         |
| ------------------------------------ | ----------------------------------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `NEXT_PUBLIC_BASE_URL`               | Insert base url of frontend                                             | http://localhost:3000    | Used for generating sitemap, redirects etc.                                                                                                                         |
| `NEXT_PUBLIC_WORDPRESS_API_URL`      | Insert base url of your WordPress installation                          | http://wp-domain.com     | Used when requesting wordpress for data                                                                                                                             |
| `NEXT_PUBLIC_WORDPRESS_API_HOSTNAME` | The hostname without protocol for your WordPress installation           | wp-domain.com            | Used for dynamically populating the next.config images remotePatterns                                                                                               |
| `HEADLESS_SECRET`                    | Insert the same random key, that you generated for your `wp-config.php` | INSERT_RANDOM_SECRET_KEY | Used for public exhanges between frontend and backend                                                                                                               |
| `WP_USER`                            | Insert a valid WordPress username                                       | username                 | Username for a system user created specifically for interacting with your WordPress installation                                                                    |
| `WP_APP_PASS`                        | Insert application password                                             | 1234 5678 abcd efgh      | [Generate an application password](https://make.wordpress.org/core/2020/11/05/application-passwords-integration-guide/) for the WordPress user defined in `WP_USER` |

> [!WARNING] > `WP_USER` and `WP_APP_PASS` are critical for making preview and redirection work

4. Adjust the ID in `not-found.tsx` to match the post id of your "404 Not Found" page in WordPress

5. `npm run dev` and build an awesome application with WordPress!

> [!NOTE] > Running `npm run dev` will automatically generate typings from the WordPress installation found on the url provided in your environment variable: `NEXT_PUBLIC_WORDPRESS_API_URL`

## GraphQL and typescript types

We are generating typescript types from the provided schema with Codegen.

### Enabling Auto Completion for graphql queries

If you want to add auto completion for your queries, you can do this by installing the "Apollo GraphQL" extension in VS Code and adding an `apollo.config.js` file, next to the `next.config.js`, and add the following to it:

```javascript
module.exports = {
  client: {
    service: {
      name: "WordPress",
      localSchemaFile: "./src/gql/schema.gql",
    },
  },
};
```

## Advanced Custom Fields PRO (optional, but recommended)

I will recommend building your page content by using the [Flexible Content](https://www.advancedcustomfields.com/resources/flexible-content/) data type in ACF Pro.
This will make you able to create a "Block Builder" editor experience, but still having everything automatically type generated, and recieving the data in a structured way.
The default "Gutenberg" editor returns a lot of HTML, which makes you loose a lot of the advantages of using GraphQL with type generation.

## Redirection setup

The example supports the WordPress "Redirection" plugin. the `WP_USER` and `WP_APP_PASS` environment variables are required, for this to work. By implementing this you can manage redirects for your content, through your WordPress CMS

## Draft / Preview support

The example supports WordPress preview (also draft preview), when enabling `draftMode` in the `api/preview/route.ts` it logs the `WP_USER` in with the `WP_APP_PASS` and requests the GraphQL as an authenticated user. This makes draft and preview available. If a post is in "draft" status, it doesn't have a real slug. In this case we redirect to a "fake" route called `/preview/${id}` and uses the supplied id for fetching data for the post.

## Cache Revalidation

All our GraphQL requests has the cache tag `wordpress` - when we update anything in WordPress, we call our `/api/revalidate` route, and revalidates the `wordpress` tag. In this way we ensure that everything is up to date, but only revalidate the cache when there actually are updates.

## Template handling

We use an "Optional Catch-all Segment" for handling all WordPress content.
When rendering this component we simply ask GraphQL "what type of content is this route?" and fetch the corresponding template.
Each template can then have their own queries for fetching specific content for that template.

## SEO

We are using Yoast SEO for handling SEO in WordPress, and then all routes are requesting the Yoast SEO object, and parsing this to a dynamic `generateMetadata()` function

## Folder structure

The boilerplate is structured as follows:

- `app`: Contains the routes and pages of the application
- `assets`: Contains helpful styles such as the variables
- `components`: Contains the components used in the application
- `gql`: Contains auto-generated types from GraphQL via CodeGen
- `queries`: Contains reusable data fetch requests to GraphQL
- `utils`: Contains helpful functions used across the application
