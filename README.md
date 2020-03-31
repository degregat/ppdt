## Similar Proposals
- [Varia, Canetti, Herzberg, Trachtenberg](https://www.linkedin.com/pulse/controlling-covid-through-cellphones-ari-trachtenberg)
- [Miers](https://twitter.com/secparam/status/1243369170621935616)
- [CoEpi and covid-watch](https://docs.google.com/document/d/1f65V3PI214-uYfZLUZtm55kdVwoazIMqGJrxcYNI4eg/edit) want to achieve interoperability

## Privacy Considerations
A classification of the tradeoffs of different approaches can be found in [[1]](#source-1).

## Ideal Functionality
A draft of [Anonymous Retroactive Broadcasts](https://gist.github.com/hdevalence/fefba3153b30e60537e84f7d2551b295). The protocol below (and most proposals above) does not implement the fetch phase, only single bit messages (confirmation of contact) are transmitted. (review wether other steps of IF are faithfully implemented in progress)

# Privacy Preserving Disease Tracking

### DISCLAIMER:
This protocol has not undergone thorough threatmodelling and review yet, but we are working on it. It is an open work in progress, since time is of the essence. Feedback is welcome.

## Problem Statement
We want to do privacy preserving contact tracing and notify users if they have come in contact with potentially infected people. This should happen in a way that is as privacy preserving as possible. We want to have the following properties:

- The users should be able to learn if they got in touch with infected parties, ideally only that.
- The server should not learn anything besides who is infected, ideally not even that.

### Acronyms
- BLE = Bluetooth Low Energy
- PID = Pseudonymous Identifier
- N = # of days of incubation period (+ some margin)
- DB = Database
- hash = should be a password hashing function e.g. a variant of Argon2

## Protocol Description
- Every participant generates a new random PID per timeslot (e.g. every 10 minutes).
- Each phone is running a BLE (or similar) beacon, broadcasting a hash of the current PID. If two devices come close to each other, they record each others hashed PIDs and save these with a timestamp and distance estimation, locally on their device.
- In case of a positive diagnosis for a participant, they submit the history of their PIDs from the last N days to a public DB. The PIDs of their contacts do not leave their device.
- Every participant regularly downloads the diffs from the DB, and does a local intersection of their recorded contact history and the hashed diffs.
- From the intersection (weighed by times and distances) each users device can estimate a risk and notify the user/recommend a test.
- In case of a positive test outcome/sufficiently high risk, the user publishes their PID history and self quarantines. In case of a negative outcome, they continue running the above protocol.

This protocol falls into the "public DB" category of [[1]](#source-1), but linkage by authorities is only possible in case of collusion with users.

### Possible extensions
- To exchange bandwidth for post-computation, instead of random PIDs we could use a HMAC on a counter, truncate the output for PIDs and publish the secret in case of infection. This would correlate all IDs though, leading to more leakage against colluding adversaries (which we don't assume at the moment).
- The (semi-honest) DB could hash the submitted PIDs, add them to a bloomfilter, and publish this to save bandwidth. Correct execution of the protocol on the server could be verified, by users checking wether their hashed PIDs are contained in the filter. In case of an error rate that is (much) higher than the natural estimated error rate of the bloom filter, the DB likely diverges from the protocol.
- During contact, if the BLE constraints permit a connection, a key exchange can be perfomed. Messages encrypted with the resulting key can be appended to the published IDs.
- If it seems to be neccessary to do (coarse) space-time bucketing for scalability, a PIR scheme could be used for fetching to hide the space-time traces from the server, since users might want to only query a subset of these buckets. To hide the space-time traces for submitting users, decorrelation from an anonymizer could be used.

### Size of DB
- Assuming 144*26 byte PIDs per day, N = 14 and 100.000 active cases, the DB will contain around 5 GB of data in total.

### Risk assessment
- Users could log distance and duration for each PID they see, to calculate risk on device after notification.

## Threatmodel
- Clients are assumed to be individually malicious, but not colluding at scale.
- The DB is assumed to be semi-honest.
- We provide only application layer de-correlation. The OS is assumed to be trustworthy, e.g. not recording the Bluetooth MACs and we do not deal with transport for submission and download.

## Possible Attacks (Application Layer)
### Linkage Attacks
If a user correlates individuals to broadcasts they receive, they will be able to determine their infection status. Simplest case: A user is only near to one person for N days.

Possible mitigations:
- Using the mixing technique from [[1]](#source-1). this would require the deployment of multiple, semi-honest mixing servers.
- [Private Join and Compute](https://github.com/google/private-join-and-compute), but malicious security would be needed (theoretical).

### Spam
Users could upload forged submissions to cause congestion.

Possible mitigations:
- https://privacypass.github.io/ grants anonymous credentials for the solution of a captcha. Requires additional infrastructure, compatibility with the threatmodel needs to still be evaluated.

### Partially Mitigated Attacks
- Impersonation: An attacker could upload the PIDs of users they meet. Is this sufficiently mitigated by broadcasting (Argon2) hashes of PIDs? Would a public rotating salt improve the situation? We do not want per client salts, to prevent correlation of PIDs. If this is deemed acceptable though, keyed schemes will have better tradeoffs.
- Local Tracing: PID rotation mitigates against non-colluding snoopers.

### Mitigated Attacks (Application Layer)
- Creating false positives: If a malicious user uploads random PIDs, the collision rate is low if the PID space is large enough.
- Creating false negatives: Is equivalent to non-participation.

## BLE
- BLE 4.2 can exchange 26 bytes without establishing a connection (31 bytes - 2 bytes company ID, 2 bytes fieldtype and 1 byte transmission power for ranging) (citation needed) [specs](https://www.silabs.com/community/wireless/bluetooth/knowledge-base.entry.html/2018/08/10/extended_advertising-aEID)
- BLE up to version 4.2 seems to be more widely supported [source from 2017](https://www.androidauthority.com/android-oreo-vs-android-nougat-bluetooth-5-794699/)
- BLE ranging seems to be accurate up to 4 meters (citation needed)
- Connectionless is preferable because of wider support (citation needed)
- Rotation of application layer and Bluetooth MAC should be harmonized to minimize leakage and prevent [address-carryover attacks](#source-2). MAC rotation is bounded by the [platform](#source-2).

### Other layers
- Anonymous submission and anonymous download can further increase user privacy
- Health authorities could give out anonymous credentials for submission with test results if it seems feasible and necessary

## Privacy and Incentives
- Only the history of PIDs of participants who were tested positive is published. Since this history is not correlated to location, and correlation to contact history happens only on users devices, neiter location, nor contact history is leaked to the server or non-contacts. This, together with voluntary participation, can increase buy-in from the population, leading to faster response time for testing larger groups. Since the health authorities will administer the tests, local statistics will also become more accurate.
- Since people get incentivized to get tested before they become symptomatic, spread can be reduced.
- Since the PIDs get rotated, local tracking/correlation by other potentially malicious participants gets impeded.
- This kind of soft, opt-in intervention is probably most useful for the long tail to monitor resurgences. To improve monitoring, the app could walk users through self diagnosis.

## Open Questions
- Does a (weighted) intersection method exist, which hides the elements of the intersection, against a malicious attacker?
- Which potential malicious user behavior did we miss?
- Can we achieve robustness against colluding clients? (e.g. regarding location tracing).
- Do we need rate limiting to prevent spam on the DB? Can we reduce false positives from forged submissions futher this way?
- Do we gain anything from anonymous submission of PIDs? (All at once, subsets, individual PIDs per circuit or on a mixnet)
- Further analysis of privacy leakage from plaintext DB
- BLE has a range of up to 10 Meters, can we get useful distance information and log it for each PID of a contact?

## Sources
<a name="source-1">[1]</a> [Contact Tracing Mobile Apps for COVID-19](https://arxiv.org/pdf/2003.11511.pdf)
<a name="source-2">[2]</a> [Tracking Anonymized Bluetooth Devices](https://petsymposium.org/2019/files/papers/issue3/popets-2019-0036.pdf)
