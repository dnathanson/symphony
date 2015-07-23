# Chat Server

## Design

* REST application using Spring Boot
* Supports horizonal scaling
* Multiple simultaneous sessions / clients supported per user
  * Each client will receive all messages
  * Clients do not have to be attached to same chat server
* Asynchrounous handling of new messages and new contacts for performance (using Redis pub/sub)
* Supports multiple push (or polling) technologies (TODO)
  * Currently only "push" implemenentation logs messages to console 
* Supports presence using Redis (TODO)
* Maven build
* Standalone JAR execution with embedded Tomcat
  * requires that Redis server is running on localhost with standard ports
* DAO layer abstract peristence (and Redis access)
  * Currently, only implementation is with in-memory maps
  * TODO: replace in-memory implementation with JDBC or NoSQL persistence

## REST API

Miniumum API implemented at this point.  For instance, there are no PUT methods available to modify records.

**NOTE: All API endpoint (except /login and /users) require query string arg `sessionId=<session ID>` where session ID is obtained
in available in the response from /login endpoint**

| Method | URL                                            | Notes                                                |
|--------|------------------------------------------------|------------------------------------------------------|
| GET    | `/login?emailAddress=<userEmailAddress>`       | Starts a session (no authentication)                 |
| GET    | `/logout`                                      | Ends a session                                       |
| GET    | `/ping`                                        | Keeps session alive. Updates presence                |
| POST   | `/users`                                       | Register a new user.  See below for POST body        |
| GET    | `/users/<email address>`                       | Gets user record by email address                    |
| GET    | `/contacts`                                    | Returns all contacts (with presence status)          |
| POST   | `/contacts/user/<contact user ID>`             | Add a new contact with specified user ID             |
| DELETE | `/contacts/user/<contact user ID>`             | Removes the contact with specified user ID           |
| GET    | `/messages?since=time`                         | Returns all message sent or received by current user |
| GET    | `/users/<contact user ID/messages`             | Returns all message between current user and contact with specified ID | 
| POST   | `/messages`                                    | Sends a message.  See below for POST body            |

#### Endpoint: POST `/users`
Body format

```json
{
    "emailAddress" : "dan@mycompany.com",
    "name" : "Dan N"
}
```
email address must be unique

#### Endpoint: POST `/messages`
Body format

```json
{
    "contents" : "foo bar",
    "receiverId" : 2
}
```
receiverId is user ID of one of current user's contacts

#### Endpoint: GET `/messages?since=time` 

Optional querty string argument `since` is standard date/time in milliseconds. If set, only messages sent since
the specified time will be returned.  This option is useful if a client goes offline for a while and wants to catch up
on any messages that it may have missed (assuming client gets messages via push).

## Shortcuts taken (or things not yet implemented)

* Login should be handled with regular HTTP form submission, not REST
* sessionId query string argument should be replaced with a secure HTTP cookie
* User presence in Redis is not implemented yet
* Implementation of various connection types supporting different push technologies not implemented yet
* There are no checks to ensure that users are online before sending messages
* There is no yet support for paging of message or contact lists which may be too long to handle in a single request
* There is no persistent storage for users, contacts or messages.  Once the server shuts down, data is lost.
