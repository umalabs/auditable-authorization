# Auditable Authorization

## Introduction

Bearer tokens are vulnerable at rest and in transit when an attacker is able to intercept a token to illegally access private information. In order to mitigate some of the risk associated with bearer tokens, Authorization Trails may be used instead of bare bearer tokens. The Authorization Trail consists of cryptographically chained blocks of data bearing a chronological tamper-resistant records of all their possessors and the changes that have been made to them. In the authorization flow, the Authorization Trails ecosystem relies on auditability of records rather than on token minting and token verification. The Authorization Trails adopt the User-Managed Access concept of authorization server, resource server, client, resource owner and requesting party.
