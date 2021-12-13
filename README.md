# UMA Macaroons

## Tagline

Macaroons, made from scratch using an UMA recipe with the fresh HMAC ingredients.

## Introduction

Bearer tokens are vulnerable at rest and in transit when an attacker is able to intercept a token to illegally access private information. In order to mitigate some of the risk associated with bearer tokens, UMA Macaroons may be used instead of bearer tokens. UMA Macaroon is a chronological tamper-resistant record of all the possessors of the macaroon and the changes that have been made. UMA Macaroons use a combined [Chained MACs with Multiple Messages][4] / [Chained MACs with Multiple Keys][5] construction as a correlation mechanism among all participants and their data in the authorization flow.

## Main Differences from Google Macaroons

* Authenticated macaroon possessors.
* Claims are used instead of caveats.
* An authorization server is required.
* Different HMAC chaining is used.
* Macaroons are verified at the authorization server.

## Concept

The [POCOP Token Mechanism][6] is used to construct macaroons.

1. To ensure integrity protection of macaroon claims, macaroons use a [Chained MACs with Multiple Messages][4] construction. All MACs must be discarded after use.

MAC<sub><i>macaroon_1</i></sub> = HMAC(...HMAC(HMAC(K<sub><i>possessor_1</i></sub>, claim_1<sub><i>possessor_1</i></sub>), claim_2<sub><i>possessor_1</i></sub>,) ...claim_n<sub><i>possessor_1</i></sub>)

2. [Chained MACs with Multiple Keys][5] construction is used to assure the authenticity of macaroons. The input MAC<sub><i>macaroon_1</i></sub> must be discarded after use. The final MAC<sub><i>macaroon_1</i></sub> can be published, there is no need to hide it.

MAC<sub><i>macaroon_1</i></sub> = HMAC(K<sub><i>possessor_1</i></sub>, MAC<sub><i>macaroon_1</i></sub>)

- Hop to the possessor_2.

MAC<sub><i>macaroon_1</i></sub> = HMAC(K<sub><i>possessor_2</i></sub>, MAC<sub><i>macaroon_1</i></sub>)

3. Macaroons use a [Chained MACs with Multiple Messages][4] construction. The MAC<sub><i>macaroon_1</i></sub> is added to the possessor_2 macaroon as the first claim. The other MACs must be discarded after use.

MAC<sub><i>macaroon_2</i></sub> = HMAC(...HMAC(HMAC(K<sub><i>possessor_2</i></sub>, MAC<sub><i>macaroon_1</i></sub>), claim_2<sub><i>possessor_2</i></sub>), ...claim_n<sub><i>possessor_2</i></sub>)

Macaroons possessors must be registered at the authorization server (public clients can use dynamic registration to become confidential clients). Macaroons are verified via introspection endpoints at the authorization server.

## Use Case

Advanced authorization scenarios e.g. chained resource servers. 

### Example


The HMAC chain may started with an AS or any other registered client.

Each macaroon starts with a random NONCE to prevent replay attack.

Claim_1 is a mandatory "iss" claim that identifies who created the macaroon.

Claim_2 is an issued-at "iat" timestamp of the macaroon.

All claims are public.


MAC<sub><i>AS</i></sub> = HMAC(K<sub><i>AS</i></sub>, NONCE<sub><i>AS</i></sub>)

MAC<sub><i>AS</i></sub> = HMAC(MAC<sub><i>AS</i></sub>, claim_1<sub><i>AS</i></sub>)

MAC<sub><i>AS</i></sub> = HMAC(MAC<sub><i>AS</i></sub>, claim_2<sub><i>AS</i></sub>)

MAC<sub><i>AS</i></sub> = HMAC(K<sub><i>AS</i></sub>, MAC<sub><i>AS</i></sub>)

-

MAC<sub><i>AS</i></sub> = HMAC(K<sub><i>client</i></sub>, MAC<sub><i>AS</i></sub>)

MAC<sub><i>client</i></sub> = HMAC(K<sub><i>client</i></sub>, NONCE<sub><i>client</i></sub>)

MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, MAC<sub><i>AS</i></sub>)

MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, claim_1<sub><i>client</i></sub>)

MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, claim_2<sub><i>client</i></sub>)

MAC<sub><i>client</i></sub> = HMAC(K<sub><i>client</i></sub>, MAC<sub><i>client</i></sub>)

-

MAC<sub><i>client</i></sub> = HMAC(K<sub><i>RS_1</i></sub>, MAC<sub><i>client</i></sub>)

MAC<sub><i>RS_1</i></sub> = HMAC(K<sub><i>RS_1</i></sub>, NONCE<sub><i>RS_1</i></sub>)

MAC<sub><i>RS_1</i></sub> = HMAC(MAC<sub><i>RS_1</i></sub>, MAC<sub><i>client</i></sub>)

MAC<sub><i>RS_1</i></sub> = HMAC(MAC<sub><i>RS_1</i></sub>, claim_1<sub><i>RS_1</i></sub>)

MAC<sub><i>RS_1</i></sub> = HMAC(MAC<sub><i>RS_1</i></sub>, claim_2<sub><i>RS_1</i></sub>)

MAC<sub><i>RS_1</i></sub> = HMAC(K<sub><i>RS_1</i></sub>, MAC<sub><i>RS_1</i></sub>)

-

MAC<sub><i>RS_1</i></sub> = HMAC(K<sub><i>RS_2</i></sub>, MAC<sub><i>RS_1</i></sub>)

MAC<sub><i>RS_2</i></sub> = HMAC(K<sub><i>RS_2</i></sub>, NONCE<sub><i>RS_2</i></sub>)

MAC<sub><i>RS_2</i></sub> = HMAC(MAC<sub><i>RS_2</i></sub>, MAC<sub><i>RS_1</i></sub>)

MAC<sub><i>RS_2</i></sub> = HMAC(MAC<sub><i>RS_2</i></sub>, claim_1<sub><i>RS_2</i></sub>)

MAC<sub><i>RS_2</i></sub> = HMAC(MAC<sub><i>RS_2</i></sub>, claim_2<sub><i>RS_2</i></sub>)

MAC<sub><i>RS_2</i></sub> = HMAC(K<sub><i>RS_2</i></sub>, MAC<sub><i>RS_2</i></sub>)


## Nested/third-party claims

A claim can contain another macaroon.

## Confidential claims

Encrypted claims. (TBD)

## Conclusion

(TBD)

## Acknowledgment

Credits go to [WG - User-Managed Access][1] and [Google Research Publications][2].

[1]: https://kantarainitiative.org/confluence/display/uma/Home
[2]: https://research.google/pubs/pub41892/
[3]: https://github.com/umalabs/uma-pocop-tokens
[4]: https://github.com/umalabs/uma-pocop-tokens#chained-macs-with-multiple-messages
[5]: https://github.com/umalabs/uma-pocop-tokens#chained-macs-with-multiple-keys
[6]: https://github.com/umalabs/uma-pocop-tokens#pocop-mechanism
