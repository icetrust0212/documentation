---
title: zkPass Credential
sidebar_position: 4
description: The Verida Wallet supports the zkPass verifiable credentials.
keywords: [Verida, zkPass, Zero Knowledge, Credentials, Verifiable Credentials]
---

import AppleAppStore from '/img/app_store_apple.svg';
import GooglePlay from '/img/play_store_google.svg';

# zkPass

The Verida Wallet supports zkPass verifiable credentials. This allows users to receive and store zkPass credentials as well as reply to proof requests in a privacy-preserving way thanks to zkPass technology.

To learn more about zkPass, [check zkPass official doc](https://zkpass.gitbook.io/zkpass/user-guides/overview) and head over to [zkPass developer documentation](https://zkpass.gitbook.io/zkpass/developer-guides/extension-js-sdk).

## Wallet Users

Users can install the Verida Wallet to receive verifiable credentials from Issuers using the Zero Knowledge technology. These credentials are stored in your Vault (your private and secured storage space on the Verida Network) and therefore shown in the Verida Wallet alongside other credentials.

Verifiers can also send you zkPass proof requests. The Verida Wallet will automatically generate the Zero-Knowledge proof (ZKP) for you and send it to the Verifier. The Zero-Knowledge proof means no data is actually shared with the Verifier, only the fact that you have a valid credential satisfying the request.

## Pre-requirement

You need to install [zkPass TransGate extension](https://zkpass.gitbook.io/zkpass/user-guides/transgate) to issue zkPass credentials.

### Issuing a zkPass credential

The [proof-connector](https://proofs.verida.network/) app can issue zkPass credential.
You need to provide `veridaDid` where generated credential will go to in the url. For example, the url can be like [this](https://proof.verida.network/add-credential?veridaDid=did:vda:mainnet:0x378E209c8Cdc071b1ad7d0b4aBE300309A7bE541), or [this](https://proof.verida.network/add-credential?veridaDid=did:vda:mainnet:0x378E209c8Cdc071b1ad7d0b4aBE300309A7bE541&schemaId=ef39adb26c88439591279e25e7856b61).

It will redirects you to page where you select schemas.
Once you select schema from zkPass protocol, you can start process to create credentials.

![zkPass credential issuer - Start generating](/img/extensions/zkpass/start-process.png)

It will redirects you to the platform (For example: Binance) and once you click `Start` button in TransGate extension, the the process should start.
![zkPass credential issuer - TransGate](/img/extensions/zkpass/transgate.png)


```
export type zkPassResult = {
  allocatorAddress: string;
  allocatorSignature: string;
  publicFields: any[];
  publicFieldsHash: string;
  taskId: string;
  uHash: string;
  validatorAddress: string;
  validatorSignature: string;
  zkPassSchemaId?: string;
  zkPassSchemaLabel?: string;
};

async function processZK(schemaId: string): Promise<zkPassResult> {
  // Create the connector instance
  const connector = new TransgateConnect(ZKPASS_APP_ID);
  // Check if the TransGate extension is installed
  // If it returns false, please prompt to install it from chrome web store
  const isAvailable = await connector.isTransgateAvailable();
  if (isAvailable) {
    const res = await connector.launch(schemaId);

    return res as zkPassResult;
  } else {
    throw new Error("You need to install ZKPass extension");
  }
}
```

### Verifying a zkPass credential

The [proof-connector](https://proofs.verida.network/verify) can verify a zero knowledge proof generated from a zkPass credential stored in the user's Verida Wallet. More information is available in the [zkPass Verification documentation](https://zkpass.gitbook.io/zkpass/developer-guides/how-to-verify-the-result).


![zkPass credential verifier - Select Credential](/img/extensions/zkpass/select-credential.png)

```
import Web3 from "web3";
import { VeridaCredentialRecord } from "@verida/verifiable-credentials";
import { zkPassResult } from "../@types";

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
    } = credentialData as zkPassResult;

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