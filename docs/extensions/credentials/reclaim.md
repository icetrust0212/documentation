---
title: Reclaim Credential
sidebar_position: 5
description: The Verida Wallet supports the Reclaim verifiable credentials.
keywords:
  [
    Verida,
    Reclaim protocol,
    Zero Knowledge,
    Credentials,
    Verifiable Credentials,
  ]
---

import AppleAppStore from '/img/app_store_apple.svg';
import GooglePlay from '/img/play_store_google.svg';

# Reclaim Protocol

The Verida Wallet supports Reclaim protocol verifiable credentials. This allows users to receive and store Reclaim credentials as well as reply to proof requests in a privacy-preserving way thanks to Reclaim Zero Knowledge technology.

To learn more about Reclaim protocol, [check reclaim protocol official doc](https://docs.reclaimprotocol.org/) and head over to [Reclaim Protocol Whitepaper](https://docs.reclaimprotocol.org/whitepaper).

## Wallet Users

Users can install the Verida Wallet to receive verifiable credentials from Issuers using the Zero Knowledge technology. These credentials are stored in your Vault (your private and secured storage space on the Verida Network) and therefore shown in the Verida Wallet alongside other credentials.

Verifiers can also send you Reclaim proof requests. The Verida Wallet will automatically generate the Zero-Knowledge proof (ZKP) for you and send it to the Verifier. The Zero-Knowledge proof means no data is actually shared with the Verifier, only the fact that you have a valid credential satisfying the request.

### Issuing a Reclaim Protocol credential

The [proof-connector](https://proofs.verida.network/) app can issue Reclaim Protocol credential.
You need to provide `veridaDid` where generated credential will go to in the url. For example, the url can be like [this](https://proof.verida.network/add-credential?veridaDid=did:vda:mainnet:0x378E209c8Cdc071b1ad7d0b4aBE300309A7bE541), or [this](https://proof.verida.network/add-credential?veridaDid=did:vda:mainnet:0x378E209c8Cdc071b1ad7d0b4aBE300309A7bE541&schemaId=f3a4394b-191a-4889-9f5c-e0d70dc26fac).

It will redirects you to page where you select schemas.
Once you select schema from Reclaim protocol, you can start process to create credentials.

![Reclaim Protocol credential issuer - Start generating](/img/extensions/reclaim/start-process.png)

It will redirects you to the platform (For example: Uber, Kaggle) and the the process should start.
![Reclaim Protocol credential issuer](/img/extensions/reclaim/generating-proofs.png)

Check [this](https://docs.reclaimprotocol.org/node/quickstart) documentation how to generate proofs using Reclaim Protocol.

### Verifying a Reclaim Protocol credential

The [proof-connector](https://proofs.verida.network/verify) can verify a zero knowledge proof generated from a Reclaim Protocol credential stored in the user's Verida Wallet. More information is available in the [Reclaim Protocol Verification documentation](https://docs.reclaimprotocol.org/node/callback).

![Reclaim Protocol credential verifier - Select Credential](/img/extensions/zkpass/select-credential.png)

#### Verify the proofs

```
import { Reclaim } from '@reclaimprotocol/js-sdk'

app.post('/callback/', async (req, res) => {
  const sessionId = req.query.callbackId
  const proof = JSON.parse(decodeURIComponent(req.body))

  const isProofVerified = await Reclaim.verifySignedProof(proof)
  if (!isProofVerified) {
    return res.status(400).send({ message: 'Proof verification failed' })
  }
})

```

#### Verify the metadata

```
import { Reclaim } from '@reclaimprotocol/js-sdk'

app.post('/callback/', async (req, res) => {
  const sessionId = req.query.callbackId
  const proof = JSON.parse(decodeURIComponent(req.body))

  const isProofVerified = await ReclaimClient.verifySignedProof(proof)
  if (!isProofVerified) {
    return res.status(400).send({ message: 'Proof verification failed' })
  }

  const context = proof.claimData.context
  const extractedParameterValues = proof.extractedParameterValues
  // TODO: Verify with the context depending on your business logic
  // TODO: Save the proof to your backend

  return res.status(200).send({ message: 'Proof verified' })
})
```
