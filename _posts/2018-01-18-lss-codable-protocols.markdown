---
title: "Long Story Short: Codable Protocols"
layout: post
date: 2018-01-18
tags: swift, software
draft: true
---
As developers, I’m sure we all have been in a place where we’ve had to deal with JSON in our lives. In my previous article, I detailed out how to deal with networking in Swift. Although the pattern discussed in the previous article still works excellently, we relied heavily on the `JSONSerialization` class to convert data back and forth to JSON.

With Swift 4, there are three fancy protocols that we could leverage to build a better model to deal with networking data:

* [Codable](https://developer.apple.com/documentation/swift/codable)
* [Encodable](https://developer.apple.com/documentation/swift/encodable)
* [Decodable](https://developer.apple.com/documentation/swift/decodable)

**Encodable** Protocol - defines a type that can be encoded into another form/representation for instance, JSON.

**Decodable** Protocol - defines a type that can be decoded from another representation into a native Swift type.

**Codable** Protocol - simply is a wrapper over **Encodable** and **Decodable**  Protocols, and represents a type that can both be encoded and decoded.

# The Fundamentals
In this section, we will discuss the concepts behind the Codable Protocols, using the example from the previous article:

{% highlight swift %}
[
  {
    "title": "Pulp Fiction",
    "actors": [ 
      { "name": "Samuel L Jackson" }, 
      { "name": "John Travolta" }
     ]
  },
  {
    "title": "Ocean's Eleven",
    "actors": [
      { "name": "George Clooney" },
      { "name": "Brad Pitt" },
      { "name": "Matt Damon" }
    ]
  },
  {
    "title": "Goodfellas",
    "actors": [
      { "name": "Robert De Niro" },
      { "name": "Ray Liotta" }
    ]
  },
  {
    "title": "I Am Sam",
    "actors": [
      { "name": "Sean Penn" }
    ]
  }
]
{% endhighlight %}

Looking at the data, we realise that we could have two types to model this:

* `Movie`
* `Actor`

Swift 4 makes it easier to make types codable. As long as the type declares its properties using types that are already codable, by simply adding `Codable` to the inheritance list on the definition it triggers the conformance to Codable automatically. 

This means the type now can be encoded/decoded using a `JSONEncoder` or a `PropertyListEncoder` to encode and decode by itself, even though we didn’t have to implement anything for it.

The types that are natively codable include:

* `String`
* `Int`
* `Double`
* `Date`
* `Data`
* `URL`

And as long as the elements they contain are _Codable_, the following types can also be considered as Codable:

* `Array`
* `Dictionary`
* `Optional`

**Code**:

{% highlight swift %}
struct Actor: Codable {
 let name: String
}

struct Movie: Codable {
 let title: String
 var actors: [Actor]
}
{% endhighlight %}

In the above example, we’ve declared both our types as Codable and the automatic conformance is triggered without having to implement anything more because we simply are trying to encode/decode types that are already Codable.

# Encoding
Now that our types are `Codable`, we can now get to the exciting part, trying to encode our type into say, JSON.

To encode the `Movie` type into JSON. We use an instance of `JSONEncoder`:

{% highlight swift %}
// Movie to be encoded
let movie = Movie(title: "Goodfellas")

// Encoder
let encoder = JSONEncoder()

// Encoded Data
let encodedMovie = try encoder.encode(movie)
{% endhighlight %}

Since the properties already conform to Codable, we were able to use the encoder directly to encode the movie.

# Decoding
Now that `encodedMovie` contains the encoded data, typically as `Data`, we can see how we could possible decode this into native movie type:

{% highlight swift %}
// Data to be decoded
let encodedMovie: Data?
 
// Decoder
let decoder = JSONDecoder()

let movie = try decoder.decode(Movie.self, from: encodedMovie)

// `movie` is a native swift object now
// movie.title is valid
{% endhighlight %}

# Coding Keys
Codable types, can optionally specify an `enum` called `CodingKeys` that conforms to `CodingKey` protocol. When this particular `enum` is present, the cases specified are the list of properties that are included when the type is encoded or decoded.

{% highlight swift %}
struct Movie: Encodable {
  enum CodingKeys: CodingKey {
    case title, actors
  }
}
{% endhighlight %}

# Doin’ it manually

Things aren’t always nice. They’re not always easy. You may often find yourself in a place where your Swift type has a different structure than the JSON you have to deal with. In such cases the way to go is:

* Let Swift know about the structure of your encoded form
* Implement an init on the type reflecting the structure of the JSON or the encoded form

**Code**:

Assuming we have a property called `meta` on `Movie` which in JSON is nested like:

{% highlight swift %}
[
  {
    "title": "Goodfellas",
    "actors": [ { "name": "Robert De Niro" }, { "name": "Ray Liotta" } ]
    "meta": {
        "rating": 8.0 
    }
  }
]
{% endhighlight %}

Because our JSON data contains a second level of nested information for the rating property, the type’s `Codable` protocol use two enumerations that each list the coding keys on a particular level.

We can extend the `Movie` type to reflect this structure by using two enumerations to list the Coding Keys used on each level.

{% highlight swift %}
struct Movie: Codable {
  let title: String
  var actors: [Actor]
  let rating: Double

  enum CodingKeys: String, CodingKey {
     case title
     case actors
     case additionalInfo
  }

   enum AdditionalInfoKeys: String, CodingKey {
      case rating
   }
}
{% endhighlight %}

To be able to support `Decodable` for our data with a nested structure, we can extend the `Movie` to conform to `Decodable` by implementing its initialiser:

{% highlight swift %}
extension Movie: Decodable {
  init(from decoder: Decoder) throws {
    let values = try decoder.container(keyedBy: CodingKeys.self)
    title = try values.decode(String.self, forKey: .title)
    actors = try values.decode([Actor].self, forKey: .actors)

    let additionalInfo = try values.nestedContainer(keyedBy: AdditionalInfoKeys.self, forKey: .additionalInfo)
    rating = additionalInfo.decode(Double.self, forKey: .rating)
  }
}
{% endhighlight %}


That should take care of decoding nested data from JSON.


Codable Protocols are very handly, and make dealing with an API a little easier, so the next time you’re looking to work with an API, give Codable Protocols a shot. 

Feel free to leave any feedback, or questions behind.

👨🏻‍💻

