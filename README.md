
## Setup

You have to downlaod [stack](https://docs.haskellstack.org/en/stable/README/) if you don't have it
 - Clone this repository
 - From comman line run `stack runghc Main.hs`. Note you have to be in the `ci285-calculator` directory
 - If everything was successfull, you should see `application launched`
 - Open your browser and navigate to `http://localhost:3000`
 - Well done!



## Screenshots

### Login screen
![Imgur](http://i.imgur.com/GGRxSI0.png)

### Signup screen
![Imgur](http://i.imgur.com/q5PiNUL.png)

### User home screen
![Imgur](http://i.imgur.com/xBTUDVH.png)


## Users API

API for dealing with users. That is creating new users, updating details, deleting as well as authenticating.

### _POST /users_

Rgisters new user. The POST method is an obvious choice, because no other method would be suitable for this task. For example, we can't use PUT, because it is used to create or replace a resource if it is already exists, which would result of replacing existing users with new ones.
Example:

```json
{
  "username": "jhonDoe",
  "password": "23#483uA"
}
```
**NOTE** This is not very secure way of doing.

Possible responses:

- 201 - if user have been created successfully.

`Location: http://localhost:3000/users/johnDoe`

Note, that response includes `Location:` in the header which indicates where to find a new resource. But to access it, client must to authenticate itself first.

- 409 - if user with such username already exists.
- 400 - if JSON is not properly formatted
- 404 - otherwise

### _GET /users/:username_

Request for users home page. This resource requires user to authenticate itself using `Basic authentication`, before he can access it. Note, that I did not use **/id** instead of **/username**, because an **/id** is less descriptive and less memorable.

Example 

```
GET /users/username HTTP/1.1
Authorization: Basic wpoauidhfiopuh=
```

Possible responses:

- 400 - if authorization header is not formatted properly
- 401 - if username and password doesn't match OR request doesn't have `Authorization` header. Also it includes `WWW-Authenticate: Basic realm="users"` header.
- 404 - if user doesn't exists.

## Calculator API

API for performing 4 basic operations of calculator: addition, subtraction, multiplication, division. Each operation is performed 
in decimal system and requires only two operands. In addition, all URIs has the following general strucutre:

### _GET /:operations/:operand/:operand_

- **GET** - request method. `GET` primary is used to retrieve information of the resource, but in this case it doesn't exist yet, because any requested calculation will be done after request. But in the future we me implement a cache where most common calculations will be stored. In that case `GET` would make perfect sense.

- **/:operations** - corresponds to addition, subtraction, multiplication and division. At the moment, API supports only those four operations, thus there are only four paths **/additions**, **/subtractions**, **/multiplications** and **/divisions**. Note, that in URI a noun is used instead of verb. While, a verb like _add_ would make better sense, it wouldn't comply with RESTful standards, where URI identifies a resource. "What kind of resource is _/add_ ?" _/additions_, on the other hand, sounds much better. And, like I sad, we may actually have a database or a table, or a map with an actual additions.

- **/:operand/:operand** - represents two integers for an operation. At the moment, all operations are bi-operations, thus they epect two operands. Each operand must consist only of the sequence of digits which can be precede by one of `-` or `+` symbols. 

In normal situations, the system will respond with status code 200 and JSON content. For example, 
a response for GET request with an URI of

**/additions/-4898/7458** will have content of

  ```json
  {"result": "2560"}
  ```
In exceptional situations, the system will respond with an error code and an error page content.

- 404 - not found. This can happen if 
  - **/:operations** has not been specified or have been mispelled. 
  - **/:operand** will have at least one non-digit character. E.g, **GET /additions/48,98/7458** 
  - **/:operations** has less than or more than two operands. E.g, **GET /additions/4898/**.
  

## Calculations history API

API for saving and retrieving calculations.

### _PUT /:operations/:operand/:operand:/:username_

Saves users calculation in the database. Note, that only authorised users can access this resource.

Possible responses:

- 201 - if calculation was saved successfuly. This response doesn't have `Location:` header, instead it returns an JSON answer in the body, as in calculator API.
- 301 - if user is not have been authenticated or session expired
- 404 - if url was wrong
- 400 - badly formatter request.
