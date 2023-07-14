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
 * contain an element with `not-yet-defined` attribute set to "true".


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
