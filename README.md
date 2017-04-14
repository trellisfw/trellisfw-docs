# fpad-docs
Documentation for the fPAD framework.

This is a work-in-progress.  Documentation will be added as various components are finalized.

## Core data models
You can find definitions of the core data models in the `oada-formats` repository under "fpad".
This describes JSON documents for representing a food safety audit, corrective actions, and
food safety certificates.  

## fPAD Document Integrity Signatures

[https://fpad.github.io/fpad-audit-sign-verify](Demo application to sign and verify documents is here.)

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

fPAD uses the [JSON Web Token standard](https://jwt.io) to represent the signature.  The process
for generating the signature is relatively straighforward, and example code which does this can 
be found in [https://github.com/fpad/fpad-audit-sign-verify](https://github.com/fpad/fpad-audit-sign-verify).

# To create a signature:
---------------------------------------------------------
```
Step 1: turn the JSON object into a string.  Since everybody must create the same string on any platform,
some additional caveats apply in order to achieve consistent hashing: 
* The key names in the string must be in lexicographic (alphabetical) order.
* All key names must be quoted with double quotes.
* All whitespace must be removed, except whitespace within quoted strings and key names.
* Serialization must be UTF-8.
* Because numeric representations differ on different platforms, all numbers in the document must be 
  represented as strings instead of Javascript numbers.

Step 2: Compute a hash of the string using sha256.

Step 3: Put that hash into a JSON object under the key "hash".  This will be the data payload for the JWT.
{ hash: 'kdjf20ijkl3josdkfj024ifjl2kfje' }

Step 4: Create JWT with your public key and data payload.  If you have a single public key for your organization,
you can embed it in the document itself and register it as your trusted public key in the trusted list.  If
you have multiple public keys for your organization, even a different key for every audit, you can put a
URL (jku) in the JWT where your public key can be found, and a "key id" (kid) indicating which key in the set 
of keys is the one for this document.  To be trusted, you must then add your jku URL or its prefix to the
trusted list.  The JWT must contain either the jwk or the jku/kid pair in the header.

Step 5: Put the JWT into the original document.  If the document does not have a signatures key at the
top level, create one with an empty array as value and put the new JWT into the array.  If the document
already has a signatures key, push the new JWT onto the end of the signatures array.
```


# To Verify a Signature:
---------------------------------------------------------
Note: this process is the reverse of creating a signature.
```
Step 1: read JSON document and remove latest signature.  Find the most recent signature in the array under the 
signatures key: the one at the highest index in the array.  Remove it from the array.  If after removal
the array is empty, remove the signatures key entirely.  The JSON object now looks as it did before that signature 
was applied.

Step 2: Serialize resulting document according to rules for creating signature above.  

Step 3: Compute a hash of the resulting string using sha256.  Save hash for comparison in a later step.  This 
represents the hash which should have been created if the document hasn't been changed.

Step 4: Decode the JWT which was the signature removed in Step 1.  Look in the JWT header for either a jwk
or a jku/kid pair.  If there are both a jwk and a jku/kid pair, use only the jwk for verification and ignore
the jku/kid pair.  

Step 5: Verify that the jwt or jku is on the trusted list.  Retrieve the trusted list of signers,
and either check for the jwt in the list, or the jku in the list.  The trusted list is allowed to contain
URL prefixes in addition to complete URL's, so you must also check if the jku matches any wildcarded
jku in the trusted list.  i.e. https://example.com/123/456 would match an entry in the trusted list of 
"https://example.com/123/*".

Step 6: If you have a jku and it is on the trusted list, make an HTTP GET request to the jku URL,
and retrieve the particular public key at the given key in the response.  This becomes the jwk for the 
this JWT.  If you already have a jwk instead of a jku, you can skip this step.

Step 7: Verify the JWT according to the JWT standard using the public key you now have.

Step 8: If the JWT verifies, retrieve the "hash" key from its payload.  Compare the string
at the "hash" key with the string you computed in step 3.  If they match, the document
is valid.  If they do not match, or if the JWT does not validate, then the document is invalid.
