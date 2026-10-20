# Officetul Documents

The Documents app inside Officetul: how a company's paper and digital records are captured, identified, versioned and connected to its business records.

## Language

### Capture

**Inbox item**:
Something captured by upload, email or phone that nobody has filed yet.
_Avoid_: Pending upload, unprocessed file

**Capture record**:
The facts about how an Inbox item arrived: who, when, by which channel, and from which sender.
_Avoid_: Upload log, intake metadata

### Documents and versions

**Document**:
A company's business record, such as "Lease with ABC Realty, Unit 3B". It holds one or more Versions.
_Avoid_: File, attachment, record

**Document type**:
The kind of Document, such as Invoice (purchase), BIR 2307 or Contract. It decides which details are asked for.
_Avoid_: Category, doc class

**Folder**:
The place a Document lives. It decides who may see and manage it.
_Avoid_: Cabinet, workspace, directory

**Version**:
One numbered state of a Document's content (v1, v2, …). It never changes after it is created.
_Avoid_: Revision, edition, copy

**Original file**:
The exact file received for a Version, kept unchanged.
_Avoid_: Upload, source file

**Derived file**:
Something made from an Original file, such as OCR text or a preview. It is never treated as the record itself.
_Avoid_: Rendition, OCR file

**Official version**:
The single Version of a Document that the company has chosen to rely on. A Document has at most one, and may have none.
_Avoid_: Latest version, current version, approved version, master copy

**Draft**:
A Version that has not been made the Official version.
_Avoid_: Unpublished, pending

**Superseded**:
A Version that used to be the Official version and was replaced by another.
_Avoid_: Old version, obsolete

**Withdrawn**:
A Version taken out of official status because it was marked by mistake or is no longer valid.
_Avoid_: Rejected, cancelled, voided

**Revision label**:
The company's own name for a Version, such as "Rev C" or "Signed".
_Avoid_: Version number

**Issue status**:
The company's own status for a Document type, such as "For Construction". It is separate from whether a Version is Official.
_Avoid_: State, version status

**Paper original**:
The physical document a Version was scanned from, with where it is kept and who keeps it.
_Avoid_: Hard copy, source paper

### Connections

**Link**:
A labelled connection between a Document and an Officetul record, such as "receipt for" Expense #881.
_Avoid_: Attachment, association

**Related document**:
A labelled connection between two Documents, such as an amendment that "amends" a contract.
_Avoid_: Linked document, child document

### Housekeeping

**Trash**:
Where deleted Documents and Drafts wait, restorable, until a company admin deletes them permanently.
_Avoid_: Recycle bin, archive

### Access

**Access level**:
What a person may do in a Folder: View, Contribute or Manage. Each level includes the one before it.
_Avoid_: Permission level, role, share level

**Restricted folder**:
A Folder that ignores the access of the Folder above it and has its own list of who may see it.
_Avoid_: Private folder, locked folder

**Confidential folder**:
A Restricted folder whose contents even company admins cannot see unless they add themselves, which is recorded and announced.
_Avoid_: Secret folder, hidden folder

**Support access**:
Time-limited, read-only access to a company's Documents by Officetul staff, approved by that company's admin.
_Avoid_: Impersonation, log in as user, admin override

### Records of what happened

**Activity log**:
The company's record of what happened to its Documents and Folders and who did it. The company can see and export it.
_Avoid_: Audit trail, history, audit log

**Security log**:
Officetul's own record of logins, two-factor checks, refused access and Support access.
_Avoid_: Activity log, system log

**Log digest**:
A daily fingerprint of a company's Activity log, kept where nobody can change or delete it, so later tampering would show.
_Avoid_: Backup, checksum file

**Fixity check**:
Re-reading a stored Original file to confirm it still matches the fingerprint taken when it arrived.
_Avoid_: Integrity scan, virus scan
