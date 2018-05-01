# Solid server: HTTP request handling flow

When an agent sends an HTTP request,
the Solid server should take the following steps.

## Step 1: Determine whether the request is directed to the personal data store
Based on the URL path, the server distinguishes between two kinds of incoming requests:
1. **Personal data store requests**, which can follow arbitrary URL patterns
2. **Account and server management requests**, which follow predefined URL patterns such as `/.well-known/`, `/.auth/`, etc.

Requests of the second kind are sent to specifically wired handlers,
which typically only accept `GET`, `HEAD`, `OPTIONS`, and `POST`.
<br>
Personal data store requests follow the steps below.

## Step 2: Parse the request to the personal data store
The HTTP request's URL, headers, and optional body and/or client certificate
are parsed into a structure consisting of the following components:

- The **target** identifies the resource that is the subject of the request.
  It corresponds to the full request URL
  (taking into account protocol and host)
  after normalization (removal of dot segments)
  and a sanity check.

- The **body** is a parsed version of the request body, which can be empty.
  The server has access to a list of parsers per content type,
  such as `text/turtle`, `application/sparql-update`.
  Parsing should be lazy,
  such that the contents are only created when their parsed form is needed
  (which might not be the case, for instance, with `PUT` requests).
  If no parser for the given content type is available,
  the object is considered a textual blob (if a charset was given),
  or a binary blob.

- The **operation** consists of the HTTP method, and a set of flags.
  Valid operations are
  `GET`, `HEAD`, `OPTIONS`, `POST`, `PUT`, `PATCH`, `DELETE`.
  Based on this method—and, in the case of `PATCH`, the parsed body—the
  _read_, _delete_, and/or _append_ flags are assigned.
  They allows us to determine the required permissions in the next step
  (which is why we are parsing the body already at this stage).

- The **authentication** is a string
  indicating the WebID of the logged-in agent,
  or `null` if no agent is logged in.
  To determine this value from the request headers and/or client certificate,
  the server has access to a list of authenticators,
  such as WebID-TLS (which reads the WebID from a client certificate)
  and OIDC (which reads the WebID based on a cookie).

- The **preferences** are a key/value object of categorized lists
  that indicate the agent's representation preferences,
  parsed from the HTTP headers.
  They include media type and language preferences,
  and are parsed along with their `q` values.

Minor parsing failures at this stage
are automatically corrected through the insertion of defaults.
For example, if preferences are missing or invalid,
the server could assume `text/turtle` and `en`;
and missing authentication results in the `null` agent.

Major parsing failures result in a `400` response
with a detail of the cause.
Examples include invalid HTTP methods,
and syntactically incorrect, unparseable, or missing `PATCH` bodies.
When possible,
the body of this response should respect
the agent's representation preferences.