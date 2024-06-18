# How to verify Address-Ownership

## Certificates

Certificates provide proof of ownership for a specific address. In combination with wallets, they offer a standardized method for signing agreements or creating a sign-in experience with VeChain.

Common use cases include user identification through sign-in and request verification between services.

### Anatomy

The structure of a certificate is clearly defined and includes a purpose, payloads, domain, and signer.

```ts
interface Certificate {
    /**
     * The purpose field indicates the intended use or context of the certificate.
     * For example, it could be used for identification, verification, or attestation.
     */
    purpose: string;

    /**
     * The payload field holds the actual content of the certificate.
     * This content can be of various types, such as text, images, or other data.
     */
    payload: {
        type: string;
        content: string;
    };

    /**
     * The domain field represents the specific context or domain for which the certificate is valid.
     * It helps ensure that the certificate is only applicable within the intended context.
     */
    domain: string;

    /**
     * The timestamp field records the time at which the certificate was created or issued.
     * This provides a temporal reference for the certificate's validity.
     */
    timestamp: number;

    /**
     * The signer field indicates the address of the entity that signs the certificate.
     * It is the public key address of the entity that issues the certificate.
     */
    signer: string;

    /**
     * The signature field contains the cryptographic signature generated by the issuer's private key.
     * This signature ensures the integrity and authenticity of the certificate's content.
     */
    signature?: string;
}
```

### Sign

To sign a certificate yourself, you will need to:

1. Obtain the signer's address.
2. Encode the certificate object into a JSON string. Avoid using `JSON.stringify` directly, as the key order may vary, potentially causing verification failures across different platforms, use `certificate.encode()` instead.
3. Sign the encoded string.

If you possess only the private key, you can derive the public address from its corresponding public key:

```js
// calculate signer from private key
const publicKey = secp256k1.derivePublicKey(privateKey);
cert.signer = addressUtils.fromPublicKey(publicKey);
```

Signing example:

```js
// encode & sign
const jsonStr = certificate.encode(cert);
const rawSignature = secp256k1.sign(blake2b256(jsonStr), privateKey);
const signature = `0x${rawSignature.toString('hex')}`;
```

Adding the `signature` attribute to the certificate turns it into a signed certificate, available for verification on other platforms.

{% hint style="info" %}
An API verification can consist of a client sending JSON.stringified and base64 encoded version in the headers. The API can then decode and verify the certificate.
{% endhint %}

### Verify

Verification is done using `certificate.verify(cert)` which will throw in case of any error. Catching errors is therefor important:

```js
try {
  certificate.verify(signedCert);
  console.log('verification successful')
}
catch (err) {
  console.log('verification failed', err.message)  
}
```

## Example Projects

### Sign & Verify

{% embed url="https://stackblitz.com/edit/vechain-sdk-certificate-sign?ctl=1&embed=1&file=index.mjs&hideExplorer=1&hideNavigation=1&view=editor" %}

### Sign in with Vechain using dApp-Kit