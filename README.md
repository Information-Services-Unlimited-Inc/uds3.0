# UDS 3.0: Claims as JSON for the modern system

The Uniform Data Standard (v2.4 April 2019) was first created in 1995 as a simple data format encompassing the core aspects of a claim and its parts. While it has served the insurance guaranty fund community well for decades and continues to represent thousands of claims annually, the fixed-length text format presents practical challenges in modern development environments. UDS 3.0 addresses these pain points by moving to a JSON-based specification that is easier to work with, more extensible, and better suited for integration with contemporary systems.

## Problems with UDS 2.0

| Problem                         | Impact                                                                                                                                                                                                                                                                                                                                                                                                                   |
|---------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Batch Limit of 999**          | Limits are easily reached during big insolvencies increasing the complexity of reporting.                                                                                                                                                                                                                                                                                                                                |
| **ZIP files are temperamental** | Delivery of large ZIP files is a time-consuming and temperamental process where the file could be corrupted or incomplete at various stages during delivery, and remediation involves redoing the process.                                                                                                                                                                                                               |
| **Data outside specification**  | Any data outside the specification must be delivered as an Excel file that should otherwise just be included in the spec.                                                                                                                                                                                                                                                                                                |
| **UDS expertise**               | A UDS expert may be necessary to assist in UDS creation because the UDS specification can differ significantly from the original format of the insolvent company data.                                                                                                                                                                                                                                                   |
| **Limited character support**   | UDS requires the ASCII character set, which only supports characters familiar in the English language                                                                                                                                                                                                                                                                                                                    |
| **File Path Separator**         | UDS requires the use of the Microsoft Windows-specific file path separator '\\' in all I Record paths, despite '/' being more widely supported on all operating systems.                                                                                                                                                                                                                                                 |
| **Key Duplication**             | UDS requires that unique key information be copied from the A Record to all other record types to create the data relationship, which pads other record types' data files with duplicate information                                                                                                                                                                                                                     |
| **Metadata Truncation**         | UDS does not fully utilize the metadata properties visible during claims handling, which can be helpful, such as document tagging, distinguishing between system notes and user notes in the source data, and payment statuses like paid/unpaid/canceled in the source data. Making use of these properties currently depends on including custom text in the description/body/memo instead of having a dedicated field. |
| **Volume Fatigue**              | Importing a UDS file relies on communications outside the file itself, and the file names blend together increasing the likelihood of mistakes.                                                                                                                                                                                                                                                                          |
| **Hard to read**                | Non-technical persons are unable to read or edit UDS files without specialized software.                                                                                                                                                                                                                                                                                                                                 |
| **Non-native number format**    | To represent numbers UDS has an inferred decimal point and sometimes optional sign. This custom format can be confusing to read for the first time.                                                                                                                                                                                                                                                                      |

## Presenting UDS 3.0

UDS 3.0 represents a fundamental shift in how insurance claim data is structured and exchanged. By adopting JSON as the data format, the specification brings native support for hierarchical data structures, eliminates the complexity of managing multiple file types, and provides built-in validation through JSON Schema. Beyond the format change, UDS 3.0 expands the data model to preserve more fields from source systems and introduces flexible options for document delivery that decouple metadata from the physical transfer of images.

### 1. JavaScript Object Notation (JSON)

- **Modern data format**: JSON is the most popular data format when integrating with web services. As the world becomes more tightly connected, sharing the same data format prepares UDS 3.0 for the future.
- **Tooling ecosystem**: JSON has endless free and open source software that can read it, display it, and make editing easy for even non-technical users.
- **Built-in validation**: JSON has a 'schema' file that can be publicly hosted and whose link can be embedded directly into the data file so third party tools can recognize UDS 3.0 and provide feedback if changes to the file do not meet the public specification.
- **Natural hierarchies**: JSON has data hierarchy where nouns like "Policy" and "Claim" can be described by its parts in one single object (One Policy, List of Claims, List of Documents in Claims).
- **Simplified file management**: Combining A, F, G, I, M, and B Records into one Policy object removes the confusion of having to load an A Record first, then finding the related child record files of the same batch. Importing a batch just involves selecting ONE file, and archiving it for later is easier than keeping 6 potential files together per batch number.

### 2. More fields that preserve original state of claim data

UDS 2.0 is missing certain fields like Claim Status under the assumption the original value doesn't matter – they'll be open once they get to the Fund – or this information is shared via spreadsheets outside of UDS. UDS 3.0 aims to expose and preserve these fields to give each organization the freedom to choose if the field matters or not.

**Expected new fields:**
- Policy, Claim, Payment Status
- Policy, Claim, Claimant level for Notes and Documents
- Note and Payment Type
- Suit Contact Information
- Insured and Claimant Middle Name
- Multiple Addresses, Emails, Phone Numbers
- Supplementary messages related to batch as whole

### 3. Flexibility in how to collect Images besides from a ZIP

UDS 3.0 offers a way to disconnect the metadata about documents from the source document import process by letting organizations choose how they want to retrieve their documents (Locally via a ZIP, or Remotely via SFTP or HTTPS). By not delivering the documents right away, organizations aren't bogged down by a full warehouse if they only have space for a mailbox of data. Software can be adjusted to download Images by your criteria (priority, coverage, size) completely eliminating the stress of batch Image processing.

**Example document reference formats:**

```json
[
    {   // HTTPS protocol using SAS token
        "DocumentName": "test.txt",
        "Description": "Some file",
        "DateCreated": "2025-04-28T16:34:56+00:00", // ISO8601
        "DateLastModified": "2025-04-28T16:34:56+00:00",// ISO8601
        "Size": 239024903, // In bytes
        "Hash": "cf2238f43a01c4bd4651f00a9b7e255fc827b6b22dda277e38f392e7660ef4f5",
        "Algorithm": "SHA256",
        "Source": "https://suds.blob.core.windows.net/testgf/test.txt?{SAS_TOKEN…}",
        "Type": "text/plain" // https://www.iana.org/assignments/media-types/media-types.xhtml
    },
    {   // SFTP protocol using secret credentials
        "DocumentName": "test.txt",
        "Description": "Some file",
        "DateCreated": "2025-04-28T16:34:56+00:00", // ISO8601
        "DateLastModified": "2025-04-28T16:34:56+00:00",// ISO8601
        "Size": 239024903, // In bytes
        "Hash": "cf2238f43a01c4bd4651f00a9b7e255fc827b6b22dda277e38f392e7660ef4f5",
        "Algorithm": "SHA256",
        "Source": "sftp://suds.blob.core.windows.net/testgf/test.txt",
        "Type": "text/plain" // https://www.iana.org/assignments/media-types/media-types.xhtml
    },
    {   // FTPS protocol using secret credentials
        "DocumentName": "test.txt",
        "Description": "Some file",
        "DateCreated": "2025-04-28T16:34:56+00:00", // ISO8601
        "DateLastModified": "2025-04-28T16:34:56+00:00",// ISO8601
        "Size": 239024903, // In bytes
        "Hash": "cf2238f43a01c4bd4651f00a9b7e255fc827b6b22dda277e38f392e7660ef4f5",
        "Algorithm": "SHA256",
        "Source": "ftps://suds.blob.core.windows.net/testgf/test.txt",
        "Type": "text/plain" // https://www.iana.org/assignments/media-types/media-types.xhtml
    },
    {   // Local file
        "DocumentName": "test.txt",
        "Description": "Some file",
        "DateCreated": "2025-04-28T16:34:56+00:00", // ISO8601
        "DateLastModified": "2025-04-28T16:34:56+00:00",// ISO8601
        "Size": 239024903, // In bytes
        "Hash": "cf2238f43a01c4bd4651f00a9b7e255fc827b6b22dda277e38f392e7660ef4f5",
        "Algorithm": "SHA256",
        "Source": "file://some_folder/test.txt",
        "Type": "text/plain" // https://www.iana.org/assignments/media-types/media-types.xhtml
    }
]
```

## Migration Path

UDS 3.0 represents a significant change to the data file format used throughout the insurance guaranty fund industry. While the benefits are substantial, adoption will be gradual—likely occurring over several years to a decade as organizations evaluate, test, and implement the new standard. The following outlines the planned migration strategy and key challenges to address.

### Phased Adoption Strategy

1. **Pipeline Development**: The UDS Data Mapper will provide dedicated pipelines for creating and sending UDS 3.0 files, allowing both Receivers and Guaranty Funds to explore how the format works in practice.

2. **Feedback and Refinement**: As organizations test the UDS 3.0 pipeline, NCIGF/GSI will collect feedback and make iterative improvements to the schema and implementation process based on real-world usage.

3. **Gradual Transition**: Once the pipeline has proven stable over an extended period, new receivers and insolvent companies will be encouraged to adopt UDS 3.0 as their primary format.

4. **Backwards Compatibility**: A transparent, automatic conversion process will transform UDS 3.0 files into backwards-compatible UDS 2.0 format for Guaranty Funds that have not yet migrated to the new standard.

### Migration Challenges

**Full Object Snapshots**: Unlike UDS 2.0's incremental updates, UDS 3.0 sends the entire policy object snapshot with each batch—even when only minor changes (such as adding a few notes) have occurred. This approach requires systems to intelligently identify and handle duplicate data from previously imported batches. Without indicators in the data file to distinguish new from existing Documents, Notes, and other child objects, downstream systems may struggle with deduplication.

**Document Delivery Changes**: The flexible document reference model in UDS 3.0 (supporting HTTPS, SFTP, FTPS, and local file paths) fundamentally differs from UDS 2.0's ZIP-based approach. Organizations will need to adjust their file ingestion processes, including when and how documents are retrieved, stored, and associated with claim data.

## How to Contribute

Your feedback is essential to making UDS 3.0 a practical and effective standard for the insurance guaranty fund community. Whether you're a developer building claim systems to support UDS 3.0 or a technical professional who regularly imports and manages UDS 2.0 files, your real-world insights will help shape the specification.

### Ways to Provide Feedback

**GitHub Issues** (Preferred): Open an issue in this repository to discuss problems, propose enhancements, or ask questions about the specification. Public issues encourage collaboration and ensure others facing similar challenges can benefit from the discussion.

**Pull Requests** (Preferred): If you have a specific change to propose for the schema, fork this repository and submit a pull request with your modifications. This allows for detailed technical review and discussion of the implementation. See [GitHub's guide on creating a pull request from a fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork) if you're new to this workflow.

**Direct Email**: You may also reach out directly to Nate Jennings, Software Engineering Manager at NCIGF, at [njennings@ncigf.org](mailto:njennings@ncigf.org). However, we strongly encourage using GitHub issues or pull requests when possible to foster public participation and collaborative problem-solving.

### Submitting Quality Feedback

When opening an issue or pull request, please include:
- **Detailed problem description**: What specific challenge or limitation are you encountering?
- **Proposed solution**: How would you address the problem? What changes to the schema or process do you suggest?
- **Rationale**: Why is this change important? How does it improve UDS 3.0?

Providing sufficient context helps reviewers understand your thought process and evaluate the proposed solution effectively.