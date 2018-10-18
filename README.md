## Artsy's GraphLQ Workshop

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

1. Open up [Metaphysics Production](https://metaphysics-production.artsy.net/)
1. Add the following code:

   ```graphql
   {
     popular_artists {
       artists {
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
  {
  "data": {
    "popular_artists": {
      "artists": [
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
  doSomething()
}
```

Where the code inside the braces (`{` `}`) will only run when that `if` statement is true. In the case of GraphQL,
this is what happens when you want to get more information from something.

```graphql
{
  # global scope
  popular_artists {
    # inside here we can access properties on the PopularArtists type
  }
}
```

So we start at global scope, which has things like `artist`, `artwork`, `popular_artists` and `order` then we start
narrowing down the data we want.

So, with this query:

```graphql
{
  # global scope

  popular_artists {
    # PopularArtists

    artists {
      # A list of Artists
      # Artist scope

      name
      artworks {
        # A list of Artworks
        # Artwork scope

        title
      }
    }
  }
}
```

You can see how we can keep diving deeper through the data (we can basically do this forever, asking for an
artists' artworks, which asks for the artist's for those artworks and so on)

##### Optional Params

Like the if statement in JavaScript, GraphQL uses `(` and `)` to contain arguments. Let's change out
`popular_artists` query to include params:

```graphql
{
  popular_artists(size: 1) {
    artists {
      name

      artworks {
        title
      }
    }
  }
}
```

Params are a list of color separated `[name]:[value]` couplets. These are built to emulate how parameters work in
JavaScript (but with names.)

These can be simple, but sometimes aren't because we use some advanced GraphQL features. Here are some small
queries that have different styles of params.

Using a constant for the `trending_artists` name:

```graphql
{
	trending_artists(name: ARTIST_INQUIRY){
    artists {
      name
    }
  }
}
```

Using an array of constants for the `filter_artworks` name, having some boolean values, and including params on 
a resolver that doesn't have children: 

```graphql
{
  filter_artworks(aggregations: [TOTAL, PRICE_RANGE], acquireable: true, at_auction: false) {
    aggregations {
      counts {
        name
        count
      }
    }
    
    hits {
      id
      title
			image {
        url(version:"original")
      }
      price
      artist_names
    }
  }
}
```

### This might be enough!

This is how we build a page on the app/website - this is the query used by the Artwork page on the iPhone app:

```graphql
artwork(id: "christo-the-gates-project-for-central-park-5") {
  id
  _id

  artist {
    id
    _id
    name
    years
    birthday
    nationality
    blurb
    image {
      url
    }
    sortable_id
  }

  partner {
    name
    id
    default_profile_id
    is_default_profile_public
    type
    href
  }

  images {
    id
    image_versions
    image_url
    is_default
    original_height
    original_width
    aspect_ratio
    max_tiled_height
    max_tiled_width
    tile_size
    tile_base_url
    tile_format
  }

  dimensions {
    cm
    in
  }

  attribution_class {
    name
  }

  edition_sets {
    id
    sale_message
    is_sold
    is_for_sale
    dimensions {
      in
      cm
    }
  }

  availability
  additional_information
  can_share_image
  category
  collecting_institution
  date
  exhibition_history
  edition_of
  image_rights
  is_for_sale
  is_price_hidden
  is_sold
  is_inquireable
  is_acquireable
  shippingInfo
  shippingOrigin
  literature
  medium
  price
  provenance
  published
  sale_message
  series
  signature
  title
}
```

It's a lot of data, but it's a complex page. We do other work on the page, but this gets the main artwork metadata.

##### Docs


