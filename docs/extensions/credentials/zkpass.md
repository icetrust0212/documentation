---
title: zkPass Credential
sidebar_position: 4
description: The Verida Wallet supports the zkPass verifiable credentials.
keywords: [Verida, zkPass, Zero Knowledge, Credentials, Verifiable Credentials]
---

import AppleAppStore from '/img/app_store_apple.svg';
import GooglePlay from '/img/play_store_google.svg';

# zkPass credentials

The Verida Wallet supports zkPass verifiable credentials. This allows users to receive and store zkPass credentials as well as reply to proof requests in a privacy-preserving way thanks to zkPass technology.

To learn more about zkPass, [check zkPass official doc](https://zkpass.gitbook.io/zkpass/user-guides/overview) and head over to [zkPass developer documentation](https://zkpass.gitbook.io/zkpass/developer-guides/extension-js-sdk).

## Wallet Users

Users can install the Verida Wallet to receive verifiable credentials from Issuers using the Zero Knowledge technology. These credentials are stored in your Vault (your private and secured storage space on the Verida Network) and therefore shown in the Verida Wallet alongside other credentials.

Verifiers can also send you zkPass proof requests. The Verida Wallet will automatically generate the Zero-Knowledge proof (ZKP) for you and send it to the Verifier. The Zero-Knowledge proof means no data is actually shared with the Verifier, only the fact that you have a valid credential satisfying the request.

## Request zkPass credential

You can request credential which is generated from zkPass protocol for your purpose.

### Example code

```
  const did = "..."; // Verida Did
  // Get message object from verida context
  const messaging = await context.getMessaging();

  // setup a callback to show the response
  await messaging.onMessage((data) => {
    // This callback should be called once user shares credential
    console.log('Received credentials: ', data);
  });

  const messageType = "inbox/type/dataRequest";
  const config = {
    did,
    recipientContextName: "Verida: Vault",
  };
  const dataToSend = {
    requestSchema: "https://common.schemas.verida.io/credential/base/v0.2.0/schema.json",
    filter: {
      $or: [
        { credentialSchema: "https://common.schemas.verida.io/credential/zkpass/v0.1.0/schema.json" }
      ]
    },
    userSelect: true,
  };

  // This is the DID the message will go to
  const requestFromDID = did;
  const messageSubject = "Please select your verifiable credential to verify",

  const res = await messaging.send(
    requestFromDID,
    messageType,
    dataToSend,
    msg.messageSubject,
    config
  );

  console.log("Request sent");
```

![Request zkPass credential from Verida Wallet](/img/extensions/zkpass/request-credential-process.png)

## Issuing a zkPass credential

You need to install [zkPass TransGate extension](https://zkpass.gitbook.io/zkpass/user-guides/transgate) to issue zkPass credentials.

If you don't have credentials in your Verida Wallet, you can issue zkPass credential through the [proof-connector](https://prove.verida.network/).
You need to provide `veridaDid` where generated credential will go to in the url. For example, the url can be like this:

```
 https://prove.verida.network/add-credential?veridaDid=[veridaDid]
 https://prove.verida.network/add-credential?veridaDid=[veridaDid]&schemaId=[zkPass schemaId]
```

### Available zkPass schemas

```
Verify ownership of Uber account: ef39adb26c88439591279e25e7856b61
Verify ownership of Discord account: c0519cf1b26c403096a6af51f41e3f8d
Verify ownership of Binance account: 556ed720e40c4fb48ea7545708e47c90
Verify ownership of Bybit account: afc3447c5b0f48588db5640472691d37
Verify ownership of KuCoin account: 01c1439e852f47aaa4f697cef14d3e94
Verify ownership of MEXC account: d73e2c2227f642dcbade873ff2b09173
Verify ownership of Gate account: a3b6bf7a231e45a582ffd0e50245c849

```

It redirects you to page where you select schemas.
Once you select schema from zkPass protocol, you can start process to create credentials.

### Example code

```
import TransgateConnect from "@zkpass/transgate-js-sdk";

// You can create your own app in zkPass dashboard.
const ZKPASS_APP_ID = "bced693b-bedc-464c-8250-566743ff5855";
// The schema Id for Uber account ownership verification
const schemaId = "ef39adb26c88439591279e25e7856b61";

// Create the connector instance
const connector = new TransgateConnect(ZKPASS_APP_ID);
// Check if the TransGate extension is installed
// If it returns false, please prompt to install it from chrome web store
const isAvailable = await connector.isTransgateAvailable();
if (isAvailable) {
  const res = await connector.launch(schemaId);

  return res;
} else {
  throw new Error("You need to install zkPass extension");
}
```

![zkPass credential issuer - Start generating](/img/extensions/zkpass/start-process.png)

It will redirect you to the platform (For example: Binance) and once you click `Start` button in TransGate extension, the the process should start.
![zkPass credential issuer - TransGate](/img/extensions/zkpass/transgate.png)

## Verifying a zkPass credential

The [proof-connector](https://prove.verida.network/verify) can verify a zero knowledge proof generated from a zkPass credential stored in the user's Verida Wallet. More information is available in the [zkPass Verification documentation](https://zkpass.gitbook.io/zkpass/developer-guides/how-to-verify-the-result).

![zkPass credential verifier - Select Credential](/img/extensions/zkpass/select-credential.png)

### Example code

```
import Web3 from "web3";
import { VeridaCredentialRecord } from "@verida/verifiable-credentials";

const web3 = new Web3();
export const verifyZKProof = (proof: VeridaCredentialRecord): boolean => {
  try {
    const { credentialData } = proof;
    const {
      taskId,
      zkPassSchemaId,
      validatorAddress,
      allocatorSignature,
      uHash,
      publicFields,
      publicFieldsHash,
      validatorSignature,
      allocatorAddress
    } = credentialData;

    // verify allocator signature
    const taskIdHex = Web3.utils.stringToHex(taskId);
    const schemaIdHex = Web3.utils.stringToHex(zkPassSchemaId);
    const encodeParams = web3.eth.abi.encodeParameters(
      ["bytes32", "bytes32", "address"],
      [taskIdHex, schemaIdHex, validatorAddress]
    );
    const paramsHash = Web3.utils.soliditySha3(encodeParams);
    const signedAllocationAddress = web3.eth.accounts.recover(
      paramsHash,
      allocatorSignature
    );

    if (signedAllocationAddress !== allocatorAddress) {
      return false;
    }

    // verify validator signature
    const encodeParamsForValidator = web3.eth.abi.encodeParameters(
      ["bytes32", "bytes32", "bytes32", "bytes32"],
      [taskIdHex, schemaIdHex, uHash, publicFieldsHash]
    );
    const paramsHashForValidator = Web3.utils.soliditySha3(
      encodeParamsForValidator
    );
    const signedValidatorAddress = web3.eth.accounts.recover(
      paramsHashForValidator,
      validatorSignature
    );

    if (signedValidatorAddress !== validatorAddress) {
      return false;
    }

    return true;
  } catch (err) {
    console.log("something went wrong while verify result from zk: ", err);
    return false;
  }
};
```
