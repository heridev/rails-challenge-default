# Firstleaf Take Home Project

## Assignment
Today, we're going to design an application that will function as a user service.
This service supports new user creation and returning users that are currently
in the system. For the initial release, your task is to build two API endpoints:

`GET /api/users`

`POST /api/users`

### User Model

The user records must be stored in a database that supports SQL queries. Each
user has the following attributes which must conform to the rules described
below:

| Field Name   | Properties                                                |
| ------------ | --------------------------------------------------------- |
| id           | integer, primary key, not null, unique, auto-incrementing |
| email        | string, max 200 characters, not null, unique              |
| phone_number | string, max 20 characters, not null, unique               |
| full_name    | string, max 200 characters                                |
| password     | string, max 100 characters, not null                      |
| key          | string, max 100 characters, not null, unique              |
| account_key  | string, max 100 characters, unique                        |
| metadata     | string, max 2000 characters                               |

### GET /api/users

- [ ] Return all current user records, most recently created first.
- [ ] Optional `query` paramaters to filter results matching `email`, `full_name`,
    and `metadata`. Return in most recently created first order.
- [ ] 200 OK Response for all success cases
- [ ] 422 Unprocessable Entity for malformed query parameters.
- [ ] 5xx for server errors

### POST /api/users

- [ ] Create a new user record in the database.
- [ ] On success, return JSON object of user that was just created
- [ ] On success, return status code 201 Created
- [ ] On failure, return status code 422 Unprocessable Entity with a list of all
    the errors.
- [ ] 5xx for server errors.
- [ ] Endpoint can only accept `email`, `phone_number`, `full_name`, `password`,
    and `metadata` fields.
- [ ] `key` field should be generated server side when user is created
- [ ] `password` should be stored hashed with a salt value.
- [ ] `account_key` field should be generated from account key service.

### JSON Specifications

- [ ] On creation of a new user, the response object should be in the following
    format:
```
{
 email: "user@example.com",
 phone_number: "5551235555",
 full_name: "Joe Smith",
 key: "72ae25495a7981c40622d49f9a52e4f1565c90f048f59027bd9c8c8900d5c3d8",
 account_key: "b97df97988a3832f009e2f18663ac932",
 metadata: "male, age 32, unemployed, college-educated"
}
```
- [ ] On returning found users, the response object should be in the following
    format:
```
{
 users: [
   {
     email: "user@example.com",
     phone_number: "5551235555",
     full_name: "Joe Smith",
     key: "72ae25495a7981c40622d49f9a52e4f1565c90f048f59027bd9c8c8900d5c3d8",
     account_key: "b97df97988a3832f009e2f18663ac932",
     metadata: "male, age 32, unemployed, college-educated"
   }
 ]
}
```
- [ ] Errors should be returned as:
```
{
 errors: [
   "Phone number is too long",
   "Email is missing"
 ]
}
```

### Account Key Service

Account Keys are generated for a user by an external service. This service
expects the `email` and `key` fields to be POSTed, and in return the service
will respond with the appropriate `account_key` to be saved to the user.

The service is designed to be somewhat unreliable, so it is important to
interact with the service in a background process and then update the user
record when that background process is complete. If an error occurs, the
application should retry on some reasonable schedule.
**Example transaction:**
```
curl -H "Content-Type: application/json" -X POST https://w7nbdj3b3nsy3uycjqd7bmuplq0yejgw.lambda-url.us-east-2.on.aws/v1/account -d "{\"email\":\"user@example.com\",\"key\":\"72ae25495a7981c40622d49f9a52e4f1565c90f048f59027bd9c8c8900d5c3d8\"}"

{"email":"user@example.com","account_key":"b97df97988a3832f009e2f18663ac932"}
```

- [ ] Create Access Key service library
- [ ] On user create, trigger Sidekiq job for access Account Key service
- [ ] Perform retry on failure from Account Key service
- [ ] Update user model with `account_key` value

### Testing
#### User Model
- [ ] Verify that all defined columns necessary exist.
- [ ] Verify that columns have proper validation on the model.
- [ ] Verify that it is possible to search for a user by `email`, `full_name`,
    and `metadata` using a single search functionality.
- [ ] Coverage should be 100% for app/models/user.rb

#### User Service Routing
- [ ] Verify that the GET /api/users endpoint routes to the appropriate method.
- [ ] Verify that the POST /api/users endpoint routes to the appropriate method.

#### User Controller
- [ ] Verify that a request without a query parameter returns all users in the
    database using the specified JSON format, ordered by most recently created
    first.
- [ ] Verify that a request with a query parameter returns all users in the
    database filtered by the query paramater, using the specified JSON format,
    ordered by most recently created first.
- [ ] Verify that creating a new user works with unique values specified, and
    returns a single User JSON object and a 201 Created status header.
- [ ] Verify that creating a new user with non-unique values specified, returns
    a 422 Unprocessable Entity status, and an array of errors in the specified
    JSON format.
- [ ] Verify that a new user that is created has a random key generated for it on
    the server side.
- [ ] Verify that a new user that is created has it's password stored in a hashed
    manner, with a salt value.
- [ ] Verify that a new user that is created has an access_key created for it by
    accessing the Account Key service.


## Getting Started

### What's Provided
- Rails
- Postgresql

- Ensure that docker-compose is installed [Docker Compose](https://docs.docker.com/compose/install/#prerequisites)

- Start docker containers
  ```bash
  docker-compose build
  docker-compose up
  ```

- Setup test databases (use another bash console)
  ```bash
  docker-compose run web rails db:create
  docker-compose run web rails db:migrate
  ```

- Test site lives at `localhost:3005`

### Environment
- Initial Rails 5.2 API application
