---
state: Draft
start-date: 2024-07-07
author: Rodrigo Arias Mallo <rodarima@gmail.com>
---

# Dillo RFC 002 - Rule-based content manipulation

## Abstract

Defines a rule-based language to describe how to manipulate content as it is
fetched or requested by Dillo. This rule mechanism allows rewriting web pages,
traduce other file formats to HTML and also implementing new protocols. It
supersedes the current DPI infrastructure.

## Motivation

One of the shortcomings of the Dillo plugin mechanism (DPI) is that it can only
operate at the protocol level. That is, a program is assigned to a protocol, for
example "gemini:" and then all browsing that requests URIs of that protocol is
forwarded to the given plugin.

The drawback of this design is that it mixes the content with the protocol. In
the case of the Gemini protocol, the usual file format is Gemtext, which is
similar to Markdown. However, if a Gemtext file is fetched via HTTP or locally
via the "file:" protocol there is no current way to translate it into HTML, in
the same way a Gemini plugin would do.

Another problem with the current design is that it can only operate at the
granularity of complete requests. For example, the user clicks on a link that
opens a given protocol and that is forwarded to the given plugin, without any
other possibility.

By allowing plugins to be able to rewrite content on their own, they can use
information of the current request to determine how to perform the rewrite
process. For example, a plugin may only operate on a set of domains, or when
certain HTTP headers are in the response.

## Design considerations

The goals of the design is to have a flexible mechanism to describe how to
perform the manipulation while we keep it simple to understand for users.

### Rule language

Using a simple rule language we can build a set of rules that can be quickly
evaluated in runtime. These rules have the capability to run arbitrary commands
that the user specifies, which are capable of manipulating the traffic.

They can also operate in such a way that they behave as endpoints, so they can
implement protocols on their own.

### Performance

As users can add a long list of manipulations with complicated matching
criteria, we should ensure that we don't introduce a lot of overhead in each
request or response.

A way to avoid this overhead is by having a restricted set of rules that can
only operate on data that is already parsed by Dillo, so it doesn't have to be
parse by each plugin.

### Domain matching

Let's consider the case where we want to match a particular domain. If we let
each plugin determine if the domain has to be intercepted or not, that would
cause the execution of every plugin in each request. However, by having a single
hash table where we store plugins that should process that request, we can
determine where to reroute the request in O(1) time.

Similarly, we could allow users to match domains by using a regex, but that
would introduce a much larger cost, as we would have to match all the regex
rules for every request. A simple solution is to match the domain first, and
then use the regex to further restrict the match. This will distribute the
regex matching overhead among the domains.

### HTTP header matching

Rules may choose to match if a header is present (or absent), or if it is
present and it contains a given value. To avoid parsing again the HTTP headers,
we perform the parsing from Dillo and then match the rules.

Only the rules that match the domain (with the optional domain regex) or the
ones that are for any domain should be processed here.

## Implementation details

Dillo currently builds a chain of modules that performs some processing on the
incoming and outgoing data:


    (0)  +--------+(1) +-------+(2) +------+(3) +-------+
    ---->| TLS IO |--->|  IO   |--->| HTTP |--->| CACHE |-...
    Net  +--------+    +-------+    +------+    +-------+
         src/tls.c     src/IO.c     src/http.c  src/capi.c

The user should be able to decide at which stage the rules are hooked. For
example, at (0) we TLS traffic is still encrypted, so there is only a limited
actions that can be done there.

At (1,2) we see the HTTP traffic, but it is still compressed (if any). At (3) we
see it uncompressed, and is the last step before being cached.

Here is an example where we introduce a new module "SED" that sees the incoming
uncompressed HTTP traffic and can perform modifications:

    Net  +--------+    +-------+    +------+    +=====+    +-------+
    ---->| TLS IO |--->|  IO   |--->| HTTP |---># SED #--->| CACHE |-...
         +--------+    +-------+    +------+    +=====+    +-------+
         src/tls.c     src/IO.c     src/http.c     |       src/capi.c
                                                   |
                                              +---------+
                                              | rulesrc |
                                              | ...     |
                                              +---------+

## Feature creep

This design introduces more complexity in the Dillo code base. However, trying
to manage this feature outside Dillo doesn't seem to be possible, as we need to
be able to reroute traffic on the different layers.

On the other hand, we can design the rule language in such a way that we only
allow operations that are quick to evaluate in runtime to reduce the overhead.

## Validation

When implemented, we should be able to do the following:

- Rewrite HTML pages to correct bugs or introduce new content such as meta
  information in the `<head>` that is rewritten as visible HTML elements. An
  example of such elements are RSS feeds.

- Patch CSS per page. As we can hook the rules to match different properties, we
  can use them to inject new CSS rules or patch the given ones to the user
  liking. This allows fixing broken rules or use fallback features while we add
  support for new CSS features.

- Handle HTTP error status codes like 404 or 500 and redirect them to the web
  archive.

- Redirect JS-only pages to alternatives that can be rendered in Dillo,
  similarly as the [libredirect plugin](https://libredirect.github.io/).

- Replace the current limited DPI mechanism for plugins.
