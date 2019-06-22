+++
title="Prefetching Data with Next.js and Apollo"
date=2018-01-08
[extra]
tags="nextjs, apollo, apollo-server, next.js, prefetch, performance, graphql, react, react.js, apollo-client"
+++

> Update: My pull requests implementing the techniques discussed here were merged into the zeit/next.js [with-data-prefetch][11] and [with-apollo][1] examples. You can easily combine the various examples (including the above two) to achieve your desired functionality.

For most modern web apps, network speed and latency is still the biggest constraint for high performance and great UX. Universally rendered Isomorphic applications do a great job to solve that to some extent without sacrificing interactivity by processing the complete DOM of the first request on the server and subsequent requests on the client. But there is a lot more we can do.

<!-- more -->

Modern web apps already have their core application logic on the client by the time the first request completes. From there on anticipating the user action and selectively prefetching the data can lead to great performance and UX improvements.

[Apollo][5] and [Next][6] is the stack of our choice, having the necessary capabilities, to demonstrate our concept.

Adam Soffer's [introduction][4] summarizes [Apollo][5] and [Next][6] with their concerns perfectly:

> Apollo is a GraphQL client that’s concerned exclusively with the data layer of our application; it cares about efficiently fetching our data. Next, on the other hand, is a minimalistic framework for server-rendered React applications that’s concerned exclusively with the UI layer of our application; it cares about efficiently rendering our UI.

## Tying together Next.js and Apollo

Next has a unique way of loading data, every page has a lifecycle hook function `getInitialProps` which is called while loading on the client as well as the server. This is a very nice separation of our data logic from the UI logic and is what enables us to use the full power of Apollo and Next as both of these frameworks make no assumptions about each others layer and provides the perfect abstraction to connects these two layers.

Apollo client's react library [react-apollo][8] provides a helper function `getDataFromTree` which recursively traverses the supplied React component and executes every `graphql()` query it finds, storing the result in its local cache (configured by us).

The excellent [example][1] from the Next.js repo shows how we can combine these two functionalities by creating a HOC which wraps `getDataFromTree` within `getInitialProps` and returning the cached data as a plain Javascript Object

Here we make sure to use `getDataFromTree` on both the client and for SSR, altering the example like so:

```javascript
import React from 'react'
import PropTypes from 'prop-types'
import { ApolloProvider, getDataFromTree } from 'react-apollo'
import Head from 'next/head'

import initApollo from './initApollo'

export default ComposedComponent => {
  return class WithData extends React.Component {
    static displayName = `WithData(${getComponentDisplayName(
      ComposedComponent
    )})`
    static propTypes = {
      serverState: PropTypes.object.isRequired
    }

    static async getInitialProps(ctx) {
      // Initial serverState with apollo (empty)
      let serverState

      // Evaluate the composed component's getInitialProps()
      let composedInitialProps = {}
      if (ComposedComponent.getInitialProps) {
        composedInitialProps = await ComposedComponent.getInitialProps(ctx)
      }

      // Run all GraphQL queries in the component tree
      // and extract the resulting data
      const apollo = initApollo();

      try {
        // Run all GraphQL queries
        await getDataFromTree(
          <ComposedComponent ctx={ctx} {...composedInitialProps} />,
          {
            router: {
              asPath: ctx.asPath,
              pathname: ctx.pathname,
              query: ctx.query
            },
            client: apollo,
          }
        )
      } catch (error) {
        // Prevent Apollo Client GraphQL errors from crashing SSR.
        // Handle them in components via the data.error prop:
        // https://www.apollographql.com/docs/react/basics/queries.html#graphql-query-data-error
      }

      if (!process.browser) {
        // getDataFromTree does not call componentWillUnmount
        // head side effect therefore need to be cleared manually
        Head.rewind()
      }

      // Extract query data from the Apollo store
      serverState = {
        apollo: {
          data: apollo.cache.extract()
        }
      }

      return {
        serverState,
        ...composedInitialProps
      }
    }

    constructor(props) {
      super(props)
      this.apollo = initApollo(this.props.serverState.apollo.data)
    }

    render() {
      return (
        <ApolloProvider client={this.apollo}>
          <ComposedComponent {...this.props} />
        </ApolloProvider>
      )
    }
  }
}
```

## Prefetching Data

Next has [in-built][2] support for prefetching pages by adding the `prefetch` prop to the `<Link>` component or imperatively calling `Router.prefetch('/dynamic')}`. However this fetches the page's JS code but not its data/initial props.

This results in instant page loads but, some would argue even worse, a loading indicator shown for however long it takes to get the relevant page data from your API.

Since we know our stack we can optimise performance using that knowledge. We know that with every call to `getInitialProps` of a component wrapped in our `WithData` HOC, our queries are cached in the local ApolloClient store. With that all we have to do is get a reference to our top level page component and manually call its `getInitialProps` method!

```javascript
import Router from 'next/router'
import { format, resolve, parse } from 'url'

export default prefetch = async (href) => {
  // if  we're running server side do nothing
  if (typeof window === 'undefined') return

  const url =
    typeof href !== 'string'
      ? format(href)
      : href

  const { pathname } = window.location

  const parsedHref = resolve(pathname, url)

  const { query } =
    typeof href !== 'string'
      ? href
      : parse(url, true);

  // get component reference
  const Component = await Router.prefetch(parsedHref)

  // fetch the component props
  // and cache locally, handled within getInitialProps
  if (Component && Component.getInitialProps) {
    const ctx = { pathname: href, query, isVirtualCall: true }
    await Component.getInitialProps(ctx)
  }
}
```

We can then call this throughout our application, like for example:
 `<a onMouseOver={() => prefetch("/product?sku=0001")}>`

 This will load not only our product.js page but also all the specific graphql queries into the ApolloClient store.

[1]: https://github.com/zeit/next.js/tree/canary/examples/with-apollo
[2]: https://github.com/zeit/next.js#with-link-1
[3]: https://dev-blog.apollodata.com/@adamsoffer
[4]: https://dev-blog.apollodata.com/whats-next-js-for-apollo-e4dfe835d070
[5]: https://www.apollographql.com/client/
[6]: https://github.com/zeit/next.js
[8]: https://github.com/apollographql/react-apollo
[9]: https://github.com/zeit/next.js/pull/3973
[10]: https://github.com/zeit/next.js/pull/3525
[11]: https://github.com/zeit/next.js/tree/canary/examples/with-data-prefetch
