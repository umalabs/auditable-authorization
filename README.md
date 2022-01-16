# Auditable Authorization

## Abstract

Bearer tokens are vulnerable at rest and in transit when an attacker is able to intercept a token to illegally access private information. In order to mitigate some of the risk associated with bearer tokens, the Authorization Audit Trail (AAT) may be used instead of bearer tokens in the authorization process. The AAT consists of cryptographically chained blocks of data bearing a chronological tamper-resistant records of all their possessors and the changes that have been made by them. All sensitive information are encrypted to preserve confidentiality. The access control relies on a real-time auditability of the AAT by the authorization server. The AAT concept is compatible with existing OAuth2 and UMA protocols.
