# A Case Against REST APIs

REST REST REST. REST has consumed the zeitgeist of API design to the
point where it doesn't mean anything anymore. Certainly the APIs
that most people are writing are **not** of a purely RESTful design.

Most of the time it's a synonym for "An API which tries to organize
its routes in heirarchical segments, sometimes embedding identities
of objects in those routes, and mostly making use of the primary couple
of HTTP verbs.".

As we proceed, I'm going to attempt to poke some holes in the reasons
why you might prefer to write a REST-style API, and suggest (what
I think is) a simpler, alternative that is more consistent and
requires less ideological debate.

## Some basic REST-ish design

> What's wrong with REST? I like it.
> ... more conversation ...
> At work our APIs are mostly single resources with various commands you
> have to send them
>
> -- anonymous

Hey now anonymous friend, that sounds decidedly not REST-like. But to be
honest, for a majority of the API I've ever personally interacted with,
this is not an uncommon pattern.

Consider the canonical REST example of authors and books, for some bookstore:

`GET /author`

ok.

`GET /author/1/book`

fine, sure.

`POST /author/1/book/3/markOutOfStock`

Hey now, you got lazy decided to go off the R(EST)ails! REST endpoints are supposed to
be Resources! It's in the name! You should have written this instead!

`PATCH /author/1/book/3 {"outOfStock": true}`

But to me, all this implies is that you must have a single route handler for your "book"
resource, that now needs to unnecessarily be aware of all possible mutations which might
possibly alter the state of a book.

So you'll inevitably end up with a mega endpoint with a bunch of unrelated, conditional
logic in it; based on a number of purely optional fields...or you'll attempt to subdivide your
book resource with sub-resources so that you can keep each handler simple.

In either case, you've lost me. There is now a tension between the behavior I want to produce,
marking a book as out of stock, and the process I must go through to produce that behavior.
SRP (single responsibility principle) makes me want to define this behavior in isolation;
but the API design is pushing me to spend extra effort doing otherwise.

## An aside on GET Requests

Now I happen to think GET requests are just kind of trash. What’s wrong with GET, I
thought I was here to hear about REST? you might ask.

GET requests are fundamental to the REST system. You want some information, you will execute
a GET request.

I say GET has no business being used for most APIs. For APIs, a GET request is fraught with issues.
With when you execute a GET request, you are probably making some internal assumptions
about what's going to happen when you make that request.

- That you’re…GETting something? **YES**. You’re getting something! And
  that “thing”, a web page, a file, an image, is something that you
  think exists.

- That it’s idempotent? Sure, I’d be worried if a GET request could
  mutate any sort of state

- That it’s infallible? Well…maybe. Obviously a GET request can fail:
  network failure, the thing you’re GETing does not exist, maybe you
  don’t have permission. But otherwise, maybe it makes sense that it
  shouldn’t fail. You’re retrieving something, you have no other
  options, right?!

### That you're GETting something?

You're getting **something**. In REST, you're pulling some canonicalized
object. It's a "book", so you get a book back.

What happens when a user wants list of books keyed on some attribute,
a single book with a specific set of fields, or a book with the fields of
some sub-resource nested beneath it.

REST doesn't tend to like these scenarios because its whole premise is that
every endpoint is a resource. So you have a few options.

You can use query params to configure your route to output that resource
in different shapes. Hot take, I dont especially like this because it makes
all your endpoints have polymorphic output shapes that aren't easily
documented with tools such as OpenAPI, RAML, etc.

You could define a 2nd route `book/` and `bookWithExtraFields/`. Hot take,
that's even worse. Those are both the same underlying object, you're just
creating fake extra "resources" to work around REST's poor semantics.

Not that I'm advocating for GraphQL, but GraphQL is obviously specifically
optimized for this scenario. It's got other faults, but at least you have
the flexibility to get whatever output shape you're looking for, without
having to jump through hoops to get it..

### That it's idempotent?

Maybe this is a failing of web frameworks, but there’s also nothing stopping you from
COMMITting some changes in pretty much any web request. Maybe
idempotency is a valueable invariant to assert, but by ditching GET it’s
at least something you dont have to debate about. Does generating an
excel file count as something you’d think you could do with a GET
request? What if it took 2 seconds? What if we needed to (as an
implementation detail) write the file to disk temporarily? What if we
cached the value in redis to avoid the 2s cost on each request? At what
point do we decide GET is no longer appropriate?

### That it’s infallible?

Again consider `/author/1/book`. Now I want to filter to a specific set of books
`/author/1/book?ids=1,2,3,4`.

Should this request fail if books 3 and 4 do not exist? Should it silently ignore
bad ids and only return the ones that do exist in the filter set? From a REST/GET
perspective, I would say the latter makes the most sense.

#### A not so short aside on path/query params

Once you toss GET requests in the trash where they belong, you’ll
realize that the only reason to reach for something like a path
or a query param are due to the deficiencies of GET requests.

Your only option for configuring the behavior of GET requests are by embedding
random fragments of strings into the url of your request and trying to
parse them into something logical after the fact. And I've encountered tons
of examples of always-required query params! How stupid is that!

A typical rest GET request might look like:
`/authors/1/articles?sort=name+asc&limit=40&ids=1,2,3,4&offset=5&includesFields=name,age,createdDate&valid=true&createdAfter=2020-01-01T05:33:11`

This is convenient in exactly one way: You can copy-paste the URL and
get the same response.

Probably.

If your API token hasn’t expired...

And the authentication mechanism is also in the URL...If it's in the headers, then you're shit out of
luck anyways...

And the data hasn’t offsets and filters haven’t been invalidated...

There are also practical limits to the length of your URL...Ever use LinkedIn or
Facebook's APIs? Both have systems for making GET requests to get analytical results
from their APIs. And both also provide POST requests for the same operations because
they apply length limits to the URL, and some perfectly valid requests exceed those
length.s

Here are some other reasons the above example sucks:

- The URL is difficult to read! You've got this big flat, raw string. Oh wait it's not raw, it's been
  URLencoded.
- `sort=name+asc`. You'll see I've now defined a one-off DSL for describing the behavior I want, and
  now I need to go write a parser for that DSL. and the next one, and the next one.

  Take a look at something like Linkedin’s [Marketing
  API](https://docs.microsoft.com/en-us/linkedin/marketing/integrations/ads-reporting/ad-budget-pricing),
  here’s the primary offending example.
  `targetingCriteria=(include:(and:List((or:({encoded urn:li:adTargetingFacet:staffCountRanges}:List({encoded urn:li:staffCountRange:(501,1000)}))))))`
  Was designing this DSL **really** easier than just using a POST request
  with the equivalent, intelligibly nested JSON structure? No.

- Path params tend to get a bit of better support in web frameworks. Typically libraries will help
  extracting path parameters and casting them to simple types like numbers.

  Ever run into priority problems where `/route/foo` routes to the wrong place because you have
  a static sub-resource "foo", and also a dynamic one, but they were ordered incorrectly? Oh yea,
  me neither.

With a POST request (or PUT or DELETE…although if we’re giving up GET
and REST, what’s the point in deciding to use anything but POST?), you
have a fancy new option: The request body.

Even with a JSON body, you’ve got a significant leg up against GET.
string, null, number, boolean, list, object. You’ve got some types to
work with, to start describing things in a more structured manner.

```json
POST /articles/list
{
    "authorId": 1,
    "sort": "name",
    "sortDirection": "asc",
    "limit": 40,
    "ids": [1, 2, 3, 4],
    "offset": 5,
    "includesFields": ["name", "age", "createdDate"],
    "valid": true,
    "createdAfter": "2020-01-01T05:33:11"
}
```

Much **MUCH** better. I’m now not surprised if you return a 400 when I
incorrectly format some field or another. I might even have documented
the schema of the body with OpenAPI or RAML so that you know precisely
what you’re allowed to send me.

Let alone the fact that that Json isn't your only option. GRPC anyone?

## REST Heirarchy

REST typically implies that there is a single, clear heirarchy of objects
that exist, and there should only be one way to access that object.
But sometimes there’s no clear heirarchy between your domain
objects.

I’ve found that this typically boils down to where you usually end up
with multiple routes to the same “object”, and then you add query params
to both, which allow you to filter to the subset related to the other.

This isn’t precisely “not REST”, but it’s just a sign that you’re close
to the other alternatives, which have a lower impedance mismatch with
what you're trying to design.

## RPC-style

I'm about to describe a particular way of writing an API, with specific
guidelines on how to structure the endpoints and such.

To be clear, these are not new ideas. It's essentially no different
than JsonRPC or GRPC in practice, but is just what I think is a decent
guideline for producing consistent, APIs that are easy to test and easy to write.

Tenets:

- thou shalt only use POST requests
- thine url paths shall be noun(s) followed by a verb
- thou shall create a new route for each new input/output shape
- thou shalt not use path params
- thou shalt not use query params
- thou shall limit thine use of http codes

### thou shalt only use POST requests

We've established that GET requests are stupid elsewhere.

DELETE, POST, PUT, PATCH, they're all pretty much the same as a POST request.
Given tenet #2 though, there's not really much point in using them.

### thine url paths shall be noun(s) followed by a verb

`/noun/nown/noun/verb`

Just a series of nouns (for “heirarchy”) with a verb at the end.

Here's some examples:

- `/author/create`
- `/author/book/create`
- `/author/book/list`
- `/author/book/get`
- `/author/book/getSpecificShape`

You get similar heirarchical semantics to that of REST, but you can choose
any arbitrary action as the thing being performed during the API call.

### thou shall create a new route for each new input/output shape

As a direct result of the previous tenet, you should define a new route
any time you might have previously had requests with polymorphic input or output
shapes.

The path to the route ought to define the shape of the response. In the
same way that you might define a function and it’s return type, you
shouldn’t be able to configure the output shape through input
parameters.

Otherwise you’re simply increasing the complexity of your consumers by
requiring that they first check the response shape/type before using it.

### thou shalt not use path params

There is no need, any path param you might have used can instead be a POST body
field.

### thou shalt not use query params

There is no need, any query param you might have used can instead be a POST body
field.

### thou shall limit thine use of http codes

In the RPC style, the usefulness of various http codes is vastly
diminished. Either your request succeeded, it failed in some
expected/validated manner, or it failed spectacularly.

There’s no need for both 200 SUCCESS and 201 CREATED when your routes
look like `/article/create`, `/arcticle/get` and `/arcticle/list`.

There’s nothing worse than spending half an hour debating whether to use
400, 422, 409, or whatever. There’s little need for various 4xx-type
responses because you can simply decide on a single 400 (or whatever)
response code, returning a specific output shape that more
specifically describes the problem. A response code is simply not enough
context on its own.

401/403 aren't exactly an exceptions, as much as they are an admission that
certain response codes are more useful than others. Many of the canonical 4xx
response codes are dedicated to various kinds of validation, in
particular request-specific validation.

It's arguable that 401/403 are at least, one level removed from that.
You’ll typically authenticate outside the context of specific routes; at the
service level.

That’s not to imply that one ought to single out 401/403. Simply that I
have a far less specific opinion about them. a 400 would likely serve
just as well in most cases. However, practical considerations that
existing alerting and/or other tooling often already has specific behavior
for certain kinds of failures at the http-code level.

And finally 5xx response codes are often outside your control or due to
bugs. A reverse proxy between your service and the caller might raise a
502, or a 504 on your behalf. Your framework might handle uncaught
exceptions or errors by returning a 500. None of this is really relevant
because we’re largely discussing business logic code. You wouldn’t
generally intentionally raise any of these response codes, so how far
you go towards consolodating the codes you use in the 5xx realm is
mostly irrelevant.
