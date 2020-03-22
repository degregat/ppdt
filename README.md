# Privacy Preserving Disease Tracking

### DISCLAIMER:
This protocol has not undergone thorough threatmodelling and review yet, but we are working on it. It is an open work in progress, since time is of the essence. Feedback is welcome.

### Acronyms
- BLE = Bluetooth Low Energy
- PSI = Private Set Intersection
- PID = Pseudonymous Identifier
- N = # of days of incubation period (+ some margin)
- DB = Database

## Protocol Description
- Every participant generates a new random PID per timeslot (e.g. every 10 minutes).
- Each phone is running a BLE (or similar) beacon, broadcasting the current PID. If two devices come close to each other, they record each others PIDs and save these with a timestamp, locally on their device.
- In case of a positive diagnosis for a participant, they submit the history of their PIDs from the last N days to a public DB. The PIDs of their contacts do not leave their device.
- Every participant regularly downloads the diffs from the DB and does a local intersection of their recorded history and the diffs.
- From the size of the intersection each users device can calculate a risk and recommend the user to get tested.
- In case of a positive test outcome the user publishes their PID history and self quarantines. In case of a negative outcome, they continue running the above protocol.

## Threatmodel
- Clients are assumed to be individually malicious, but not colluding at scale.
- The DB is assumed to be semi-honest.
- We provide only application layer de-correlation. The OS is assumed to be trustworthy, e.g. not recording the Bluetooth MACs and we do not deal with transport for submission and download.

### Malicious Clients
Possible ways for a malicious client to misbehave would be to forge/omit submissions to produce false positives and false negatives. If the PIDs are sufficiently long, the collision rate should low enough to produce few false positives in case of forged submission. Since this is an opt-in protocol, false negatives are identical to non-participation. 

A client could correlate PIDs to other users on sidechannels, to later look up which people are positive. This might be mitigated with a construction that only outputs the size (or non-emptiness) of an intersection, but not its elements.

### Other Layers
- Bluetooth MAC rotation on the OS level could provide further de-correlation
- Anonymous submission and anonymous download can further increase user privacy 

## Privacy and Incentives
- Only the history of PIDs of participants who were tested positive is published. Since this history is not correlated to location, and correlation to contact history happens only on users devices, neiter location, nor contact history is leaked to the server or non-contacts. This, together with voluntary participation, can increase buy-in from the population, leading to faster response time for testing larger groups. Since the health authorities will administer the tests, local statistics will also become more accurate.
- Since people get incentivized to get tested before they become symptomatic, spread can be reduced.
- Since the PIDs get rotated, local tracking/correlation by other potentially malicious participants gets impeded.

## Open Questions
- Does a PSI with intersection size but not element disclosure exist?
- Which potential malicious user behavior did we miss?
- Can we achieve robustness against colluding clients? (e.g. regarding location tracing)
- Do we need rate limiting to prevent spam on the DB? Can we reduce false positives from forged submissions futher this way?
- Do we gain anything from anonymous submission of PIDs? (All at once, subsets, individual PIDs per circuit or on a mixnet)
- Further analysis of privacy leakage from plaintext DB
- BLE has a range of up to 10 Meters, can we get useful distance information and log it for each PID of a contact?

