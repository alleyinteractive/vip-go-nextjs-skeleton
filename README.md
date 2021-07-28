# VIP Go Next.js boilerplate for decoupled / headless WordPress applications

This is WordPress VIP's [Next.js][nextjs] boilerplate for decoupled WordPress. In order to operate correctly, VIP's [decoupled plugin bundle][bundle] must be installed and activated on the WordPress backend. The notes below describe its behavior when run on [WordPress VIP's platform][wpvip].

## Features

+ [Next.js][nextjs] 11
+ Fetch data with [Apollo][apollo] and [WPGraphQL][wpgraphql]
+ Seamless previewing
+ Easily map Gutenberg blocks to React components and incorporate your design system
+ Automatic [code generation][code-generation] from GraphQL queries
+ Optional TypeScript support
+ Sitemap and RSS feeds

## Getting started

### Install dependencies

```sh
npm install
```

### Environment variables

Update the environment variables defined in the `.env` file:

+ `NEXT_PUBLIC_GRAPHQL_ENDPOINT`: Your WordPress GraphQL endpoint. You can find it in the WordPress Admin Dashboard > Settings > VIP Decoupled.
+ `WORDPRESS_ENDPOINT`: Your WordPress root URL, needed for previewing, syndication feeds, and sitemaps.

If you have additional environment variables, you can add them here.

### Development server

Start a development server, with hot-reloading, at [http://localhost:3000][local].

```sh
npm run dev
```

### Production build

```sh
npm run build
npm start
```

These are the exact same commands that will be executed when your application runs on WordPress VIP. Testing your production build locally is a good step to take when troubleshooting issues in your production site.

## Previewing

Previewing unpublished posts or updates to published posts works out of the box. Simply click the “Preview” button in WordPress and you’ll be redirected to a one-time-use preview link on the Next.js site. You can share your preview link with others; as long as they are logged in to WordPress in the same browser and have permissions to access that content, they will be able to preview it as well.

## Gutenberg / block support

When you query for content (posts, pages, and custom post types), you'll receive the post content as blocks. If the content was written with WordPress's [block editor][gutenberg] (Gutenberg), these blocks will correspond directly with the blocks you see in the editor.

Receiving the content as blocks allows you to easily map each block type to a React component, and the [block attributes][block-attributes] to component props. This boilerplate provides a few mappings for basic components like headings, paragraphs, and lists. Here is a simple example of this mapping:

```js
function PostContent ( props ) {
	return (
		<>
			{
				props.blocks.map( block => {
					switch ( block.type ) {
						case 'core/heading':
							return (
								<Heading
									innerHTML={block.innerHTML}
								/>
							);

						case 'core/paragraph':
							return (
								<Paragraph
									innerHTML={block.innerHTML}
								/>
							);

						return null;
					}
				} )
			}
		</>
	);
}
```

See the [`PostContent`][post-content] component for a full working example. You will probably need to write additional components and modify the `switch` statement in `PostContent` in order to support all of the block types and custom blocks that you use in your WordPress instance.

If you used WordPress's [classic editor][classic-editor], you will receive a single block representing the HTML content of the entire post. A `ClassicEditorBlock` component is provided to render these blocks.

### Unsupported blocks

When running the development server, in order to help you identify blocks that have not yet been mapped to a React component, this boilerplate will display an "Unsupported block" notice. This notice is suppressed in production and the block is simply ignored.

## Data fetching

Next.js is optimized to create performant pages that are statically generated at build time ([`getStaticProps`][get-static-props]) or server-side-rendered at request time ([`getServerSideProps`][get-server-side-props]). This results in HTML that is cacheable at the edge and immediately crawlable by search engines—both critically important factors in the performance and success of your site.

This boilerplate uses [Apollo][apollo] to query for data using GraphQL, and it is configured and ready to use. Note that Apollo hooks (e.g., `useQuery`) are not compatible with `getStaticProps` or `getServerSideProps`.

### Client-side data fetching

Many Apollo implementations, including Next.js’s official examples, implement a complex, isomorphic approach that bootstraps and hydrates the data from the server-side render into an in-memory cache, where it can be used for client-side requests. We have intentionally avoided this approach because it introduces a large performance penalty and increases the risk that performance degrades even more over time.

Before adding client-side data fetching, examine your typical user flows in detail and consider whether it truly benefits your application and its users. Skipping this complicated step simplifies your configuration, decreases page weight, and usually increases overall performance. If you absolutely need to perform client-side data fetching, an `ApolloProvider` is exported and ready to use in [`graphql-provider`][apollo-provider]. Note that data from the server-side render will not be hydrated into the store.

### Code generation

Our boilerplate has a code generation step that examines the GraphQL queries in `./graphql/queries/`, introspects your GraphQL schema, and generates TypeScript code that can be used to load data via Apollo. See [`LatestContent`][latest-content] for an example of using generated code with `getStaticProps` and [`Post`][post] for an example with `getServerSideProps`.

Having declared types across the entire scope of data fetching chain—queries, responses, and React component props—is incredibly powerful and provides confidence as you build your site. Code generation runs automatically on all GraphQL queries in `./graphql/queries/`. In development, if you make changes or additions to your queries, you will need to restart the development server to see those changes reflected.

## Caching

Responses from Next.js are cached by [VIP's page cache][page-cache] for five minutes by default.

As `POST` requests, GraphQL queries are not cached. However, when using static or server-side data loading—which is strongly recommended—these queries are effectively cached by the page cache.

## Custom server

WordPress VIP's platform requires a [healthcheck endpoint][healthcheck] to assist in monitoring. Providing this endpoint correctly requires a [custom server][custom-server]. We have also found that it is easy to outgrow Next.js's builtin server, so having a custom server (based on [Express 4][express]) available out-of-the-box can be useful.

## Sitemap and syndication (RSS) feeds

`/sitemap.xml` is proxied to your WordPress backend, where it is fulfilled by the default sitemap provided by WordPress (or whatever is served there, if it is overridden by a plugin). Any path that ends in `/feed` is *redirected* to your WordPress backend. This matches the rewrite rule that exists in WordPress.

## TypeScript

This boilerplate is written in [TypeScript][typescript]. Next.js has [built-in support for TypeScript][nextjs-ts] and processes it automatically in both development and production. If you're already proficient in TypeScript, see [`tsConfig.json`][ts-config] for details.

You don’t need to use TypeScript to use this boilerplate: our `tsConfig.json` is lenient and allows you to write code in either TypeScript or JavaScript.

[apollo]: https://www.apollographql.com
[apollo-provider]: https://github.com/Automattic/vip-go-nextjs-skeleton/blob/main/graphql/apollo-provider.tsx
[block-attributes]: https://developer.wordpress.org/block-editor/reference-guides/block-api/block-attributes/
[bundle]: https://github.com/Automattic/vip-decoupled-bundle
[classic-editor]: https://wordpress.com/support/classic-editor-guide/
[code-generation]: https://www.graphql-code-generator.com
[custom-server]: https://nextjs.org/docs/advanced-features/custom-server
[express]: https://expressjs.com
[get-server-side-props]: https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering
[get-static-props]: https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation
[gutenberg]: https://developer.wordpress.org/block-editor/
[healthcheck]: https://docs.wpvip.com/technical-references/vip-platform/node-js/#h-requirement-1-exposing-a-health-route
[latest-content]: https://github.com/Automattic/vip-go-nextjs-skeleton/blob/main/pages/latest/%5Bcontent_type%5D.tsx
[local]: http://localhost:3000
[nextjs-ts]: https://nextjs.org/docs/basic-features/typescript
[nextjs]: https://nextjs.org
[page-cache]: https://docs.wpvip.com/technical-references/caching/page-cache/
[post]: https://github.com/Automattic/vip-go-nextjs-skeleton/blob/main/pages/%5B...slug%5D.tsx
[post-content]: https://github.com/Automattic/vip-go-nextjs-skeleton/blob/main/components/PostContent/PostContent.tsx
[ts-config]: https://github.com/Automattic/vip-go-nextjs-skeleton/blob/main/tsconfig.json
[typescript]: https://www.typescriptlang.org
[wpgraphql]: https://www.wpgraphql.com
[wpvip]: https://wpvip.com
