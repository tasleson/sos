# SOS (Save Our Stuff)

#### Synopsis
Data interity file format specification and reference implementation(s)

#### Background

Users have the expectation that when they save a file on their computer that they can retrieve it safely intact later.  Whether it be 1 hour or 10 years from now.  This is expected of computing devices in general.  Historically, the industry has done well in this pursuit, but there is evidence suggesting that things are not as good as one would hope[[1]](http://indico.cern.ch/event/13797/session/0/contribution/3/attachments/115080/163419/Data_integrity_v3.pdf).  Opinions vary on the need, but some newer file systems like ZFS, BTRFS, ReFS, and bcachefs implement checksums, so that the user can detect when files are no longer as they once were through periodic scrubs or when accessing the file.  This is valuable as it allows the user to retrieve a good copy from backup and also be aware of the possibility of hardware or software issues which are causing the corruption.  However, for some users running scrubbing tools is quite costly [[1]](http://indico.cern.ch/event/13797/session/0/contribution/3/attachments/115080/163419/Data_integrity_v3.pdf).

The argument can be made that this problem can be better solved at the application level.  That each and every application add the requisite checks to their software to ensure integrity.  This however requires a separate tool to validate each and every file type if the user wants to scrub.  This quickly becomes intractable.  We want to shield the users from the associated data loss of unreliable hardware and/or software without making it overly difficult.

#### Proposal

Originally my stance was this needs to be in the file system.  However, after considering this some more I believe the best solution is a file format specification and reference implementation library which achieves the following:

* Single file format which encapsulates and protects the application data
* Strong checksums to ensure data is intact
* Erasure coding support with configurable amounts of protection based on the end users goals


##### Pros:
* Allows a single utility to verify and correct any file that adheres to the file specification
* Reduces the need for scrubbing as the file can correct itself
* Protection is file system and platform agnostic
* File is protected when transmitted across the network
* File is protected when on archival media which typically does not get scrubbed

##### Cons:
* Requires every application be modified to support, or maybe abuse linker preloading like other utilies do to add features to existing applications [[2]](http://static.usenix.org/event/usenix05/tech/freenix/full_papers/eriksen/eriksen.pdf).
* Consumes more resources for the application (CPU/Memory/Storage)
* Unable to validate the data payload, it can only validate that data checksum validates (common issue for any generic solution)

#### Implementation details

Much needs to be discussed with regards to this, but ultimately a file format which allows any part of a file to be omitted and/or corrupted and subsequently recovered and validated is the objective.
