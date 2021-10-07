<h1 style="text-align: center; vertical-align: middle">
  <center><a href="https://www.gimly.io/"><img src="https://avatars.githubusercontent.com/u/64525639?s=200&v=4" alt="Gimly" width="120" style="vertical-align: middle"></a> &nbsp;and &nbsp; <a href="https://www.sphereon.com"><img src="https://sphereon.com/content/themes/sphereon/assets/img/logo.svg" alt="Sphereon" width="320" style="vertical-align: middle" ></a></center>
ONTOCHAIN Demo
</h1>
<br>


## Summary

This demo shows an authentication flow using the [DIDSIOP library](#did-auth-siop). The underlying technology allows a user to authenticate to a provider by scanning a QR code that is presented on by the relying party (RP). For this demo project, the [Demo WEB](#onto-web-demo) app is the relying party (RP) and the [Demo RN](#demo-rn) is the OpenID provider (OP). The flow starts with the RP where the user clicks a login button on the web page, after which a QR code is generated. The user scans the QR code using the Demo RN app, which in turn requests access to the protected resources hosted by the RP. Since the user is not logged in, an authentication request will be sent back to the OP. If the request is valid, the OP sends an authentication response back to the RP, which verifies the validity of response. If all steps succeed, access to the protected resources is granted to the user. The user will now be redirected on the web app to view the protected resource. [GimlyID-QR-Code](#gimlyid-qr-code) and [GimlyID-QR-Code-Scanner](#gimlyid-qr-code-scanner) are employed to respectively generate and read the QR code used in the authentication process.


## Authentication Flow

![ONTO Demo Flow](https://lucid.app/publicSegments/view/02cb0788-5fe6-4fcd-91f6-3f82d6ca1402/image.png)


## ONTO web demo

This is the web application (RP) that generates the QR code and starts the authentication process;

###Setup

#### Build & start

clone the repository:

```bash
git clone git@github.com:Sphereon/onto-web-demo.git
cd onto-demo-server
```

#### Configure environment

In the `onto-demo-server` folder, create a file called `.env.local` and populate it using `.env` as example.  A valid config will look like this:

```dotenv
NODE_ENV=development
PORT=5001
COOKIE_SIGNING_KEY=8E5er6YyAO6dIrDTm7BXYWsafBSLxzjb
REDIRECT_URL_BASE=http://192.168.1.200:5001/ext
RP_DID=did:ethr:0xe1dB95357A4258b33A994Fa8cBA5FdC6bd70011D
RP_PRIVATE_HEX_KEY=850e54b92c6291a1ff7b8c3ef30e032571ed77c9e5c78b1cd6ee5fec4fea984f
AUTH_REQUEST_EXPIRES_AFTER_SEC=300
MOCK_AUTH_RESPONSE=false
```

(Exception for the IP address this is a valid configuration to test with.)

From the root directory

```bash
cd onto-web-demo
yarn global add concurrently
yarn global add ts-node
yarn install-all
yarn build-types
yarn start
```

The server will start on port `5001`, the client will start & open a browser on `http://localhost:3000/`

#### Docker

From the root folder run:

```bash
docker build -t onto-web-demo .
docker run -it -p 5001:5001 -p 3000:3000 onto-web-demo
```

### Docker compose

From the root folder run:

```bash
docker-compose up
```

Reference: [Onto-Web-Demo](https://github.com/Sphereon/onto-web-demo)


## Demo RN

This is the React-Native application (OP) that stores the self issued credentials

###Setup

### Requirements

- The Android device must have a fingerprint reader
- Make sure you have the Android platform SDK installed on your computer, your mobile is plugged in and in debug mode.
- Execute `adb devices` and confirm your device is listed

### To build

clone the repository:

```bash
git clone git@github.com:Sphereon/rn-did-siop-example-app.git
```

From the root directory

Note: Sphereon-Opensource/rn-did-siop-auth-lib.git may need to be removed from dependencies

```bash
cd rn-did-siop-example-app
yarn global add react-native-cli
yarn install
yarn nodeify
yarn android
```

Reference: [Demo-RN](https://github.com/Sphereon/rn-did-siop-example-app)


## GimlyID QR Code

This is the QR code generator, it works with `React` and `React-Native`. It generates a QR code containing:

| prop                    | type                         | default value | description                                                                                                                                                                                              |
| ----------------------- | ---------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`                  | `QRType`                     |               | This is the type stored within the QR code. Options: [AUTHENTICATION].                                                                                                                                   |
| `did`                   | `string`                     |               | This is the DID that the relying party (the party that integrated the authentication support on their website) will use to sign/encrypt data towards the client. This will be stored within the QR code. |
| `mode`                  | `QRMode`                     |               | This is the mode stored within the QR code. Options: [DID_AUTH_SIOP_V2].                                                                                                                                 |
| `redirectUrl`           | `string`                     |               | This is a redirect url that will be stored within the QR code.
| `state`                 | `string`                     |               | This is a string representing a short UUID

Reference: [GimlyID-QR-Code](https://github.com/Sphereon-Opensource/gimlyid-qr-code)


## GimlyID QR Code Scanner

A GimlyID QR code scanner component for `React-Native`. This scanner can scan QR codes generated by the gimlyid-qr-code component.


## Api

| prop                 | type                         | default value | description                                                                                |
| -------------------- | ---------------------------- | ------------- | ------------------------------------------------------------------------------------------ |
| `style`              | `React.CSSProperties`        |               | Sets the styling of the qr code scanner.                                                   |
| `onRead`             | `Function`                   |               | The onRead will be called when the QR code is read and will have access to the QR content. |

Reference: [GimlyID-QR-Code-Scanner](https://github.com/Sphereon-Opensource/gimlyid-qr-code-scanner):


## DID Auth Siop

SIOP v2 (REFRENCE) is an extension of OpenID Connect (REFRENCE) to allow End-users to act as OpenID Providers (OPs) themselves. Using Self-Issued OPs, End-users can authenticate themselves and present claims directly to the Relying Parties (RPs), typically a webapp, without relying on a third-party Identity Provider. This makes the solution fully self sovereign, as it does not rely on any third parties and strictly happens peer 2 peer, but still uses the OpenID Connect protocol. Next to the user acting as an OpenID Provider, this library also includes support for Verifiable Presentations (REFRENCE) using the Presentation Exchange (REFRENCE to DIF) support provided by Sphereon's pe-js library (REFRENCE). This means that the Relying Party can pose submission requirements on the Verifiable Credentials it would like to receive from the client/OP. The OP then checks whether it has the credentials to support the submission requirements. Only if that is the case it will send the relevant (parts of the) credentials as a Verifiable Presentation in the Authentication Response destined for the Webapp/Relying Party. The relying party in turn checks validity of the Verifiable Presentation(s) as well as the match with the submission requirements. Only if everything is verified successfully the RP serves the protected page(s). This means that the authentication can be extended with claims about the authenticating entity, but it can also be used to easily consume credentials from supporting applications, without having to setup DIDComm (REFRENCE) connections for instance. The term Self-Issued comes from the fact that the End-users (OP) issue self-signed ID Tokens to prove validity of the identifiers and claims. This is a trust model different from that of the rest of OpenID Connect where OP is run by the third party who issues ID Tokens on behalf of the End-user to the Relying Party upon the End-user's consent. This means the End-User is in control about his/her data instead of the 3rd party OP.


# Steps involved

Flow diagram:

![Flow diagram](https://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/Sphereon-Opensource/did-auth-siop/develop/docs/auth-flow.puml)

1. Client (OP) initiates an Auth request by POST-ing to an endpoint, like for instance `/did-siop/v1/authentications` or clicking a Login button and scanning a QR code
2. Web (RP) receives the request and access the RP object which creates the authentication request as JWT, signs it and returns the response as an OpenID Connect URI
    1. JWT example:
       JWT Header
       
       ```json
        {
          "alg": "ES256K",
          "kid": "did:ethr:0xcBe71d18b5F1259faA9fEE8f9a5FAbe2372BE8c9#controller",
          "typ": "JWT"
        }
        ```

        JWT Payload

        ```json
        {
          "iat": 1632336634,
          "exp": 1632337234,
          "response_type": "id_token",
          "scope": "openid",
          "client_id": "did:ethr:0xcBe71d18b5F1259faA9fEE8f9a5FAbe2372BE8c9",
          "redirect_uri": "https://acme.com/siop/v1/sessions",
          "iss": "did:ethr:0xcBe71d18b5F1259faA9fEE8f9a5FAbe2372BE8c9",
          "response_mode": "post",
          "nonce": "qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg",
          "state": "b32f0087fc9816eb813fd11f",
          "registration": {
             "did_methods_supported": [
              "did:ethr:",
              "did:eosio:"
             ],
             "subject_identifiers_supported": "did"
          }
        }
        ```

    2. The Signed JWT, called the JWS follows the following scheme (JWS Compact Serialization, https://datatracker.ietf.org/doc/html/rfc7515#section-7.1):

   `BASE64URL(UTF8(JWT Protected Header)) || '.' ||
   BASE64URL(JWT Payload) || '.' ||
   BASE64URL(JWS Signature)`

    3. Create the URI containing the JWS:
    
    ```
    openid://?response_type=id_token 
      &scope=openid
      &client_id=did%3Aethr%3A0xBC9484414c1DcA4Aa85BadBBd8a36E3973934444
      &redirect_uri=https%3A%2F%2Frp.acme.com%2Fsiop%2Fjwts
      &iss=did%3Aethr%3A0xBC9484414c1DcA4Aa85BadBBd8a36E3973934444
      &response_mode=post
      &nonce=qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg&state=b32f0087fc9816eb813fd11f
      &registration=%5Bobject%20Object%5D
      &request=<JWS here>
    ```
   
3. Web receives the Auth Request URI Object from RP
4. Web sends the Auth Request URI in the response body to the client
5. Client accesses OP object to create an Authentication response
6. OP verifies the authentication request, including checks on whether the RP did and keytypes are supported, next to whether the OP can satisfy the RPs requested Verifiable Credentials
7. OP creates the authentication response object as follows:
    1. Create an ID token as shown below:

    JWT encoded ID Token
    JWT Header
   
    ```json
    {
      "alg": "ES256K",
      "kid": "did:ethr:0x998D43DA5d9d78500898346baf2d9B1E39Eb0Dda#keys-1",
      "typ": "JWT"
    }
    ```
   
    JWT Payload
   
    ```json
    {
      "iat": 1632343857.084,
      "exp": 1632344857.084,
      "iss": "https://self-issued.me/v2",
      "sub": "did:ethr:0x998D43DA5d9d78500898346baf2d9B1E39Eb0Dda",
      "aud": "https://acme.com/siop/v1/sessions",
      "did": "did:ethr:0x998D43DA5d9d78500898346baf2d9B1E39Eb0Dda",
      "sub_type": "did",
      "sub_jwk": {
         "kid": "did:ethr:0x998D43DA5d9d78500898346baf2d9B1E39Eb0Dda#key-1",
         "kty": "EC",
         "crv": "secp256k1",
         "x": "a4IvJILPHe3ddGPi9qvAyXY9qMTEHvQw5DpQYOJVA0c",
         "y": "IKOy0JfBF8FOlsOJaC41xiKuGc2-_iqTI01jWHYIyJU"
      },
      "nonce": "qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg",
      "state": "b32f0087fc9816eb813fd11f",
      "registration": {
         "issuer": "https://self-issued.me/v2",
         "response_types_supported": "id_token",
         "authorization_endpoint": "openid:",
         "scopes_supported": "openid",
         "id_token_signing_alg_values_supported": [
            "ES256K",
            "EdDSA"
         ],
         "request_object_signing_alg_values_supported": [
            "ES256K",
            "EdDSA"
         ],
         "subject_types_supported": "pairwise"
      }
    }
    ```

    2. Sign the ID token using the DID key (kid) using JWS scheme (JWS Compact Serialization, https://datatracker.ietf.org/doc/html/rfc7515#section-7.1) and send it to the RP:

   `BASE64URL(UTF8(JWS Protected Header)) || '.' ||
   BASE64URL(JWS Payload) || '.' ||
   BASE64URL(JWS Signature)`

8. OP returns the Auth response and jwt object to the client
9. Client does an HTTP POST to `redirect_uri` from the request (and the aud in the response): `https://acme.com/siop/v1/sessions` using `application/x-www-form-urlencoded`
10. Web receives the ID token (auth response) and uses the RP's object verify method
11. RP performs the validation of the token, including signature validation, expiration and Verifiable Presentations if any. It returns the Verified Auth Response to WEB
12. WEB returns a `200 OK` response to Client with a redirect to another page (logged in or confirmation of VP receipt etc).
13. From that moment on Client can use the Auth Response as bearer token as long as it is valid

### Relying Party (RP)

```typescript
// The relying party (web) private key and DID and DID key (public key)
const rpKeys = {
    hexPrivateKey: 'a1458fac9ea502099f40be363ad3144d6d509aa5aa3d17158a9e6c3b67eb0397',
    did: 'did:ethr:ropsten:0x028360fb95417724cb7dd2ff217b15d6f17fc45e0ffc1b3dce6c2b8dd1e704fa98',
    didKey: 'did:ethr:ropsten:0x028360fb95417724cb7dd2ff217b15d6f17fc45e0ffc1b3dce6c2b8dd1e704fa98#controller'
}
const rp = RP.builder()
    .redirect(EXAMPLE_REDIRECT_URL)
    .requestBy(PassBy.VALUE)
    .internalSignature(rpKeys.hexPrivateKey, rpKeys.did, rpKeys.didKey)
    .addDidMethod("ethr")
    .registrationBy(PassBy.VALUE)
    .build();
```

### OpenID Provider (SIOP)

```typescript
// The OpenID Provider (client) private key and DID and DID key (public key)
const opKeys = {
    hexPrivateKey: '88a62d50de38dc22f5b4e7cc80d68a0f421ea489dda0e3bd5c165f08ce46e666',
    did: 'did:ethr:ropsten:0x03f8b96c88063da2b7f5cc90513560a7ec38b92616fff9c95ae95f46cc692a7c75',
    didKey: 'did:ethr:ropsten:0x03f8b96c88063da2b7f5cc90513560a7ec38b92616fff9c95ae95f46cc692a7c75#controller'
}

const op = OP.builder()
    .withExpiresIn(6000)
    .addDidMethod("ethr")
    .internalSignature(opKeys.hexPrivateKey, opKeys.did, opKeys.didKey)
    .registrationBy(PassBy.VALUE)
    .build();
```

### RP creates the Authentication Request

```typescript
const reqURI = await rp.createAuthenticationRequest({
    nonce: "qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg",
    state: "b32f0087fc9816eb813fd11f"
});

console.log(`nonce: ${reqURI.nonce}, state: ${reqURI.state}`);
// nonce: qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg, state: b32f0087fc9816eb813fd11f


console.log(reqURI.encodedUri)
// openid://?response_type=id_token&scope=openid&client_id=did.......&jwt=ey..........
```

### OP creates the Authentication Response using the Request

```typescript
// The create method also calls the verify and parse methods, so no need to do it manually
const authRespWithJWT = await op.createAuthenticationResponse(requestURI.jwt);

// The below call would be equivilent if the optional steps above would have been used
//const authRespWithJWT = await op.createAuthenticationResponseFromVerifiedRequest(verifiedReq);
```

### RP verifies the Authentication Response

```typescript
const verifiedAuthResponseWithJWT = await rp.verifyAuthenticationResponseJwt(authRespWithJWT.jwt, {
    audience: EXAMPLE_REDIRECT_URL,
})

expect(verifiedAuthResponseWithJWT.jwt).toBeDefined();
expect(verifiedAuthResponseWithJWT.payload.state).toMatch("b32f0087fc9816eb813fd11f");
expect(verifiedAuthResponseWithJWT.payload.nonce).toMatch("qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg");
```

## DID resolution

### Description

Resolves the DID to a DID document using the DID method provided in didUrl and using DIFs [did-resolver](https://github.com/decentralized-identity/did-resolver) and Sphereons [Universal registrar and resolver client](https://github.com/Sphereon-Opensource/did-uni-client).
This process allows retrieving public keys and verificationMethod material, as well as services provided by a DID controller. Can be used in both the webapp and mobile applications. Uses the did-uni-client, but could use other DIF did-resolver drivers as well. The benefit of the uni client is that it can resolve many DID methods. Since the resolution itself is provided by the mentioned external dependencies above, we suffice with a usage example.

#### Usage

```typescript
import { Resolver } from 'did-resolver'
import { getResolver as getUniResolver } from '@sphereon/did-uni-client'

const resolver = new Resolver(getUniResolver('ethr'));

resolver.resolve('did:ethr:0x998D43DA5d9d78500898346baf2d9B1E39Eb0Dda').then(doc => console.log)
```


## JWT and DID creation and verification

Please note that this chapter is about low level JWT functions, which normally aren't used by end users of this library. Typically, you use the AuthenticationRequest and Response classes (low-level) or the OP and RP classes (high-level).

### Create JWT

Creates a signed JWT given a DID which becomes the issuer, a signer function, and a payload over which the signature is created.

#### Usage

```typescript
const signer = ES256KSigner(process.env.PRIVATE_KEY);
createDidJWT({requested: ['name', 'phone']}, {issuer: 'did:eosio:example', signer}).then(jwt => console.log)
```

### Verify JWT

Verifies the given JWT. If the JWT is valid, the promise returns an object including the JWT, the payload of the JWT, and the DID Document of the issuer of the JWT, using the resolver mentioned earlier. The checks performed include, general JWT decoding, DID resolution, Proof purposes
 allows restriction of verification methods to the ones specifically listed, otherwise the 'authentication' verification method of the resolved DID document will be used

#### Usage

```typescript
verifyDidJWT(jwt, resolver, {audience: '6B2bRWU3F7j3REx3vkJ..'}).then(verifiedJWT => {
       const did = verifiedJWT.issuer;                          // DID of signer
       const payload = verifiedJWT.payload;                     // The JHT payload
       const doc = verifiedJWT.didResolutionResult.didDocument; // DID Document of signer
       const jwt = verifiedJWT.jwt;                             // JWS in string format 
       const signerKeyId = verifiedJWT.signer.id;               // ID of key in DID document that signed JWT
       // ... etc.
   });
```


## AuthenticationRequest class

### createURI

Create a signed URL encoded URI with a signed SIOP Authentication request

#### Usage

```typescript
const EXAMPLE_REDIRECT_URL = "https://acme.com/hello";
const EXAMPLE_REFERENCE_URL = "https://rp.acme.com/siop/jwts";
const HEX_KEY = "f857544a9d1097e242ff0b287a7e6e90f19cf973efe2317f2a4678739664420f";
const DID = "did:ethr:0x0106a2e985b1E1De9B5ddb4aF6dC9e928F4e99D0";
const KID = "did:ethr:0x0106a2e985b1E1De9B5ddb4aF6dC9e928F4e99D0#keys-1";

const opts = {
   redirectUri: EXAMPLE_REDIRECT_URL,
   requestBy: {
      type: SIOP.PassBy.VALUE,
   },
   signatureType: {
      hexPrivateKey: HEX_KEY,
      did: DID,
      kid: KID,
   },
   registration: {
      didMethodsSupported: ['did:ethr:'],
      subjectIdentifiersSupported: SubjectIdentifierType.DID,
      registrationBy: {
         type: SIOP.PassBy.VALUE,
      },
   }
};

AuthenticationRequest.createURI(opts)
    .then(uri => console.log(uri.encodedUri));
```

#### Output:

```
// openid://
//      ?response_type=id_token
//      &scope=openid
//      &client_id=did%3Aethr%3A0x0106a2e985b1E1De9B5ddb4aF6dC9e928F4e99D0
//      &redirect_uri=https%3A%2F%2Facme.com%2Fhello
//      &iss=did%3Aethr%3A0x0106a2e985b1E1De9B5ddb4aF6dC9e928F4e99D0
//      &response_mode=post
//      &response_context=rp
//      &nonce=aTO_jvEBPyigHFIueD1cT657LxVZwWxBesd2v6LVnjA
//      &state=b34b64db619e798b317fd4c0
//      &registration=%5Bobject%20Object%5D
//      &request=eyJhbGciOiJFUzI1NksiLCJraWQiOiJkaWQ6ZXRocjoweDAxMDZhMmU5ODViMUUxRGU5QjVkZGI0YUY2ZEM5ZTkyOEY0ZTk5RDAja2V5cy0xIiwidHlwIjoiSldUIn0.eyJpYXQiOjE2MzIzNTAxNDYsImV4cCI6MTYzMjM1MDc0NiwicmVzcG9uc2VfdHlwZSI6ImlkX3Rva2VuIiwic2NvcGUiOiJvcGVuaWQiLCJjbGllbnRfaWQiOiJkaWQ6ZXRocjoweDAxMDZhMmU5ODViMUUxRGU5QjVkZGI0YUY2ZEM5ZTkyOEY0ZTk5RDAiLCJyZWRpcmVjdF91cmkiOiJodHRwczovL2FjbWUuY29tL2hlbGxvIiwiaXNzIjoiZGlkOmV0aHI6MHgwMTA2YTJlOTg1YjFFMURlOUI1ZGRiNGFGNmRDOWU5MjhGNGU5OUQwIiwicmVzcG9uc2VfbW9kZSI6InBvc3QiLCJyZXNwb25zZV9jb250ZXh0IjoicnAiLCJub25jZSI6Im1kSFdxQnc1TlRkNTVlckJjcFlBdmNrMEdVOHRDQWZJYUdscVVHVE1rREEiLCJzdGF0ZSI6ImYyYTIzYTNkZDI2MWQ4NTczOGE1ZWMyYyIsInJlZ2lzdHJhdGlvbiI6eyJkaWRfbWV0aG9kc19zdXBwb3J0ZWQiOlsiZGlkOmV0aHI6Il0sInN1YmplY3RfaWRlbnRpZmllcnNfc3VwcG9ydGVkIjoiZGlkIn19.gPLLvFhD77MJC7IulbvdZ1dm0A1pXMh5VxFfz1ExMA_IQZmBdjyXih6RMWvYFh3Hn0Cn8R_su-ki5OP9HH7jLQ
```

### verifyJWT

Verifies a SIOP Authentication Request JWT. Throws an error if the verifation fails. Returns the verified JWT and metadata if the verification succeeds

#### Usage

```typescript
const verifyOpts: VerifyAuthenticationRequestOpts = {
   verification: {
      mode: VerificationMode.INTERNAL,
      resolveOpts: {
         didMethods: ["ethr"]
      }
   },
}
const jwt = 'ey..........' // JWT created by RP
AuthenticationRequest.verifyJWT(jwt).then(req => {
   console.log(`issuer: ${req.issuer}`);
   console.log(JSON.stringify(req.signer));
});
// issuer: "did:ethr:0x56C4b92D4a6083Fcee825893A29023cDdfff5c66"
// "signer": {
//      "id": "did:ethr:0x56C4b92D4a6083Fcee825893A29023cDdfff5c66#controller",
//      "type": "EcdsaSecp256k1RecoveryMethod2020",
//      "controller": "did:ethr:0x56C4b92D4a6083Fcee825893A29023cDdfff5c66",
//      "blockchainAccountId": "0x56C4b92D4a6083Fcee825893A29023cDdfff5c66@eip155:1"
// }
```

## AuthenticationResponse class

### verifyAuthResponse

Verifies a DidAuth ID Response Token and the audience. Return a DID Auth Validation Response, which contains the JWT payload as well as the verification method that signed the JWT.

#### Usage

```typescript
verifyAuthResponse('ey....', 'my-audience').then(resp => {
    console.log(JSON.stringify(resp.signer));
    // output: 
    //{
    //    id: 'did:eosio:example#key-0',
    //    type: 'authentication',
    //    controller: 'did:eosio:example',
    //    publicKeyHex: '1a3b....'
    //}
    
    console.log(resp.payload.nonce);
    // output: 5c1d29c1-cf7d-4e14-9305-9db46d8c1916
});
```

### verifyAccessToken

Verifies the bearer access token on the RP side as received from the OP/client. Throws an error if the token is invalid, otherwise returns the JWT

#### Usage

```typescript
verifyAccessToken('ey......').then(jwt => {
    console.log(`iss: ${jwt.iss}`);
    // output: iss: did:eosio:example
})
```

Reference: [DID-Auth-Siop](https://github.com/Sphereon-Opensource/did-auth-siop)
