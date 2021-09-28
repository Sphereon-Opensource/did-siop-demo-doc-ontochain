## Summary

In this demo, the authentication flow is reproduced using our [DIDSIOP library](#did-auth-siop). The [Demo WEB](#onto-web-demo)
app plays the role of the relying party (RP) and the [Demo RN](#demo-rn) app plays the role of the OpenID provider (OP). 
[GimlyID-QR-Code](#gimlyid-qr-code) and [GimlyID-QR-Code-Scanner](#gimlyid-qr-code-scanner) are employed to respectively
generate and read the QR code used in the authentication process. The flow starts with the RP. The user clicks the button
on the web page and a QR code is generated. The user reads the QR code using the Demo RN app which requests access to 
protected resources hosted by the RP. Since the user is not logged in, an authentication request will be sent back to 
the OP. If the request is valid, the OP sends an authentication response back to the RP, which verifies the validity of
response. If all steps succeed, access to the protected resources is granted to the user.

## GimlyID QR Code
This is the QR code generator, it works with React and React-Native. It generates a QR code containing:

| prop                    | type                         | default value | description                                                                                                                                                                                              |
| ----------------------- | ---------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`                  | `QRType`                     |               | This is the type stored within the QR code. Options: [AUTHENTICATION].                                                                                                                                   |
| `did`                   | `string`                     |               | This is the DID that the relying party (the party that integrated the authentication support on their website) will use to sign/encrypt data towards the client. This will be stored within the QR code. |
| `mode`                  | `QRMode`                     |               | This is the mode stored within the QR code. Options: [DID_AUTH_SIOP_V2].                                                                                                                                 |
| `redirectUrl`           | `string`                     |               | This is a redirect url that will be stored within the QR code.
| `state`                 | `string`                     |               | This is a string representing a short UUID

Reference: [GimlyID-QR-Code](https://github.com/Sphereon-Opensource/gimlyid-qr-code)

## GimlyID QR Code Scanner
A GimlyID QR code scanner component for React-Native. This scanner can scan QR codes generated by the gimlyid-qr-code component.

## Api

| prop                 | type                         | default value | description                                                                                |
| -------------------- | ---------------------------- | ------------- | ------------------------------------------------------------------------------------------ |
| `style`              | `React.CSSProperties`        |               | Sets the styling of the qr code scanner.                                                   |
| `onRead`             | `Function`                   |               | The onRead will be called when the QR code is read and will have access to the QR content. |

Reference: [GimlyID-QR-Code-Scanner](https://github.com/Sphereon-Opensource/gimlyid-qr-code-scanner):

## ONTO web demo
This is the web application (RP) that generates the QR code and starts the authentication process;

## Endpoints
| endpoint                 | http method |
| -------------------------| ------------|
| `/authenticate`          | `GET`
| `/get-qr-variables`      | `GET`
| `/register-auth-request` | `POST`
| `/poll-auth-response`    | `POST`      |

Reference: [Onto-Web-Demo](https://github.com/Sphereon/onto-web-demo)

## Demo RN
This is the React-Native application (OP) that stores the self issued credentials
-> Needs input from Sander

Reference: [Demo-RN](https://github.com/Sphereon/rn-did-siop-example-app)

## DID Auth Siop
SIOP v2 is an extension of OpenID Connect to allow End-users to act as OpenID Providers (OPs) themselves. Using Self-Issued
OPs, End-users can authenticate themselves and present claims directly to the Relying Parties (RPs), typically a webapp,
without relying on a third-party Identity Provider. This makes the solution fully self sovereign, as it does not rely on
any third parties and strictly happens peer 2 peer, but still uses the OpenID Connect protocol. Next to the user acting 
as an OpenID Provider, this library also includes support for Verifiable Presentations using the Presentation Exchange 
support provided by our pe-js library. This means that the Relying Party can pose submission requirements on the 
Verifiable Credentials it would like to receive from the client/OP. The OP then checks whether it has the credentials to
support the submission requirements. Only if that is the case it will send the relevant (parts of the) credentials as a 
Verifiable Presentation in the Authentication Response destined for the Webapp/Relying Party. The relying party in turn 
checks validity of the Verifiable Presentation(s) as well as the match with the submission requirements. Only if 
everything is verified successfully the RP serves the protected page(s). This means that the authentication can be 
extended with claims about the authenticating entity, but it can also be used to easily consume credentials from 
supporting applications, without having to setup DIDComm connections for instance. The term Self-Issued comes from the 
fact that the End-users (OP) issue self-signed ID Tokens to prove validity of the identifiers and claims. This is a 
trust model different from that of the rest of OpenID Connect where OP is run by the third party who issues ID Tokens 
on behalf of the End-user to the Relying Party upon the End-user's consent. This means the End-User is in control about 
his/her data instead of the 3rd party OP.

# Steps involved

Flow diagram:

![Flow diagram](https://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/Sphereon-Opensource/did-auth-siop/develop/docs/auth-flow-diagram.txt)


1. Client (OP) initiates an Auth request by POST-ing to an endpoint, like for instance `/did-siop/v1/authentications` or clicking a Login button and scanning a QR code
2. Web (RP) receives the request and access the RP object which creates the authentication request as JWT, signs it and returns the response as an OpenID Connect URI
    1. JWT example:
    ```json
    // JWT Header
    {
      "alg": "ES256K",
      "kid": "did:ethr:0xcBe71d18b5F1259faA9fEE8f9a5FAbe2372BE8c9#controller",
      "typ": "JWT"
    }
     
    // JWT Payload
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

    ````json
    // JWT encoded ID Token
    // JWT Header
    {
      "alg": "ES256K",
      "kid": "did:ethr:0x998D43DA5d9d78500898346baf2d9B1E39Eb0Dda#keys-1",
      "typ": "JWT",
    }
    // JWT Payload
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
    ````

    2. Sign the ID token using the DID key (kid) using JWS scheme (JWS Compact Serialization, https://datatracker.ietf.org/doc/html/rfc7515#section-7.1) and send it to the RP:

   `BASE64URL(UTF8(JWS Protected Header)) || '.' ||
   BASE64URL(JWS Payload) || '.' ||
   BASE64URL(JWS Signature)`

8. OP returns the Auth response and jwt object to the client
9. Client does an HTTP POST to redirect_uri from the request (and the aud in the response): https://acme.com/siop/v1/sessions using "application/x-www-form-urlencoded"
10. Web receives the ID token (auth response) and uses the RP's object verify method
11. RP performs the validation of the token, including signature validation, expiration and Verifiable Presentations if any. It returns the Verified Auth Response to WEB
12. WEB returns a 200 response to Client with a redirect to another page (logged in or confirmation of VP receipt etc).
13. From that moment on Client can use the Auth Response as bearer token as long as it is valid

### Relying Party (RP)

````typescript
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
````
### OpenID Provider (SIOP)
````typescript
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
````
### RP creates the Authentication Request
````typescript
const reqURI = await rp.createAuthenticationRequest({
    nonce: "qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg",
    state: "b32f0087fc9816eb813fd11f"
});

console.log(`nonce: ${reqURI.nonce}, state: ${reqURI.state}`);
// nonce: qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg, state: b32f0087fc9816eb813fd11f


console.log(reqURI.encodedUri)
// openid://?response_type=id_token&scope=openid&client_id=did.......&jwt=ey..........
````
### OP creates the Authentication Response using the Request
````typescript
// The create method also calls the verify and parse methods, so no need to do it manually
const authRespWithJWT = await op.createAuthenticationResponse(requestURI.jwt);

// The below call would be equivilent if the optional steps above would have been used
//const authRespWithJWT = await op.createAuthenticationResponseFromVerifiedRequest(verifiedReq);
````
### RP verifies the Authentication Response
````typescript
const verifiedAuthResponseWithJWT = await rp.verifyAuthenticationResponseJwt(authRespWithJWT.jwt, {
    audience: EXAMPLE_REDIRECT_URL,
})

expect(verifiedAuthResponseWithJWT.jwt).toBeDefined();
expect(verifiedAuthResponseWithJWT.payload.state).toMatch("b32f0087fc9816eb813fd11f");
expect(verifiedAuthResponseWithJWT.payload.nonce).toMatch("qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg");
````
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
Reference: [DID-Auth-Siop](https://github.com/Sphereon-Opensource/did-auth-siop)

## Gimly-ID-Mobile-App
Reference: [Gimly-ID-Mobile-App](https://github.com/Gimly-Blockchain/gimly-id-mobile-app)

Setting up/ using the Docker containers for the proj

