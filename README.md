# UMA Macaroons

## Tagline

Macaroons, made from scratch using an UMA recipe with the fresh HMAC ingredients.

## Introduction

Bearer tokens are vulnerable at rest and in transit when an attacker is able to intercept a token to illegally access private information. In order to mitigate some of the risk associated with bearer tokens, UMA Macaroons may be used instead of bearer tokens. UMA Macaroons are cryptographically chained tokens bearing a chronological tamper-resistant record of all their possessors and the changes that have been made to them. In the authorization flow, UMA Macaroons use a complex combination of [Chained-MACs-with-Multiple-Messages][4] and [Chained-MACs-with-Multiple-Keys][5] constructions as a correlation mechanism among all participants and their data. UMA Macaroons adopt the User-Managed Access concept of authorization server, resource server, client, resource owner and requesting party.

## Key Differences from Google Macaroons

* Authenticated possessors.
* Claims are used instead of caveats.
* Different HMAC chaining.
* Verification at the authorization server.

Following we use the term *macaroon* to refer to UMA Macaroon.

## Concept of MACs Chaining

The [POCOP Token Mechanism][6] is used to construct macaroons.

1. To ensure integrity protection of macaroon claims, the first macaroon uses a [Chained-MACs-with-Multiple-Messages][4] construction. All MACs must be discarded after use.

MAC<sub><i>macaroon_1</i></sub> = HMAC(...HMAC(HMAC(K<sub><i>possessor_1</i></sub>, claims_1<sub><i>possessor_1</i></sub>), claims_2<sub><i>possessor_1</i></sub>), ...claims_n<sub><i>possessor_1</i></sub>)

2. [Chained-MACs-with-Multiple-Keys][5] construction is used to assure the authenticity of macaroons. The input MAC<sub><i>macaroon_1</i></sub> must be discarded after use. The final MAC<sub><i>macaroon_1</i></sub> can be published, there is no need to hide it.

MAC<sub><i>macaroon_1</i></sub> = HMAC(K<sub><i>possessor_1</i></sub>, MAC<sub><i>macaroon_1</i></sub>)

- Hop to the possessor_2.

MAC<sub><i>macaroon_1</i></sub> = HMAC(K<sub><i>possessor_2</i></sub>, MAC<sub><i>macaroon_1</i></sub>)

3. The second macaroon uses the [Chained-MACs-with-Multiple-Messages][4] construction in a similar manner to the first macaroon. The MAC<sub><i>macaroon_1</i></sub> is added to the possessor_2 macaroon in the first claims. The other MACs must be discarded after use.

MAC<sub><i>macaroon_2</i></sub> = HMAC(...HMAC(HMAC(K<sub><i>possessor_2</i></sub>, MAC<sub><i>macaroon_1</i></sub>), claims_2<sub><i>possessor_2</i></sub>), ...claims_n<sub><i>possessor_2</i></sub>)

Macaroons possessors must be registered at the authorization server (public clients can use dynamic registration to become confidential clients). Macaroons are verified via the introspection endpoint of the authorization server.

## Use Cases

Advanced authorization scenarios e.g. chained resource servers.

### Example of Chained Macaroons

Each macaroon contains the mandatory claims:

* The random NONCE to prevent replay attack.
* The claim that identifies who created the token.
* The timestamp of when the token was issued.

The HMAC chain may started with an AS or any other registered client. 

- The AS is the first macaroon possessor.<br><br>
MAC<sub><i>AS</i></sub> = HMAC(K<sub><i>AS</i></sub>, NONCE<sub><i>AS</i></sub>)<br>
MAC<sub><i>AS</i></sub> = HMAC(MAC<sub><i>AS</i></sub>, claims_1<sub><i>AS</i></sub>)<br>
MAC<sub><i>AS</i></sub> = HMAC(MAC<sub><i>AS</i></sub>, claims_2<sub><i>AS</i></sub>)<br>
...<br>
MAC<sub><i>AS</i></sub> = HMAC(MAC<sub><i>AS</i></sub>, claims_n<sub><i>AS</i></sub>)<br>
MAC<sub><i>AS</i></sub> = HMAC(K<sub><i>AS</i></sub>, MAC<sub><i>AS</i></sub>)<br>
- Hop to the next possessor – the client.<br><br>
MAC<sub><i>AS</i></sub> = HMAC(K<sub><i>client</i></sub>, MAC<sub><i>AS</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(K<sub><i>client</i></sub>, NONCE<sub><i>client</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, MAC<sub><i>AS</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, claims_1<sub><i>client</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, claims_2<sub><i>client</i></sub>)<br>
...<br>
MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, claims_n<sub><i>client</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(K<sub><i>client</i></sub>, MAC<sub><i>client</i></sub>)<br>
- Hop to the next possessor – the RS_1.<br><br>
MAC<sub><i>client</i></sub> = HMAC(K<sub><i>RS_1</i></sub>, MAC<sub><i>client</i></sub>)<br>
MAC<sub><i>RS_1</i></sub> = HMAC(K<sub><i>RS_1</i></sub>, NONCE<sub><i>RS_1</i></sub>)<br>
MAC<sub><i>RS_1</i></sub> = HMAC(MAC<sub><i>RS_1</i></sub>, MAC<sub><i>client</i></sub>)<br>
MAC<sub><i>RS_1</i></sub> = HMAC(MAC<sub><i>RS_1</i></sub>, claims_1<sub><i>RS_1</i></sub>)<br>
MAC<sub><i>RS_1</i></sub> = HMAC(MAC<sub><i>RS_1</i></sub>, claims_2<sub><i>RS_1</i></sub>)<br>
...<br>
MAC<sub><i>RS_1</i></sub> = HMAC(MAC<sub><i>RS_1</i></sub>, claims_n<sub><i>RS_1</i></sub>)<br>
MAC<sub><i>RS_1</i></sub> = HMAC(K<sub><i>RS_1</i></sub>, MAC<sub><i>RS_1</i></sub>)<br>
- Hop to the next possessor – the RS_2.<br><br>
MAC<sub><i>RS_1</i></sub> = HMAC(K<sub><i>RS_2</i></sub>, MAC<sub><i>RS_1</i></sub>)<br>
MAC<sub><i>RS_2</i></sub> = HMAC(K<sub><i>RS_2</i></sub>, NONCE<sub><i>RS_2</i></sub>)<br>
MAC<sub><i>RS_2</i></sub> = HMAC(MAC<sub><i>RS_2</i></sub>, MAC<sub><i>RS_1</i></sub>)<br>
MAC<sub><i>RS_2</i></sub> = HMAC(MAC<sub><i>RS_2</i></sub>, claims_1<sub><i>RS_2</i></sub>)<br>
MAC<sub><i>RS_2</i></sub> = HMAC(MAC<sub><i>RS_2</i></sub>, claim_2<sub><i>RS_2</i></sub>)<br>
...<br>
MAC<sub><i>RS_2</i></sub> = HMAC(MAC<sub><i>RS_2</i></sub>, claims_2<sub><i>RS_2</i></sub>)<br>
MAC<sub><i>RS_2</i></sub> = HMAC(K<sub><i>RS_2</i></sub>, MAC<sub><i>RS_2</i></sub>)<br>

- The last MAC<sub><i>RS_2</i></sub> can be verified via the introspection endpoint of the AS.

## Nested Macaroon / Third-Party Claims

A macaroon can contain another macaroon.

### Example of Nested Macaroon

This is an excerpt from the above example extended by third party claims.

...
- Hop to the next possessor – the client.<br><br>
MAC<sub><i>AS</i></sub> = HMAC(K<sub><i>client</i></sub>, MAC<sub><i>AS</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(K<sub><i>client</i></sub>, NONCE<sub><i>client</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, MAC<sub><i>AS</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(K<sub><i>client</i></sub>, MAC<sub><i>client</i></sub>)<br><br>
- Hop to the next possessor – the AS_third_party.<br><br>
MAC<sub><i>client</i></sub> = HMAC(K<sub><i>AS_third_party</i></sub>, MAC<sub><i>client</i></sub>)<br>
MAC<sub><i>AS_third_party</i></sub> = HMAC(K<sub><i>AS_third_party</i></sub>, NONCE<sub><i>AS_third_party</i></sub>)<br>
MAC<sub><i>AS_third_party</i></sub> = HMAC(MAC<sub><i>AS_third_party</i></sub>, MAC<sub><i>client</i></sub>)<br>
MAC<sub><i>AS_third_party</i></sub> = HMAC(MAC<sub><i>AS_third_party</i></sub>, claims_1<sub><i>AS_third_party</i></sub>)<br>
MAC<sub><i>AS_third_party</i></sub> = HMAC(MAC<sub><i>AS_third_party</i></sub>, claims_2<sub><i>AS_third_party</i></sub>)<br>
...<br>
MAC<sub><i>AS_third_party</i></sub> = HMAC(MAC<sub><i>AS_third_party</i></sub>, claims_n<sub><i>AS_third_party</i></sub>)<br>
MAC<sub><i>AS_third_party</i></sub> = HMAC(K<sub><i>AS_third_party</i></sub>, MAC<sub><i>AS_third_party</i></sub>)<br><br>
- Hop to the next possessor – back to the client.<br><br>
MAC<sub><i>AS_third_party</i></sub> = HMAC(K<sub><i>client</i></sub>, MAC<sub><i>AS_third_party</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, MAC<sub><i>AS_third_party</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, claims_1<sub><i>client</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, claims_2<sub><i>client</i></sub>)<br>
...<br>
MAC<sub><i>client</i></sub> = HMAC(MAC<sub><i>client</i></sub>, claims_n<sub><i>client</i></sub>)<br>
MAC<sub><i>client</i></sub> = HMAC(K<sub><i>client</i></sub>, MAC<sub><i>client</i></sub>)<br><br>
- Hop to the next possessor – the RS_1.

...

## Confidential Claims

The third-party claims can be chained using the AES-GCM authenticated encryption algorithm instead of the HMAC message authentication algorithm.

## Conclusion

(TBD)

## Acknowledgment

Credits go to [WG - User-Managed Access][1] and [Google Research Publications][2].

[1]: https://kantarainitiative.org/confluence/display/uma/Home
[2]: https://research.google/pubs/pub41892/
[3]: https://github.com/umalabs/uma-pocop-tokens
[4]: https://github.com/umalabs/uma-pocop-tokens#chained-macs-with-multiple-messages
[5]: https://github.com/umalabs/uma-pocop-tokens#chained-macs-with-multiple-keys
[6]: https://github.com/umalabs/uma-pocop-tokens#pocop-token-mechanism
