Interinstitutional Agreements Approval API
==========================================

* [What is the status of this document?][statuses]
* [See the index of all other EWP Specifications][develhub]


Summary
-------

This document describes the **Interinstitutional Agreements Approval API**.
This API allows HEIs to approve agreements sent by their partners
in the [Interinstitutional Agreements API][iias-api].

**Clients and servers MUST NOT use IIA Approval API for agreements
that:**
 * are not properly mapped (do not provide partner agreement ID),
 * contain an element with `not-yet-defined` attribute set to "true",
 * are being served using IIA API version 6 (IIA Approval version 1 MUST be used for those agreements).


Upgrade to v2 steps
-------------------

Both IIA API v7 and IIA Approval API v2 have been released and must be used together.

Both APIs introduced new IIA hash calculation algorithm. The old hash used in IIA v6 (`conditions-hash` element) will not be used anymore.

A new hash has to be calculated for IIA approvals. The new element is now named iia-hash:
https://github.com/erasmus-without-paper/ewp-specs-api-iias/blob/v7.0.0/endpoints/get-response.xsd#L242-L249

A snapshot of the approved v6 IIA is needed to be able to upgrade to IIA v7 and Approval v2.
Such snapshot is stored as an IIA v6 GET API response XML of the approved agreement.

To calculate the new hash an XSLT v6
(https://github.com/erasmus-without-paper/ewp-specs-api-iias/blob/stable-v7/resources/xsltKit/transform_version_6.xsl)
has to be used to transform the stored snapshot to a string that will be hashed
(see: https://github.com/erasmus-without-paper/ewp-specs-api-iias/tree/stable-v7/resources/xsltKit).

For example for a stored approved IIA:
https://github.com/erasmus-without-paper/ewp-specs-api-iias/blob/stable-v7/resources/xsltKit/get-response-v6.xml

An IIA Approval v1 would contain:
```
<conditions-hash>f2b01c46c8b87b895597ecc40122ff377da8df16bebfa9c0c8020fb2207a50ba</conditions-hash>
```

XSLT v6 used on that IIA GET v6 response would produce:
```
<?xml version="1.0" encoding="UTF-8"?>
<iia>
   <iia-id>0f7a5682-faf7-49a7-9cc7-ec486c49a281</iia-id>
   <text-to-hash>_0f7a5682-faf7-49a7-9cc7-ec486c49a281__1954991__uw.edu.pl__140__hibo.no__031__Social and behavioural sciences__5__false__7__8__2014/2015__2020/2021__uw.edu.pl__140__hibo.no__2__en__C1__0314__8__2016/2017__2017/2018_</text-to-hash>
</iia>
```

The `text-to-hash` element value when hashed using `SHA-256` would produce `7bdd07b3a5088cb85da17326afa4d0514521b53ee6fc78cf7025a9ee6f5b11ab`.

That hash would then be inserted into an IIA Approval v2:
```
<iia-hash>7bdd07b3a5088cb85da17326afa4d0514521b53ee6fc78cf7025a9ee</iia-hash>
```

And should match the IIA v7 hash after partner upgrades his IIA.


Request method
--------------

 * Requests MUST be made with either HTTP GET or HTTP POST method. Servers MUST
   support both these methods. Servers SHOULD reject all other request methods.

 * Clients are advised to use POST when passing large number of parameters
   (servers MAY set a limit on their GET query string length).


Request parameters
------------------

Parameters MUST be provided in the regular `application/x-www-form-urlencoded`
format.


### `iia_id` (repeatable, required)

A list of IIA identifiers **as assigned by the calling partner**, no more than
`<max-iia-ids>`.

HEI covered by the caller and HEI covered by the server
MUST be the partners of all the referenced IIAs.

This parameter is *repeatable*, so the request MAY contain multiple occurrences
of it. The server is REQUIRED to process all of them.

Server implementers provide their own chosen value of `<max-iia-ids>`via their
[manifest-entry.xsd](manifest-entry.xsd). Clients SHOULD parse these values
(or assume they're equal to `1`).


Security
--------

This version of this API uses [standard EWP Authentication and Security, Version 2][sec-v2].
Server implementers choose which security methods they support by declaring them
in their Manifest API entry.


Handling of invalid parameters
------------------------------

 * General [error handling rules][error-handling] apply.

 * Values of `iia_id` that are not known by the server as covered HEI agreements
   or not approved (yet or at all) MUST be **ignored**.
   Servers MUST return a valid (HTTP 200) XML response in such cases, but the
   response will simply not contain any information on these missing entities.
   If all values are unknown, servers MUST respond with an empty `<response>`
   element. This requirement is true even when `<max-iia-ids>` is set to `1`.


Response
--------

Servers MUST respond with a valid XML document described by the
[response.xsd](response.xsd) schema. See the schema annotations for further
information.


Examples
--------

An example scenario of exchanging and accepting agreements is available
in the [Interinstitutional Agreements API][iias-api] repository.


[develhub]: http://developers.erasmuswithoutpaper.eu/
[error-handling]: https://github.com/erasmus-without-paper/ewp-specs-architecture#error-handling
[iias-api]: https://github.com/erasmus-without-paper/ewp-specs-api-iias
[statuses]: https://github.com/erasmus-without-paper/ewp-specs-management#statuses
[sec-v2]: https://github.com/erasmus-without-paper/ewp-specs-sec-intro/tree/stable-v2
