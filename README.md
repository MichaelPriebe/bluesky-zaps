# Bluesky Zaps

### ⚠️ This is just to brainstorm, do not implement. ⚠️

Alice wants to zap Bob for their amazing post on [Bluesky](https://bsky.app/).

Alice [looks up](https://atproto.com/lexicons/com-atproto-repo#comatprotorepogetrecord) Bob's lud16 via their `org.bitcoin.lightning.profile` record.

Schema:

```json
{
  "lexicon": 1,
  "id": "org.bitcoin.lightning.profile",
  "defs": {
    "main": {
      "type": "record",
      "key": "literal:self",
      "record": {
        "type": "object",
        "properties": {
          "lud16": {
            "type": "string",
            "description": "The LUD-16 lightning address."
          }
        }
      }
    }
  }
}
```

Example:

```json
{
  "lud16": "bob@bitcoin.org"
}
```

Alice's client makes a request to the LNURL service according to [LUD-06](https://github.com/lnurl/luds/blob/luds/06.md).

⚠️ Alice may see `allowBluesky=true` is missing from the response. This would indicate that Bob's LNURL service is not connected to Bluesky, in which their client might warn them that Bob may not publish their zap. Alice's client may have an option to proceed anyway with the knowledge that the zap may not be published.

If `allowBluesky=true` is present, Alice's client makes a request to the `callback` with the following query parameters:

1. `amount`: The amount in millisatoshi.
2. `bluesky`: The URI encoded json details about this zap:
   1. `uri`: The URI of the subject post.
   2. `cid`: The CID of the subject post.
   3. `identifier`: Optional at-identifier of who's paying. May be omitted if payer wishes to remain anonymous.

Alice's client displays the `pr` payment request for them to pay with their chosen lightning wallet.

Once Alice pays the payment request, Bob's LNURL service will [publish a record](https://atproto.com/lexicons/com-atproto-repo#comatprotorepocreaterecord) to their `org.bitcoin.lightning.zap` collection.

Schema:

```json
{
  "lexicon": 1,
  "id": "org.bitcoin.lightning.zap",
  "defs": {
    "main": {
      "type": "record",
      "key": "tid",
      "record": {
        "type": "object",
        "required": ["subject", "createdAt", "amount"],
        "properties": {
          "subject": {
            "type": "ref",
            "ref": "com.atproto.repo.strongRef"
          },
          "createdAt": {
            "type": "string",
            "format": "datetime"
          },
          "amount": {
            "type": "integer",
            "description": "The amount paid in millisatoshi."
          },
          "identifier": {
            "type": "string",
            "format": "at-identifier",
            "description": "The handle or did of the payer. May be omitted if the payer wishes to remain anonymous."
          }
        }
      }
    }
  }
}
```

Example:

```json
{
  "subject": {
    "uri": "<uri>",
    "cid": "<cid>"
  },
  "createdAt": "2009-01-03T13:15:00.000Z",
  "amount": "21000",
  "identifier": "alice.bitcoin.org"
}
```

⚠️ Because it is impossible as a third party to verify a lightning transaction even happened, no details about the transaction are included. Assume the author publishes zap transactions in good faith, for accounting and record keeping purposes, but these records cannot be used as fact or legal evidence.
