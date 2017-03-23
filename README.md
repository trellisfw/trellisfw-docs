# fpad-docs
Documentation for the fPAD framework.

This is a work-in-progress.  Documentation will be added as various components are finalized.

## Core data models
You can find definitions of the core data models in the `oada-formats` repository under "fpad".
This describes JSON documents for representing a food safety audit, corrective actions, and
food safety certificates.  

## fPAD Document Integrity Signatures

(Demo application to sign and verify documents is here.)[https://fpad.github.io/fpad-audit-sign-verify]

Food safety audit data is sensitive and therefore needs a means by which a recipient can verify
that the audit has not been tampered with since it was first created by an auditor.  fPAD 
document integrity signatures are intended to solve this problem.

As a basic outline, fPAD represents all documents as a JSON object.  When a trusted authority creates
one of these documents, they can apply a digital signature to that object.  This signature is saved with 
the object and included wherever that document may end up in the supply chain.  If the document itself
is tampered with, the signature will not match.  If the signature is tampered with, the document
will not match.  If both the signature and the document are tampered with, one must verify that the
signature came from a trusted authority in order to determine trust by checking if the public key
in the signature is from a trusted authority.

fPAD uses the (JSON Web Token (JWT) standard)[https://jwt.io] to represent the signature.  The process
for generating the signature is relatively straighforward, and example code which does this can 
be found in (https://github.com/fpad/fpad-audit-sign-verify)[https://github.com/fpad/fpad-audit-sign-verify].

To create a signature:
```
Step 1: turn the JSON object into a string.  Since everybody must create the same string on any platform,
some additional caveats apply in order to achieve consistent hashing: 
* the key names in the string must be in lexicographic (alphabetical) order.
* all key names must be quoted with double quotes.
* all whitespace, except whitespace within strings, must be removed.
* serialization must be UTF-8.
* because numeric representations differ on different platforms, all numbers in the document must be 
  represented as strings instead of Javascript numbers.

Step 2: Compute a hash of the string using sha256.

Step 3: Put that hash into a JSON object under the key "hash".  This will be the data payload for the JWT.
{ hash: 'kdjf20ijkl3josdkfj024ifjl2kfje' }

Step 4: Create JWT with your public key and data payload.  If you have a single public key for your organization,
you can embed it in the document itself and register it as your trusted public key in the trusted list.  If
you have multiple public keys for your organization, even a different key for every audit, you can put a
URL (jku) in the JWT where your public key can be found, and a "key id" (kid) indicating which key in the set 
of keys is the one for this document.

STOPPED HERE: to be continued.....

Step 5: 
