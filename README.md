# Privacy Preserving Disease Tracking

### DISCLAIMER:
This protocol has not undergone thorough threatmodelling and review yet, but we are working on it. It is an open work in progress, since time is of the essence. Feedback is welcome.

### Acronyms
- BLE = Bluetooth Low Energy
- PSI = Private Set Intersection
- PID = Pseudonymous Identifier
- N = # of days of incubation period
- DB = Database

## Protocol Description
- Every participant generates a new random PID per timeslot (e.g. every 10 minutes).
- Each phone is running a BLE (or similar) beacon, broadcasting the current PID. If two devices come close to each other, they record each others PIDs and save these with a timestamp, locally on their device. This provides decorrelation of the application layer.
- In case of a positive diagnosis for a participant, they publish the history of their PIDs. The PIDs of their contacts stay confidential.
- Every day, every participant runs a PSI with the collected PIDs of their contacts from the last N days against the DB. (One DB per day, from truncated timestamps of the PID histories.)
- From the size of the intersection each users device can calculate a risk locally and recommend the user to get tested.
- In case of a positive test outcome the user publishes their data and self quarantines. In case of a negative outcome, they continue running the above protocol.

## Privacy and Incentives
- Only the history of PIDs of participants who were tested positive is published. Since this history is not correlated to location, and not publically correlated to contact history, very little privacy leakage occurs. This, together with voluntary participation, can increase buy-in from the population, leading to faster response time for testing larger groups. Since the health authorities will administer the tests, local statistics will also become more accurate.
- Since people get incentivized to get tested before they become symptomatic, spread can be reduced.
- Since the PIDs get rotated, local tracking/correlation by other potentially malicious participants gets impeded.

## PSI 
The primitive that seems to be the most advanced at the moment, regarding performance and security against malicous users is
[Mobile Private Contact Discovery at Scale](https://eprint.iacr.org/2019/517.pdf). ([Conference Version](https://www.usenix.org/system/files/sec19-kales.pdf))

There exists an [implementation for android](https://github.com/contact-discovery).

### Scalability
- It takes 2.93 seconds on WiFi / 6.53 seconds on LTE and 6.07 MiB of data for
1024 client contacts to be checked against a DB of 2^28 (268.435.456) entries.

- With 144 PIDs/day or 2016/14 days at 6 PIDs/Hour and 100.000 active cases
we get 201.600.000 DB entries.

- 1024/6 would equal ~170 contact hours per day, or ~7 fulltime contacts.

These calculations are for 6 PIDs/hour.

## Alternative Approach for Untrusted Database
If it is acceptable to publish PID history, we could omit the PSI scheme. In this case, PIDs would be published in plaintext and clients would download the diffs, to do set intersection on their own devices. 
Since it is preferable not having to trust the DB, we should aim to achieve this.

## Threatmodel
- Clients are assumed to be individually malicious, but not colluding at scale.
- DB is assumed to be semi-honest.
- We provide only application layer de-correlation. The OS is assumed to be trustworthy, and not recording the Bluetooth MACs.

### Considerations
- BLE has a range of up to 10 Meters

## TODO
- analyze privacy leakage for plaintext DB
- achieve robustness against colluding clients
- refine threatmodel
