## Artsy's GraphQL Workshop

**Who is this aimed at:** Engineering Collaborators like design, marketing, product and anyone who is interested in
how the pieces come together with our API.

### Overview:

- What is an API?
- Why do we need an API?
- How come there are different types of API choices?
- What are some of our choices?

### Metaphysics:

- What is metaphysics?
- Why did we build it?
- Why is it so central to writing apps at Artsy?

### GraphQL

- What problem was it created to solve?
- What problems does it solve for Artsy?

### How GraphQL works

We're going to use GraphQL to get info on popular Artists at Artsy.

1. Open up [Metaphysics Production](https://metaphysics-production.artsy.net/v2)
1. Add the following code:

   ```graphql
   {
      highlights {
        popularArtists(size: 1) {
          name
        }
      }
    }
   ```

1. Go through the syntax
1. Command click on `popular_artists`
1. Run the query

### GraphQL syntax

JSON-like. What is JSON?

```json
  {
  "data": {
    "highlights": {
      "popularArtists": [
          {
            "name": "Pablo Picasso"
          },
          {
            "name": "Banksy"
          },
        ]
      }
    }
  }
```

JSON is used to communicate between nearly all of our different services, it's considered a human-readable data
format. The JS in JSON is JavaScript, and that's pretty popular.

GraphQL's syntax aims to emulate the shape of the JSON data that you're going to receive.

##### Scopes

`{` and `}` represent a scope like in JavaScript: you may have seen something like this before:

```js
if (name == "Banksy") {
  doSomethingJustForBanksy()
}
```

Where the code inside the braces (`{` `}`) will only run when that `if` statement is true. In the case of GraphQL,
this is what happens when you want to get more information from something.

```graphql
{
  # global scope
  popularArtists {
    # inside here we can access properties on the PopularArtists type
  }
}
```


It roughly looks like:

```graphql
{
  fieldObj {
    field1
    field2
  }
}
```

We call the text inside the braces a "field", so in our current case, there is a field `popular_artists`. That has
fields `artists`, and that has fields like `name`.


So we start at global scope, which has things like `artist`, `artwork`, `popular_artists` and `order` then we start
narrowing down the data we want.

So, with this query:

```graphql
{
  # global scope
  highlights {
    # PopularArtists
    popularArtists {

      # A list of Artists
      # Artist scope
      artists {
        name

        # A list of Artworks
        # Artwork scope
        artworks {
          title
        }
      }
    }
  }
}
```

You can see how we can keep diving deeper through the data (we can basically do this forever, asking for an
artists' artworks, which asks for the artist's for those artworks and so on)

##### Optional Params

Like the if statement in JavaScript, GraphQL uses `(` and `)` to contain arguments. Let's change out
`popularArtists` query to include params:

```graphql
{
  highlights {
    popularArtists(size: 1) {
      name
    }
  }
}
```

Params are a list of color separated `[name]:[value]` couplets. These are built to emulate how parameters work in
JavaScript (but with names.)

These can be simple, but sometimes aren't because we use some advanced GraphQL features. Here are some small
queries that have different styles of params.

Using an array of constants for the `artworksConnection` name, having some boolean values, and including params on a
resolver that doesn't have children:

```graphql
{
  artworksConnection(first: 5, aggregations: [TOTAL, PRICE_RANGE], acquireable: true, atAuction: false) {
    edges {
      node {
        title
        image {
          url(version: "original")
        }
        price
        artistNames
      }
    }
  }
}
```

### This might be enough!

This is how we build a page on the app/website - this is the query used by the Artwork page on the iPhone app:

```graphql
{
  artwork(id: "christo-the-gates-project-for-central-park-5") {
    slug
    internalID

    artist {
      slug
      internalID
      name
      years
      birthday
      nationality
      blurb
      image {
        url
      }
      sortableID
    }

    partner {
      name
      slug
      defaultProfileID
      isDefaultProfilePublic
      type
      href
    }

    images {
      imageVersions
      imageURL
      isDefault
      originalHeight
      originalWidth
      aspectRatio
      maxTiledHeight
      maxTiledWidth
      tileSize
      tileBaseURL
      tileFormat
    }

    dimensions {
      cm
      in
    }

    attributionClass {
      name
    }

    editionSets {
      saleMessage
      isSold
      isForSale
      dimensions {
        in
        cm
      }
    }

    availability
    additionalInformation
    category
    collectingInstitution
    date
    exhibitionHistory
    editionOf
    imageRights
    isForSale
    isPriceHidden
    isSold
    isInquireable
    shippingInfo
    shippingOrigin
    literature
    medium
    price
    provenance
    published
    saleMessage
    series
    signature
    title
  }
}

```

It's a lot of data, but it's a complex page. We do other work on the page, but this gets the main artwork metadata.

##### Docs

So, we know enough to be dangerous. Now what? I think it's worth digging into the how we can figure out what how to
find things we're interested in.

**Using the sidebar**

The sidebar on the left represents the types of objects that exist inside our API. In simple, everything is a type.
There are two "root" types `Query` and `Mutation`. A mutation, roughly, represents a change to the database, like
following an Artist.

We're interested in the other: `Query`. If you click on the word `Query` in the right sidebar, it will show you all
of the available fields. These are the things that you can start your query with from the global scope.

This gives you the ability to jump through the connections between things without writing code, and you can see our
docs about what things are and how they come together.

**Guess by typing**

Generally, I work this way.

##### Logging in

We're currently working with all public data. Sometimes, you'll need to log in with your user account. This is a
bit tricky. To do this you're going to need to set up some request headers. This is a bit of a thing, sorry. I
might be able to improve this in the future, but for now you'll need to download an app.

1. Go here: https://electronjs.org/apps/graphiql
1. Click on `GraphiQL-0.7.x.dmg` in the sidebar
1. Click on the new DMG in your downloads
1. Drag the app into Applications
1. Open it from your Applications folder

That opens up the GraphQL app. First, let's just to Chrome. Open the artsy website
[artsy.net](https://www.artsy.net/) and we need to go into the developer tools.

1. Press `alt + cmd + i`, this should open up a window that represents the developer tools for Chrome
1. In the bottom half of the window, there is a section called "Console". You want to run this code in there:

   ```js
   sd.CURRENT_USER.accessToken
   ```
1. This will output something like:

  ```js
  "eyJ0eXAiOiJKV4QiLCiJIUzI1NiJ9.asdaffvcsdkjsbksg .bY8n_s0dK6P6VHASDA3gtcV7QVi_iTx2GXr58mCzk"
  ```

1. Select the text and copy it.

Now we have your access token, let's add it to the GraphiQL app.

1. Click on "Edit HTTP Headers"
1. Click on "Add Header"
1. Paste your token in the "Header Value"
1. Set the "Header name" to be "x-access-token"
1. Hit save
1. Click outside of the white box to get rid of the modal
1. Set the GraphQL Endpoint to be: https://metaphysics-production.artsy.net/v2

OK! Now you are logged in.

Let's verify this by running this query:

```graphql
{
  me {
    name
    email
  }
}
```

If you get a response like:

```json
{
  "data": {
    "me": null
  }
}
```

Then the header for your access token is not hooked up correctly.

##### Now you're connected

Let's take a look at your followed Artists:

```graphql
{
  me {
    followArtists {
      artists {
        name
      }
    }
  }
}
```

Nearly all of the information related to a user account is structured inside the `me {`.

## What now?

Ideally, I'd like some ideas on bits of data we can look up, and I'll try show how to find it.

Other than that, I've left a bunch of example queries in the [`examples/`](examples) folder in this repo.

