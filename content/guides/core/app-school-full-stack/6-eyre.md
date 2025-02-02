+++
title = "6. Eyre"
weight = 6
+++

Now that we have our structure file, agent, `$json` conversion library and mark
file, our back-end is complete. Before we start writing our front-end, though,
we should give a brief overview of how Eyre works.

[Eyre](/reference/arvo/eyre/eyre) is the HTTP server [vane](/reference/glossary/vane) of
Arvo. Eyre has a handful of different subsystems, but the main two are the
channel system and the scry interface. These two are what we'll focus on here.

In order to use the channel system or perform scries, a web client must have
authenticated with the ship's web login code (e.g.
`lidlut-tabwed-pillex-ridrup`) and obtained a session cookie. Our front-end will
be served directly from the ship by the `%docket` agent, so we can assume a
session cookie was already obtained when the user logged into landscape, and
skip over authentication.

## Channels

Eyre's channel system is the main way to interact with agents from a web client.
It provides a JSON interface to the ordinary poke and subscription system for
Gall agents.

First, a unique channel ID is generated by the web client (`@urbit/http-api`
uses the current Unix time concatenated with a random hex string). The client
then sends a poke or subscription request for the channel, and Eyre
automatically opens a new channel with that ID. Once open, the client can then
connect to the channel and receive any events such as poke acks, watch acks,
facts from subscriptions, etc.

The new channel is an SSE ([Server Sent
Event](https://html.spec.whatwg.org/#server-sent-events)) stream, and can be
handled by an `EventSource` object in Javascript. The `@urbit/http-api` library
we'll use abstracts this for us, so we won't need to deal with an `EventSource`
object directly. The channel can handle multiple concurrent subscriptions to
different agent subscription paths, and different agents can be poked through
the one channel. This means a client only needs to open a single channel for all
of its interactions with the ship. Each subscription is given a different ID, so
they can be individually unsubscribed later.

A channel will timeout after 12 hours of inactivity, and the timeout is reset
every time Eyre receives a message of any kind from the client. Additionally,
each subscription on the channel may only accumulate 50 unacknowledged facts
before it's considered "clogged", in which case the individual clogged
subscription will be closed by Eyre after a short delay. All events of any kind
which Eyre sends out on the channel must be ack'd by the client. Ack'ing one
event will also ack all previous events too. The `@urbit/http-api` library we'll
use automatically acks events for us, so we don't need to worry about clogged
subscriptions or manually ack'ing events.

Eyre expects a particular JSON object structure for each of these different
requests, but the `@urbit/http-api` library we'll use includes functions to send
pokes, subscription requests, etc, so we won't need to manually construct the
JSON objects in our front-end.

## Scries

Eyre's scry interface is separate to the channel system. Scries are performed by
a simple GET request to a path with a format of `/~/scry/{agent}{path}.{mark}`.
If successful, the HTTP response will contain the result with the mark
specified. If unsuccessful, an HTTP error will be thrown in response.

The `@urbit/http-api` library we'll use includes a function for performing
scries, so we'll not need to manually send GET requests to the ship.

## Resources

- [The Eyre vane documentation](/reference/arvo/eyre/eyre) - This section of the vane
  docs covers all aspects of Eyre.
- [Eyre External API Reference](/reference/arvo/eyre/external-api-ref) - This section
  of the Eyre documentation contains reference material for Eyre's external API.

- [The Eyre Guide](/reference/arvo/eyre/guide) - This section of the Eyre
  documentation walks through using Eyre's external API at a low level (using
  `curl`).
