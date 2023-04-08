# @passkey/nodejs

Passkeys allow users to sign in to your app without typing a password. It's an alternative method of user authentication that eliminates the need for a password while being easier to use and far more secure. Learn more about passkeys: [Google](https://developers.google.com/identity/passkeys), [Apple](https://developer.apple.com/passkeys/).

@passkey/nodejs is a Node.js module that provides an easy-to-use server-side implementation of the Fido library by wrapping its basic functionalities. It is designed to work seamlessly with the @passkey/native and/or @passkey/react packages, enabling developers to easily integrate secure passkey management into their server-side applications.
## Installation
```ws
npm install @passkey/nodejs
```

## Overview
@passkey/nodejs provides methods for setting options, performing an attestation (signing up a new user), and performing an assertion (signing in an existing user). Attestation and assertion involve generating and verifying cryptographic signatures, and @passkey/nodejs wraps the underlying cryptographic operations for ease of use.

@passkey/nodejs exposes the following methods:
- To set the options:
  - `setPasskeyOptions`: Set the options for the passkey operations.
- To perform an attestation (sign-up a new user):
  - `getChallengeForAttestation`: Get a challenge to perform an attestation.
  - `checkAttestationResult`: verifying the correctness of the attestation result you received.
- To perform an assertion (sign-in an user):
  - `getChallengeForAssertion`: Get a challenge to perform an assertion.
  - `checkAssertionResult`: verifying the correctness of the assertion result you received.

## Terminology
@passkey/nodejs uses the following terms:
- `Passkey:` A cryptographically secure identifier that allows users to sign in to an app without typing a password.
- `Attestation:` The process of generating a cryptographic signature that links a passkey to a user's device.
- `Assertion:` The process of generating a cryptographic signature that proves a user's ownership of a passkey.

## Usage
It's a good practice to set the passkey options when your server starts up, so you don't have to set them every time you use the library.
Here's an example of how to set the passkey options:

```ts
// your_server_app.js
const { setPasskeyOptions } = require('@passkey/nodejs');

// Set the passkey options
setPasskeyOptions({
  timeout: 60000,                             // 60 seconds 
  rpId   : 'example.com',                     // your domain (the one associated with your app)   
  rpName : 'Example',                         // your domain name
  rpIcon : 'https://example.com/icon.png',    // your icon
});
````

Here's an example of how to use @passkey/nodejs to sign up a new user:

```ts
// When you receive a challenge request to sign up a new user from your client app
// => you have to perform an attestation
const {
    getChallengeForAttestation,
    checkAttestationResult,
} = require('@passkey/nodejs');

const processClientRequestChallengeToSignUp = async () => {
  try {
    // you need a userId (you can also see it as a passkeyId)
    const userId = 'some_user_id'; // you can generate a random userId for example
    
    // Get an attestation
    const attestation = await getChallengeForAttestation(userId);
    
    const {challenge} = attestation;
    // Save the challenge and the userId (on the server side, in the user session for example)
    
    // Send the attestation to your client app
    return attestation;
  } catch (error) {
    // Handle error
  }
};

// When you receive the attestationResult back from your client app
const processClientSentAttestationResult = async (attestationResult) => {
  try {
    // you need to retrieve the challenge and the userId from the user session or other storage you saved them in
    const challenge = 'challenge retrived from the user session or other storage';
    const userId = 'userId retrived from the user session or other storage';
    
    // Verify the correctness of the attestationResult you received
    const {publicKeyPem} = await checkAttestationResult({attestationResult, challenge}); // will throw an error if the attestation is not valid
    // Save the publicKeyPem for this userId in your database for example
    
    // You can now create the new user account and login the user into your app
    const token = 'some_token_to_login_this_user';
    return token;
  } catch (error) {
    // Handle error
  }
};
```

And here's an example of how to use @passkey/nodejs to sign in an existing user:
```ts
const {
  getChallengeForAssertion,
  checkAssertionResult,
} = require('@passkey/nodejs');

// When you receive a challenge request to sign in a user from your client app
// => You have to perform an assertion
const processClientRequestChallengeToSignIn = async () => {
  try {
    // Get a challenge to perform an assertion
    const assertion = await getChallengeForAssertion();
    const {challenge} = assertion;
    // Save this challenge (on the server side, in the user session for example)
    // Send the assertion to your client app
    return assertion;
  } catch (error) {
    // Handle error
  }
};

// when you receive the assertionResult back from your client app
const processClientSentAssertionResult = async (assertionResult) => {
  try {
    // retrieve the challenge from the user session or other storage you saved them in
    const challenge = 'challenge retrived from the user session or other storage';

    // verify the correctness of the assertionResult you received
    // you need to extract the userId (it can also represent the passkeyId) from the assertionResult
    const userId = await getAssertionUserId(assertionResult);
    // using this userId to retrieve the publicKeyPem from your database (or other storage you saved it in)
    const publicKeyPem =  'publicKeyPem retrieved from your database';
    
    // then verify the assertionResult with this publicKeyPem
    const isAssertionValid = await checkAssertionResult({attestationResult}); // will throw an error if the assertion is not valid
    // you can now login this user (for example by creating a token for this user and sending it to the client app) 
    const token = 'some_token_to_login_the_user';
    return token;
  } catch (error) {
    // handle error
  }
};
````


### More details
@passkey/nodejs exposes the following methods:
- `setPasskeyOptions:`

  The setPasskeyOptions object has the following properties:<br>
  - `rpId`: string,<br>
  The relying party identifier.
  - `rpName`: string,<br>
  The relying party name.
  - `timeout?`: number (default value: 60000, 60 seconds),<br>
  The timeout for the operation in milliseconds.
  - `rpIcon?`: string,<br>
  The relying party icon.
  - `challengeSize?`: number (default value: 128),<br>
  The challenge size in bytes.
  - `attestation?`: string (default value: 'direct'),<br>
  The attestation type.
  - `cryptoParams?`: number[] (default value: [-7, -257]),<br>
  The crypto parameters.
  - `authenticatorAttachment?`: string (default value: 'platform'),<br>
  The authenticator attachment.
  - `authenticatorRequireResidentKey?`: boolean (default value: false),<br>
  If true, the authenticator will require a resident key.
  - `authenticatorUserVerification?`: string (default value: 'preferred'),<br>
  The authenticator user verification.

- `getChallengeForAttestation`
- `checkAttestationResult`
- `getChallengeForAssertion`
- `checkAssertionResult`

## Contributing

See the [contributing guide](CONTRIBUTING.md) to learn how to contribute to the repository and the development workflow.

## License

MIT

---

Made with [create-react-native-library](https://github.com/callstack/react-native-builder-bob)

