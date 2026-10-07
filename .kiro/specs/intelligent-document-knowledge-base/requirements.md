# Requirements Document

## Introduction

GovArchive is a Government Policy & Document Portal that provides deterministic archiving and search over official documents such as policies, SOPs, circulars, guidelines, reports, and meeting minutes. This specification scopes a **4-hour hackathon prototype** and is deliberately constrained to what can be demonstrated in that window.

The product reflects a major pivot away from a RAG/LLM answer-generation approach. The new direction is a **deterministic, zero-hallucination** portal: search results are produced exclusively by client-side keyword and substring matching against a pre-populated mock dataset. The portal never fabricates answers. Every result shown traces back to literal text that exists in a stored document, and matched keywords are highlighted so the user can visually verify the match.

Design and scope decisions governing this prototype:

- **Frontend-first prototype.** The implementation target is Next.js/React with Tailwind CSS, Lucide React icons, and Shadcn UI. No real backend is required for the demo. All document data is a pre-populated, client-side mock dataset held in browser memory.
- **Deterministic search.** Matching is performed client-side over document titles, text snippets, tags, and metadata. No LLM-generated answers appear in the core experience.
- **AWS integration seam (future work).** AWS S3, Amazon Kendra, and Amazon Bedrock integration is represented only as a modular REST API seam and documented as a future integration point. It is not built in the prototype. Requirements remain implementation-agnostic while noting that the seam exists.
- **Upload handling.** Uploaded `.txt` files have their text content parsed client-side and become keyword-searchable in the demo. Uploaded PDF and Word files appear in the registry feed with their metadata and increment folder counters, but their body text is not searchable in the prototype.
- **AI Executive Brief is a stub.** The Executive Brief is a stretch, [NICE-TO-HAVE] feature rendered from static, pre-written mock summary cards. It is not a real model call.

### Known Limitations

- PDF and Word document **body text is not searchable** in the prototype. For those formats, only metadata and tags are searchable. In-browser PDF/Word text extraction is out of scope. Only `.txt` body text and all documents' metadata/tags are searchable.
- The AI Executive Brief displays static mock content and does not reflect the actual contents of any uploaded document.
- The AWS architecture status badge is presentational; no live AWS connection exists in the prototype.

## Glossary

- **GovArchive**: The Government Policy & Document Portal application defined by this specification.
- **Portal**: The GovArchive single-page web application presented to the user. Used as the system actor in acceptance criteria where the whole application is the responder.
- **Document**: A single archived item consisting of body text (when available) and metadata, including title, file code, category, effective date, department, status, and index keywords/tags.
- **Supported_Format**: A document file type the prototype accepts for upload. Supported formats are `.txt`, PDF, and Word.
- **Searchable_Document**: A Document whose body text participates in keyword matching. In the prototype this comprises mock-dataset documents and uploaded `.txt` documents. PDF and Word documents are not Searchable_Documents at the body-text level; only their metadata and tags are matched.
- **Category**: A classification grouping for documents. The seeded categories are Policies, SOPs, Circulars, Guidelines, Reports, and Meeting Minutes.
- **Folder**: The interactive navigation card representing a Category, displaying a live file counter.
- **Registry**: The in-memory collection of all Documents known to the Portal, including seeded mock documents and documents added during the session.
- **Mock_Dataset**: The pre-populated, client-side set of Documents loaded into the Registry when the Portal starts.
- **Deterministic_Search**: Client-side keyword/substring matching over Document titles, body text (where searchable), tags, and metadata that returns the same results for the same input and dataset, with no model inference.
- **Keyword_Highlight**: The visual marking, in yellow, of the exact text that matched the user's search terms.
- **Filter_Control**: A UI control that narrows the Registry by Category, Effective Year, or Department.
- **Quick_Search_Chip**: A preset control that runs a predefined search query when activated.
- **Results_Feed**: The central list of result cards reflecting the current search and filter state.
- **Preview_Panel**: The side-by-side drawer that shows a selected Document's full text, metadata, and matched-keyword locations.
- **Dual_View**: The toggle inside the Preview_Panel between the Verified Exact Search tab and the AI Executive Brief tab.
- **Executive_Brief**: A stubbed tab in the Preview_Panel that displays static mock structured summary cards.
- **Upload_Modal**: The dialog for selecting a file and entering filing classification before saving a Document to the Registry.
- **Category_Modal**: The dialog for creating a new Category Folder.
- **Folder_Counter**: The numeric file count displayed on a Folder, reflecting the number of Documents in that Category.
- **AWS_Integration_Seam**: A documented, modular REST API boundary reserved for future AWS S3, Kendra, and Bedrock integration; not implemented in the prototype.

## Requirements

### Requirement 1: Header Bar [CORE]

**User Story:** As a government staff member, I want a clear header with the portal identity and a prominent upload action, so that I understand the tool I am using and can quickly file a new document.

#### Acceptance Criteria

1. THE Portal SHALL display the application title "GovArchive | Government Policy & Document Portal" in the header bar.
2. THE Portal SHALL display the sub-headline "Deterministic Document Archiving & Search Portal" in the header bar.
3. THE Portal SHALL display an AWS architecture status badge with the text "Hosted on AWS Architecture • S3 & Kendra Ready" in the header bar.
4. THE Portal SHALL display a primary button labeled "+ Upload & File New Document" in the header bar.
5. WHEN the user activates the "+ Upload & File New Document" button, THE Portal SHALL open the Upload_Modal.

### Requirement 2: Keyword Search [CORE]

**User Story:** As a government staff member, I want to type keywords and see matching documents immediately, so that I can locate the exact policy text I need without waiting.

#### Acceptance Criteria

1. THE Portal SHALL display a primary keyword search bar in the hero area that accepts up to 200 characters of input.
2. WHEN the user enters text in the keyword search bar, THE Portal SHALL update the Results_Feed within 300 milliseconds of the last keystroke to show only Documents whose title, searchable body text, tags, or metadata contain the entered text.
3. THE Portal SHALL perform Deterministic_Search using case-insensitive substring matching over the Registry, treating leading and trailing whitespace in the entered text as trimmed before matching.
4. WHEN the keyword search bar is empty and no Filter_Control is active, THE Portal SHALL display all Documents in the Registry in the Results_Feed, ordered consistently across repeated empty-query views.
5. IF no Document matches the entered keyword and active filters, THEN THE Portal SHALL display a no-results message in the Results_Feed indicating that no Documents match the current search and filters, while retaining the entered keyword text in the search bar.
6. THE Portal SHALL match keywords in PDF and Word Documents against metadata and tags only, excluding their body text.
7. WHILE the entered keyword text contains only whitespace, THE Portal SHALL treat the search as empty and display all Documents in the Registry in the Results_Feed.

### Requirement 3: Filter Controls [CORE]

**User Story:** As a government staff member, I want to filter documents by category, year, and department, so that I can narrow a large archive to the relevant subset.

#### Acceptance Criteria

1. THE Portal SHALL display a Category filter with the options All Categories, Policies, SOPs, Circulars, Guidelines, Reports, and Meeting Minutes.
2. THE Portal SHALL display an Effective Year filter with the options All Time, 2026, 2025, and 2024.
3. THE Portal SHALL display a Department filter with the options All Departments, SDEC, Treasury, Public Works, and HR.
4. WHEN the user selects a value in any Filter_Control, THE Portal SHALL update the Results_Feed to show only Documents that satisfy all active Filter_Controls combined with the current keyword search using logical AND.
5. WHEN a Filter_Control is set to its "All" option, THE Portal SHALL treat that Filter_Control as inactive and SHALL NOT restrict results by that attribute.

### Requirement 4: Quick-Search Chips [CORE]

**User Story:** As a government staff member, I want preset quick-search chips, so that I can run common lookups with a single click during a demo or daily use.

#### Acceptance Criteria

1. THE Portal SHALL display Quick_Search_Chips labeled "Emergency Procurement Limit", "Remote Work Allowance", and "SDEC Taskforce Minutes 2026".
2. WHEN the user activates a Quick_Search_Chip, THE Portal SHALL populate the keyword search bar with that chip's predefined query and SHALL update the Results_Feed to reflect the predefined search.

### Requirement 5: Category Folder Navigation [CORE]

**User Story:** As a government staff member, I want category folders with live counts, so that I can browse the archive by document type and see how much each category holds.

#### Acceptance Criteria

1. THE Portal SHALL display 6 Folders labeled Policies, SOPs, Circulars, Guidelines, Reports, and Meeting Minutes.
2. THE Portal SHALL display an initial Folder_Counter of 142 for Policies, 380 for SOPs, 512 for Circulars, 210 for Guidelines, 195 for Reports, and 640 for Meeting Minutes.
3. WHEN the user activates a Folder, THE Portal SHALL filter the Results_Feed to Documents in that Folder's Category.
4. WHEN a Document is added to the Registry, THE Portal SHALL increment the Folder_Counter of the Document's Category by 1.

### Requirement 6: Search Results Feed [CORE]

**User Story:** As a government staff member, I want result cards that show category, title, matching text, and status, so that I can judge relevance before opening a document.

#### Acceptance Criteria

1. THE Portal SHALL display each result in the Results_Feed as a card containing a Category badge, the Document title, and the Document file code.
2. WHEN a result card is produced from a keyword search, THE Portal SHALL display a matching text snippet with the matched keywords rendered as Keyword_Highlight in yellow.
3. THE Portal SHALL display each result card's metadata, including effective date, filing category, department, and verified/active status, with the verified/active status rendered using an emerald accent.
4. THE Portal SHALL display an action labeled "View Document" or "Open Document Preview" on each result card.
5. WHEN the user activates the view action on a result card, THE Portal SHALL open the Preview_Panel for that Document.

### Requirement 7: Document Preview Panel [CORE]

**User Story:** As a government staff member, I want a side-by-side preview showing the full document text with matched keyword locations, so that I can verify the exact source of a result.

#### Acceptance Criteria

1. WHEN the Preview_Panel opens for a Document, THE Portal SHALL display the Document's full available text in a side-by-side drawer.
2. WHILE a keyword search is active, THE Portal SHALL render matched keywords in the Preview_Panel text as Keyword_Highlight in yellow.
3. THE Portal SHALL display the selected Document's metadata and active/verified status in the Preview_Panel.
4. THE Portal SHALL display the page or clause location of each matched keyword in the Preview_Panel.
5. THE Portal SHALL display a download button in the Preview_Panel.
6. WHERE a Document has no available body text, THE Portal SHALL display a notice in the Preview_Panel that body text is unavailable for the Document's format.

### Requirement 8: Dual-View Toggle — Verified Exact Search [CORE]

**User Story:** As a government staff member, I want a verified exact-search view with citations, so that I can trust that the displayed text is the literal source without any generated content.

#### Acceptance Criteria

1. THE Portal SHALL display a Dual_View toggle in the Preview_Panel with a tab labeled "Verified Exact Search" and a tab labeled "AI Executive Brief".
2. THE Portal SHALL select the "Verified Exact Search" tab by default when the Preview_Panel opens.
3. WHILE the "Verified Exact Search" tab is selected, THE Portal SHALL display the exact matching text with page or clause citations.

### Requirement 9: Upload & File Document Modal [CORE]

**User Story:** As a government staff member, I want to upload and classify a document, so that it joins the registry and becomes discoverable.

#### Acceptance Criteria

1. WHEN the user opens the Upload_Modal, THE Portal SHALL display a file dropzone containing the text "Drag and drop PDF/Word documents here".
2. WHEN the user opens the Upload_Modal, THE Portal SHALL display a filing classification form containing a required Category selector dropdown, a required Effective Date date input, a required Issuing Department text input, and an optional Index Keywords/Tags tag input.
3. WHEN the user opens the Upload_Modal, THE Portal SHALL display a primary button labeled "Save & File to Registry".
4. IF the user activates "Save & File to Registry" while one or more of the required fields (Category, Effective Date, Issuing Department) or the uploaded file is absent, THEN THE Portal SHALL display an inline validation message identifying each absent required field, SHALL retain the Upload_Modal in its current state, and SHALL NOT add a Document to the Registry.
5. WHEN the user activates "Save & File to Registry" with all required fields completed and a Supported_Format file selected, THE Portal SHALL add the Document to the Registry with its entered metadata, SHALL increment the selected Category's Folder_Counter by exactly 1, and SHALL display a success toast for 3 seconds.
6. WHEN a `.txt` Document is uploaded, THE Portal SHALL parse the Document's text content and index the parsed text such that the Document becomes a Searchable_Document in Deterministic_Search.
7. WHEN a PDF or Word Document is uploaded, THE Portal SHALL add the Document to the Registry with its metadata and SHALL exclude the Document's body text from Deterministic_Search.
8. IF the user selects a file whose format is not a Supported_Format (.txt, .pdf, or Word), THEN THE Portal SHALL reject the file, SHALL display an error message indicating that the format is unsupported, and SHALL NOT add the Document to the Registry.

### Requirement 10: AI Executive Brief Stub [NICE-TO-HAVE]

**User Story:** As a government staff member, I want a structured brief view, so that I can preview how summarized guidance would appear in a future release.

#### Acceptance Criteria

1. WHEN the user selects the "AI Executive Brief" tab in the Preview_Panel, THE Portal SHALL display static mock summary cards titled "Mandatory Rules", "Approval Matrix", and "Action Steps".
2. THE Portal SHALL render the Executive_Brief content from static mock data without invoking a model.

### Requirement 11: Create New Category Folder Modal [NICE-TO-HAVE]

**User Story:** As a government staff member, I want to create a new category folder, so that I can organize documents under classifications beyond the seeded set.

#### Acceptance Criteria

1. THE Portal SHALL display in the Category_Modal a Category Name input, a Description/Scope input, and a Category Color Accent tag input.
2. THE Portal SHALL display in the Category_Modal a button labeled "Create Folder".
3. WHEN the user activates "Create Folder" with the Category Name provided, THE Portal SHALL add the new Category to the Folder navigation with a Folder_Counter of 0, to the Category selector dropdown in the Upload_Modal, and to the Category Filter_Control.
4. IF the user activates "Create Folder" with the Category Name empty, THEN THE Portal SHALL display a validation message and SHALL NOT create a Category.

### Requirement 12: Mock Dataset and AWS Integration Seam [CORE]

**User Story:** As a developer demonstrating the prototype, I want a pre-populated dataset and a documented integration boundary, so that the demo runs without a backend while leaving a clear path to AWS.

#### Acceptance Criteria

1. WHEN the Portal starts, THE Portal SHALL load the Mock_Dataset into the Registry held in browser memory.
2. THE Portal SHALL perform all search and filtering against the in-memory Registry without a backend service call.
3. THE Portal SHALL isolate data retrieval behind an AWS_Integration_Seam defined as a modular REST API boundary that is documented as future work and not implemented in the prototype.
```