# Solid server: HTTP request handling flow

When an HTTP request arrives, the Solid server should take the following steps.

## Step 1: Determine whether the request is directed to the personal data store
Based on the URL path, the server distinguishes between two kinds of incoming requests:
1. **Personal data store requests**, which can follow arbitrary URL patterns
2. **Account and server management requests**, which follow predefined URL patterns such as `/.well-known/`, `/.auth/`, etc.

Requests of the second kind are sent to specifically wired handlers.
<br>
Personal data store requests follow the steps below.
