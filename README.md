### Auth Service

Handles uses Auth0 to handle authentication and authorization, authorizes Lambda functions with JSON web token 

Overview Authentication Flow
- user gets authenticated to Auth0 
- Auth0 sends 200 ok with JSON web token
- sends request to API Gateway with Authorization header that has token 
- API gateway calls authorizer 
- autherizer verifies the token against the public key
- if successful then API Gateway will forward request to Lambda Function with JWT containing user details

```
functions:
  createAuction:
    handler: src/handlers/createAuction.handler
    events:
      - http:
          method: POST
          path: /auction
          cors: true
          authorizer: ***{Auth Service ARN here}*** <----
```

How does the the authenticaiton work?

```
const claims = jwt.verify(token, process.env.AUTH0_PUBLIC_KEY);
```

using the jsonwebtoken library to verify a JSON Web Token (JWT) with a public key

- The jsonwebtoken.verify() method is called with two arguments: the JWT token and the public key.
- The token argument is the JWT token that was previously generated by an authorized party and sent to the server.
- The process.env.AUTH0_PUBLIC_KEY argument is the public key that corresponds to the private key used to sign the JWT token.
- The jwt.verify() method decodes the token's header and payload and verifies its signature using the provided public key.
- If the token's signature is valid, the jwt.verify() method returns the decoded claims object, which contains the payload data encoded in the JWT.
- If the token's signature is invalid, the jwt.verify() method will throw an error indicating that the token is not authentic and has been tampered with.

## Features

- Test front-end application
- Private endpoint for testing
- Public endpoint for testing

## Getting started

### 1. Install dependencies

```sh
npm install
```

### 2. Create `secret.pem` file

This file will contain your Auth0 public certificate, used to verify tokens.

Create a `secret.pem` file in the root folder of this project. Simply paste your public certificate in there.

### 3. Deploy the stack

We need to deploy the stack in order to consume the private/public testing endpoints.

```sh
sls deploy -v
```

### 4. Final test

To make sure everything works, send a POST request (using curl, Postman etc.) to your private endpoint.

You can grab a test token from Auth0. Make sure to provide your token in the headers like so:

```
"Authorization": "Bearer YOUR_TOKEN"
```

You should be good to go!

<hr/>

## Bonus: Cross-stack authorization

This is very useful in a microservices setup. For example, you have an Auth Service (this service) which owns anything auth/user-related, and a bunch of other services that require user authorization.
Fear not, it is very easy to make your authorizer work anywhere else in your AWS account.

When defining your Lambdas in other services, simply define the `authorizer` as well and provide the ARN of your `auth` function (can be found in the AWS Console or via `sls info`).

#### Example:

```yaml
functions:
  someFunction:
    handler: src/handlers/someFunction.handler
    events:
      - http:
          method: POST
          path: /something
          authorizer: arn:aws:lambda:#{AWS::Region}:#{AWS::AccountId}:function:sls-auth-service-draft-dev-auth
```

If everything was set up correctly, all incoming requests to your `someFunction` Lambda will first be authorized. You can find the JWT claims at `event.requestContext.authorizer`.
