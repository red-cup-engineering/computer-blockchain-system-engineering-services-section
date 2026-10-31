# @red-cup-engineering/verify-evm-signed-message-service

Verifies an EIP-4361 Sign-In with Ethereum message signed with EIP-191
personal_sign: parses the message, recovers the signer through the Group
EVM primitives, and checks the address, the expected bindings, and the
time window.

## Operation

~~~js
export function verifyEvmSignedMessage(request)
~~~

### Parameters

- request (@param {string} request.message The exact EIP-4361 message text that was signed.
 * @param {string} request.signature The 65-byte `personal_sign` signature as 0x-prefixed hex (r||s||v).
 * @param {{domain?: string, nonce?: string, address?: string, chainId?: number, uri?: string}} [request.expected] Bindings the relying party issued and requires the message to carry.
 * @param {string} [request.now] ISO timestamp for the time-window check; defaults to the wall clock.
 * ): An EvmSignedMessageVerificationRequest.

### Returns

{verified: true, address: string, domain: string, uri: string, chainId: number, nonce: string, issuedAt: string, expirationTime: string|null, notBefore: string|null, requestId: string|null, statement: string|null, resources: string[]} | {verified: false, reason: string}: 

## Command

verify-evm-signed-message reads one JSON value per input line and writes one JSON value per output line.

## Examples

### a malformed signature is refused before the message is parsed

Input:

~~~json
{
  "message": "hello",
  "signature": "0xnothex"
}
~~~

Output:

~~~json
{
  "verified": false,
  "reason": "signature must be a 0x-prefixed 130-hex-digit EIP-191 signature"
}
~~~

### a non-object request is refused

Input:

~~~json
null
~~~

Output:

~~~json
{
  "verified": false,
  "reason": "request must be a JSON object"
}
~~~
