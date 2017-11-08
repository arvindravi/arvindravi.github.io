---
title: "Networking with Swift"
layout: post
date: 2017-11-08
tags: swift, software
---
## Introduction
We all know why dependencies are bad, and writing software dependant on other software is not necessarily good. It probably reduces complexity for the time being, and helps you write code faster, but I wouldn’t say it’s exactly the way things are to be done, from my experience.

Writing software independent of 3rd party libraries, helps build robust software that won’t go out of date with time. I’ve used networking libraries in the past, some really excellent ones (*_cough Alamofire cough_*) to abstract and make things easier while building the networking layer. But of late, I’m trying to write code with zero to no-dependencies. Especially, when working with the iOS Platform. The iOS platform comes baked in with most, if not all, things that you need to build out your app. 

Networking is something most apps need to deal with these days, the apps on your phone have to communicate with the internet one way or the other, process requests/responses, and display information. When that’s being done all over the place in your code, it’s nightmare.

Recently, I came across a Swift talk by Florian Kugler and Chris Eidhof of objc.io where they talk about writing the Networking Layer for an app, and how best to do that. In this post, I intend to explain their process with some fundamental Swift concepts, and how to keep concerns separate and why it’s good.

## The Architecture: Model, Resource, Webservice
Architecting a networking layer can be a task, a difficult one at that, especially without direction. Once you know how to do something, doing something isn’t hard. Let’s see one way to architect your networking layer that you can consider as a direction to write your own.

There are three concerns that need to be addressed when dealing with Networking:
1. Send a request to a server for a resource
2. Get a response back
3. Parse responses into native swift types

We’ll see how to address each of this with Swift.

- Send a request to a server for a resource 
- Get a response back

Usually when you’re sending a request over to a server, it’s for a resource of some kind. This could be a post, an article, a movie, or anything. Let’s call this a resource from now.

We could handle this by having a class do that for us, say a **Webservice** class that could `load` a resource, that returns a response.

- Parse responses into native swift types

Usually the responses we get back from server these days is in JSON. So, the task at hand is to parse JSON into native swift types. There are many ways to do this, but let’s see how to handle it so its nicer and cleaner and in a way you don’t have to deal with it each and every time you need to request for a response.

Let’s assume we have the following milestones:

**Milestone One**: Get Binary Data<br>
**Milestone Two**: Data —> JSON  —> Native Types<br>
**Milestone Three**: Clean up

## Webservice — Class that handles requests
We need a way to send requests to a server for a resource, don’t we? We’ll call that class `Webservice`. This class could have a method `load` that could be responsible to `load` a `resource`. We’ll see how to build this out in a while.

## Resource
We also need a way to tell the Webservice class to load a `resource`, we can accomplish that by having a `Resource` object that encapsulates the request `URL` and a way to parse the response.

## Model — Native Swift Objects with Value Types
Finally, you need native types of the data that you receive from the server to be able to use it across the app. We’ll use structs for each model to address that.

## Code

Example:

Let’s assume we have json data being served from a server that looks like this:

{% highlight swift %}
[
  {
    "title": "Pulp Fiction",
    "actors": ["Samuel L Jackson", "John Travolta"]
  },
  {
    "title": "Ocean's Eleven",
    "actors": ["George Clooney", "Brad Pitt", "Matt Damon"]
  },
  {
    "title": "Goodfellas",
    "actors": ["Robert De Niro", "Ray Liotta"]
  },
  {
    "title": "I Am Sam",
    "actors": ["Sean Penn"]
  }
]
{% endhighlight %}

For the sake of this example, we’ll put this data in a file called `movies.json`, and try to build a networking layer in Swift that parses this data into native types.

### Resource

The `Resource` is a struct that encapsulates the `url` and a way to parse, a parse function, that helps parse the data from the URL. For example, In our case we could have a `MoviesResource` that could retrieve movies.

We could model Resource like —>

{% highlight swift %}
struct Resource<A> {
 let url: URL
 let parse: (Data) -> A?
}
{% endhighlight %}

- URL: A `url` property that denotes the URL of the resource
- `parse` function: A function that takes some `Data` and returns an object of the associated resource.

Let’s write an initialiser for a `Resource` in an extension like —>

{% highlight swift %}
extension Resource {
 init?(url: URL, parse: @escaping (Any) -> A?) {
   self.url = url
   self.parse = parse
 }
}
{% endhighlight %}

Here, along with the URL, we’re just passing a closure to the parse parameter and assigning it to the resource’s parse property.

Now, a movies resource could be initialised like —>

{% highlight swift %}
let url = URL(string: "http://localhost:8080/movies.json")!
let moviesResource = Resource<Data>(url: url) { data in
 return data
}
{% endhighlight %}

### Webservice
Now that we’ve seen how we could create a Resource type for our data, we need a way for us to fetch the data from the server. 

Luckily the `URLSession` class offers a way for us to do this easily without having to use any third party libraries, we’ll wrap this up in a function within a `Webservice` class so we could use it. 

We could write this like —>

{% highlight swift %}
final class Webservice {
 func load<T>(resource: Resource<T>, completion: @escaping (Any) -> ()) {
   URLSession.shared.dataTask(with: resource.url) { (data, _ \_) in
     completion(data)
   }.resume()
 }
}
{% endhighlight %}

We’re simply using a shared URLSession object to call the following method

{% highlight swift %}
URLSession.shared.dataTask(with: URL, completionHandler: (Data?, URLResponse?, Error?) -> Void)
{% endhighlight %}

with the resource’s URL parameter and call the completion with data, this will just call the closure with whatever data it retrieves at this point.

We could now use the Webservice by creating an instance of it like —>

{% highlight swift %}
let sharedWS = Webservice()
sharedWS.load(resource: moviesResource) { (result) in
 print(result)
}
{% endhighlight %}

This should return whatever the size of the data it retrieved was, when you run it a playground, you get something like this -->

{% highlight swift %}
Optional(324 bytes)
{% endhighlight %}

Now, this means we have a service trying to communicate with the server and returning response, but the response is in binary and not useful for us. 

🏁  **We have now reached Milestone One.** 

We’ll see how to get meaningful data and parse it to native types so we can use it.

### Models
Looking at the JSON data, we figure that we need a `Movie` type to represent the movies, and possibly an `Actor` type with a name property that can hold their names.

We could model this with native types like —>

{% highlight swift %}
// Actor
struct Actor {
 let name: String
}

// Movie
struct Movie {
 let title: String
 let actors: [Actor]
}
{% endhighlight %}

We now need a way to model JSON data with native types, we could use a type alias for a dictionary like —>

{% highlight swift %}
typealias JSONDictionary = [String: AnyObject]
{% endhighlight %}

We can now write an initialiser for a `Movie` like this —>

{% highlight swift %}
extension Movie {
 init?(dictionary: JSONDictionary) {
   guard let title = dictionary["title"] as? String,
     let actors = dictionary["actors"] as? [String] else { return nil }
   self.title = title
   self.actors = actors.flatMap(Actor.init)
 }
}
{% endhighlight %}

What’s happening here is pretty straightforward —>

The `guard` statement pulls out our fields `title` and `actors` from the dictionary, which is of type `JSONDictionary` and assigns it to the respective properties. We also `flatMap` the `actors` string and pass each string to the `Actor`’s init to instantiate actor objects within the Movie.

Now that we have a way to create `Movie` objects, let’s rewrite our `moviesResource` to return them —>

{% highlight swift %}
let moviesResource = Resource<[Movie]>(url: url) { data in
 let json = try? JSONSerialization.jsonObject(with: data, options: [])
 guard let dictionaries = json as? [JSONDictionary] else { return nil }
 return dictionaries.flatMap(Movie.init)
}
{% endhighlight %}

We do the following here:
- Instantiate `moviesResource` as a `Resource` that returns array of `Movie` objects
- Use `JSONSerialization` class to convert our data (`Data`) object into json `JSON`
- Typecast `json` into an array of `JSONDictioanry` objects
- `flatMap` over the `dictionaries` array to instantiate a `Movie` with each block of json

We now have a process setup that can parse the data into our required objects, but we still haven’t updated our `load` method to make use of the `parse` method that we’ve modified above.

This can look like —>

{% highlight swift %}
 final class Webservice {
   func load<T>(resource: Resource<T>, completion: @escaping (Any) -> ()) {
     URLSession.shared.dataTask(with: resource.url) { (data, \_, \_) in
       completion(data.flatMap(resource.parse))
     }.resume()
   }
' }
{% endhighlight %}

We modify our `load` method for a resource type `T` inside which we simply try to `flatMap` over the stream of data that gets loaded as `Data`, thanks to `URSession`,  and pass the resource’s `parse` method to it so the parse functionality we’ve written is useful, and send it to the completion handler.

Now, let’s see what data we’re able to load with our web service —>

{% highlight swift %}
sharedWS.load(resource: moviesResource) { (result) in
 print(result)
}
{% endhighlight %}

If you run this in a playground, you should see something like this —>

{% highlight swift %}
Optional([__lldb_expr_55.Movie(title: "Pulp Fiction", actors: [__lldb_expr_55.Actor(name: "Samuel L Jackson"), __lldb_expr_55.Actor(name: "John Travolta")]), __lldb_expr_55.Movie(title: "Ocean\'s Eleven", actors: [__lldb_expr_55.Actor(name: "George Clooney"), __lldb_expr_55.Actor(name: "Brad Pitt"), __lldb_expr_55.Actor(name: "Matt Damon")]), __lldb_expr_55.Movie(title: "Goodfellas", actors: [__lldb_expr_55.Actor(name: "Robert Di Niro")]), __lldb_expr_55.Movie(title: "I Am Sam", actors: [__lldb_expr_55.Actor(name: "Sean Penn")])])
{% endhighlight %}

Which means good news. You’re now able to get native objects from the json now. 

We could refactor the load method to give us something nicer —>

{% highlight swift %}
sharedWS.load(resource: moviesResource) { (result) in
 guard let movies = result as? [Movie] else { return }
 for movie in movies {
   print("\nMovie: \(movie.title)")
   for actor in movie.actors {
     print("Actor: \(actor.name)")
   }
 }
}
{% endhighlight %}

You’ll see this prettified output —>

{% highlight yaml %}
 Movie: Pulp Fiction
 Actor: Samuel L Jackson
 Actor: John Travolta
 
 Movie: Ocean's Eleven
 Actor: George Clooney
 Actor: Brad Pitt
 Actor: Matt Damon
 
 Movie: Goodfellas
 Actor: Robert Di Niro
 
 Movie: I Am Sam
 Actor: Sean Penn
{% endhighlight %}

🏁 **We have now reached Milestone Two.**

## Refactor
Now that we’ve seen how to get to what we need, we could get to cleaning up the code a bit, and refactoring it to make it much nicer.

We could start by moving our `moviesResource` into the `Movie` struct to make it more concise using an `all` property —>

{% highlight swift %}
struct Movie {
 let title: String
 let actors: [Actor]
 
 static let all = Resource<[Movie]>(url: url) { data in
   let json = try? JSONSerialization.jsonObject(with: data, options: [])
   guard let dictionaries = json as? [JSONDictionary] else { return nil }
   return dictionaries.flatMap(Movie.init)
 }
}
{% endhighlight %}

We could also simplify this further by abstracting away the `JSON` decoding part to the `Resource` type by creating an extension —>

{% highlight swift %}
extension Resource {
 init(url: URL, parseJSON: @escaping (Any) -> A?) {
   self.url = url
   self.parse = { data in
     let json = try? JSONSerialization.jsonObject(with: data, options: [])
     return json.flatMap(parseJSON)
   }
 }
}
{% endhighlight %}

Here, we convert `data` object into JSON and `flatMap` over it to recursively parse the closure to parse the json data, now our `Resource` could become simpler —>

{% highlight swift %}
struct Movie {
 let title: String
 let actors: [Actor]
 
 static let all = Resource<[Movie]>(url: url) { json in
   guard let dictionaries = json as? [JSONDictionary] else { return nil }
   return dictionaries.flatMap(Movie.init)
 }
}
{% endhighlight %}

🏁 **We have now reached our final milestone, Milestone Three.**

Now, we have code that’s cleaner, concise and keeps concerns separate. This pattern can be followed with whatever Resource that you want to work with.


You can download the files we worked with incase you want to try it our yourself:

- [movies.json](https://gist.github.com/arvindravi/5be87b14b9782a2b0957863310d87af7)
- [Neworking.playground](https://www.dropbox.com/s/zyryg0qtvxvqisq/Networking.playground.zip?dl=0)

## Conclusion

So, that was about how to write a Networking layer without any 3rd party libraries. Feel free to leave any feedback or questions!
