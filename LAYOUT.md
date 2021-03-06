### GIT-GED CONCEPTS

1. Git-ged commands
    - repo-level commands:
        - _init_: populates `refs/heads/{master,content,intent/master}` with README, LICENSE_DEFAULT, etc. as specified by user
        - _clone_: same as git clone
        - _attach_: same as git remote add, except that it allows the user to define the set of permanames of interest in the remote repo
        - _fetch_: same as git fetch, except that only permanames that the user currently has are fetched (along with `refs/heads/master`)
        - _push_: same as git push, except that only permanames that the remote repo has are updated
    - entity- and state-level commands:
        - _intent_: captures one-line user intent to capture large-scale project (separate from commit message on a single entity)
        - _ingest_: populates `refs/local/gedcoms/{permaname(s)}` with a raw gedcom, adding new gedN files for permaname collisions
        - _filter_: performs living filter and populates `refs/heads/gedcoms/{permaname}`
        - _import_: creates any new persons/families and records import flag & new person/family permaname+state links in gedX.IMPORT
        - _resolve_: deals with multiple-record histories in a smart way (gedcoms/persons/families)
        - _owner_: establishes identity of the owner of the repo in `refs/heads/content:/OWNER`, adds person for the user and allows specification of 'mpmmp' ancestor links to deceased horizon
    - interactive-use commands:
        - _workspace_: (subcommands: checkout, update, reset)
            - _checkout_: grabs every specified person/family/gedcom permaname and puts it under the path repo/[persons|families|gedcoms]/{permaname} in a transient commit
            - _update_: grabs the head of every specified person/family/gedcom permaname and puts it under a new `refs/heads/workspace` commit
            - _reset_: warns if un-git-ged-committed `refs/heads/workspace` history, then nukes it and recreates a `refs/heads/workspace` commit with the same permanames as it had before
        - _commit_: creates commits on every separate permaname that has changed, using current intent as default commit subject plus details of all entities that changed
<br>
2. Data Licenses
    - Data license defaults to Creative Commons Attribution-ShareAlike 3.0 Unported
    - Users can update license at repo/gedcom/record level
    - Other common license options are easily specifiable
        - Other Creative Commons options: CC-BY-3.0, CC-BY-NC-3.0, CC-BY-NC-SA-3.0, CC0-1.0
        - Open Data Commons options: ODC-PDDL-1.0, ODC-BY-1.0, ODC-ODBL-1.0
<br>
3. Gedcom
    - all ingested gedcoms are stored verbatim under a `refs/local/gedcoms/{permaname}` ref
    - all imported, living-filtered gedcoms are stored under a `refs/heads/gedcoms/{permaname}` ref
    - there are multiple permanames for a gedcom:
        - primary permaname for gedcom is the sha256 hash of "FILE & SUBM name[/email]"
        - alternate permaname for gedcom is the sha256 hash of "FILE & DATE"
        - alternate permaname for gedcom is the sha256 hash of "source path & SUBM name"
        - alternate permaname for gedcom is the sha256 hash of all person _UUIDs, sorted alphanumerically
    - keep your email, paths & filenames consistent to maximize permaname collisions
    - copies the repo's LICENSE\_DEFAULT unless overridden by the user at ingest time
<br>
4. Person
    - all imported persons are stored in a standard JSON format under a `refs/heads/persons/{permaname}` refs
    - there are **multiple** permanames for a person:
        - one is designated as primary, all permanames are annotated internally to the person by a versioned hash function (for easy recomputation)
        - primary permaname for person defaults to the sha256 hash of the gedcom UID + sorted 'not-same-as' list
        - alternate permaname for person is the sha256 hash of "&lt;displayname&gt;[ birthdate][ deathdate][ sorted 'not-same-as' list]" (can be designated as primary)
        - alternate permaname for person is the sha256 hash of "&lt;displayname&gt; child of &lt;permaname&gt;" (for each parent, if there is no birth/death)
        - other alternate permanames are definable, as long as the context-free code for computing them is included under `refs/heads/code`
        - permanames based on "displayname" state should not change when the person's name is updated, they are designed to produce collisions based on import data
        - although a person's primary permaname should remain largely constant, it can change for reasons of disambiguation (addition of a 'not-same-as' link)
    - proper protection of living data
        - all imported persons must be deceased and publicly-viewable, or the record will initially exist under the `refs/local/persons/...` refspace
        - import is responsible for initially segmenting living vs. deceased, but after import the app is responsible for calculating living after edit and moving the record if necessary
        - the same living check & segmentation logic that import uses will be available on-demand for apps to use after edit
        - when a deceased record is made living, the deceased record's permaname gets a new commit that marks it as no-longer-visible (so that on subsequent fetch, other repositories can delete their records)
    - includes optional "supersedes", "superseded-by", "derived-from", "descendant-of", "same-as", and "not-same-as" link attributes
        - the "supersedes", "superseded-by", and "derived-from" link attributes allow for proper history-aware linkage across permanames for tracing merge or other complicated person record derivation
        - the "descendant-of" attribute is for attaching non-descript living persons to their deceased horizon via "mpmmp" links
        - the "same-as" attribute is a loose identity link to an alternate history of a person determined to be the same historical person, it is a hint that a merge or other record derivation may be useful in the future
        - if the "not-same-as" attribute refers to a disconnected history of a *different* person under the same permaname, it is best to change the permaname of each person this attribute is added to
    - person merge is often not needed if the persons originated unchanged from the same gedcom
    - person merge may be a "comes before/comes after" decision at import time
    - person merge decisions can be deferred by storing two person entities under the same permaname and letting the user make the before/after decision at a later time
    - multiple persons can be stored under the same permaname under 'entity1'..'entityN' blob names
    - copies the gedcom LICENSE if imported, copies the repo's LICENSE\_DEFAULT if a newly-created person
<br>
5. Family
    - direct represenation of the GEDCOM family concept
    - all imported families are stored in a standard JSON format under a `refs/heads/families/{permaname}` refs
    - there are multiple permanames for a family:
        - one is designated as primary, all permanames are annotated internally to the family by a versioned hash function (for easy recomputation)
        - primary permaname for family defaults to the sha256 hash of the parents' primary permanames in alphanumeric order
        - alternate permaname for family is the sha256 hash of "<displayname>[ displayname][ marriage date]" (if there is marriage information, displaynames ordered by primary permaname alphanumeric ordering of parents)
        - other alternate permanames are definable, as long as the context-free code for computing them is included under `refs/heads/code`
        - permanames based on "displayname" state should not change when the person's name is updated, they are designed to produce collisions based on import data
        - primary permaname should change if the parents' permanames are modified, or if a parent is replaced with a different person
    - includes person links for each person in each family position
    - includes optional "supersedes", "superseded-by", "derived-from", "same-as", and "not-same-as" link attributes (like person)
    - includes optional "prior-family" link attribute that tracks temporal household dissolution & recomposition
    - family merge is often not needed if the family originated unchanged from the same gedcom
    - family merge can be a "comes before/comes after" decision at import time, but is not as likely as person to fit this workflow
    - family merge decisions can be deferred by storing two family entities under the same permaname and letting the user make the before/after decision at a later time
    - multiple families can be stored under the same permaname under 'entity1'..'entityN' blob names
    - copies the gedcom LICENSE if imported, copies the repo's LICENSE\_DEFAULT if a newly-created record
<br>
6. Link Attribute
    - link attributes always refer to BOTH primary permaname & state
    - link attributes are encoded similar to XFN to start
<br>
7. Merge Strategies
    - merge comes in at least 4 flavors:
        1. (import) alternate record storage + history linkage (recommended for automated import processes)
        2. (post-import) "comes before/comes after" record replacement strategy for dealing with record reconciliation
        3. (post-import) "pick correct values" record reconciliation strategy for dealing with partial truth from multiple sources (sets "derived-from" attribute(s))
        4. (post-import) "disambiguation" record reconciliation strategy for separating records that are NOT the same person (sets "not-same-as" attribute(s))
    - of course anyone can do anything they please to their records, but these strategies seem to deserve automation support
<br>
8. Record deriviation tracing
    - each of the following link attributes refers to another record via permaname+state reference
    - "supersedes" indicates that a person (or family) is a full, identical replacement for another (used to be able to react to remote merges)
    - "superseded-by" indicates that a static, not-to-be-further-edited  person (or family) is left behind after having been merged into the canonical record (used to detect conflicts between local edits & remote merge or vice versa)
    - "same-as" indicates a more-tentative-than-merge attempt at establishing identity to allow for disparate efforts to proceed before attempting a full merge
    - "disambiguated-by" indicates a forward-reference from a person/family that has no useful future except for providing opportunity for fruitful permaname collision & disambiguation hints to users on subsequent imports
    - "derived-from" indicates a record that took a significant amount of data from another record, from which an identity was extracted
    - "not-same-as" indicates a definitive statement that one person (or family) is NOT the same as another (used when extracting a half-merged entity that collided on a permaname)
    - "prior-family" indicates that a family has many of the same members of a prior family, but temporal household dissolution & recomposition require two different families to be tracked (allows for "current family" computations based on a directed graph of prior-family edges)
<br>
9. Deletion of Records
    - person/family/gedcom delete comes in three flavors:
        1. "deref": I no longer care to track or maintain this person, if anyone has this person cloned, let them continue to maintain the record
        2. "hide": I want the record gone on any repo that follows mine, if they fetch from me, MAKE their record go away, dead-to-living transition uses this mechanism
        3. "delete": I want the record gone on any repo that follows mine, if they fetch from me, SUGGEST that their record go away
    - "hide" and "delete" stubs hang around for a longish-but-limited amount of time, and there is a mechanism that automatically cleans them up every so often
<br>

### GIT-GED REFS LAYOUT

Here are the general patterns of things that live under `refs/`:

1. `refs/heads/*`:
    - stuff that can be cloned/fetched
<br>
2. `refs/heads/{gedcoms,persons,families/intents}/*`:
    - data that can be cloned/fetched/forked piecemeal
<br>
3. `refs/local/*`:
    - dispensible stuff that is used for local import actions (not needed for collaboration)
    - hidden stuff that should not be published on a clone/fork
<br>

Here are specific details about each kind of ref:

1. `refs/heads/master` (fetchable, but non-mergeable):
    - `README`: simple documentation of git-ged, with pointer to software to parse/use
    - `LAYOUT`: this file containing details of git-ged structure
<br>
2. `refs/heads/content` (fetchable, but non-mergeable):
    - `META`: last version of git-ged that wrote
    - `OWNER`: the identity of the owner of this repo + OpenID URLs, blog URLs and/or email(s)
    - `LICENSE_DEFAULT`: default license for any new records added or imported into to this repository, defaults to Creative Commons Share-alike
    - `ROOTS`: links to various person permaname roots, to start tree navigation from
    - `ENTITIES`: list of gedcom/person/family/intent permaname+state links, may end up needing to be sharded for very large repos
    - `INTENTS`: list of last N intents & date & author
    - `CHANGELOG`: contains up to last 100 edits performed, including list of entities changed, updated by git-ged commit
    - `LIMITS`: maximum number of intents, etc.
<br>
3. `refs/heads/intents/master` (fetchable, but non-mergeable):
    - `INTENT`: stores the user's git identity, date, and, single-line 140-char intent message in the tree itself to capture "why"
    - MUST exist: if no intent is explicitly stored, init/import/edit must stub one in that describes the largest-scope action being taken
    - able to cherry-pick an intent from some other user to show you're working toward the same goal as someone else
    - able to generate a feed of intents from here
    - capped at 100 or so max commits (large limit)
<br>
4. `refs/heads/intents/{permanames}`:
    - selective intents used to document why a certain change is made (linked from entities at commit)
<br>
5. `refs/heads/workspace` (fetchable, but non-mergeable):
    - non-history-preserving workspace tree for pulling a subset of records into a filesystem for edit
<br>
6. `refs/heads/gedcoms/{permanames}`:
    - `ged1`: the living-filtered gedcom file renamed to a standard filename
    - `gedN`: the living-filtered gedcom file renamed to a standard filename
    - contains NO living data, as a result of "ingest" followed by "import"
    - `gedX.META`: link to intent; original file name & path; where/who the file came from; all permanames this gedcom was stored under (by permaname kind)
    - `gedX.LICENSE`: license for use of this gedcom as a whole, copied to individual persons/families at import time
<br>
7. `refs/local/gedcoms/{permanames}`:
    - _may_ contain living data, populated without filtering by "ingest"
    - `ged1`: the raw gedcom file renamed to a standard filename
    - `gedN`: the raw gedcom file renamed to a standard filename
    - pre-existing permanames get forwarded up to the new tree (collisions, yay!)
    - new permanames get created, non-colliding permanames don't get forwarded (gc'd by some other mechanism)
    - if the gedcom contains no living data, "import" can delete the `refs/local` refs
    - can coexist with `refs/heads/gedcoms/{permaname}` for a while as documentation of exactly what got imported
    - fetch/clone/fork does NOT include the original gedcom, just the "post-import" one (because only `refs/heads` comes along)
<br>
8. `refs/heads/persons/{permanames}`:
    - `entity1`: the primary person in a standard JSON form
    - `entityN`: alternate, not-yet-merged person records
    - `entityX.{format}`: the primary person (or an alternate) in an alternate format
    - `entityX.META`: link to intent; link(s) to gedcom permanames
    - `entityX.IDENTITY`: optional "supersedes", "superseded-by", "derived-from", "same-as", and "not-same-as" link attributes; also any arbitrary XFNs to other permanames
    - `entityX.LICENSE`: license for use of this person, copied from gedcom at import time, or from repo's LICENSE\_DEFAULT at non-import-creation time
    - commits on a person can easily be merged/fast-forwarded if history is shared and there are no textual conflicts
    - commits on two persons of the same permaname that do NOT share history can undergo person merge
    - merge commits set "supersedes" and "superseded-by" link attributes to allow post-merge traceability
    - merging of META takes a "most recent intent / union" approach
    - merging of IDENTITY takes a "union with conflict detection & manual resolution" approach
    - merging of LICENSE takes the more restrictive license by default
<br>
9. `refs/local/persons/{permanames}`:
    - non-public person record, behaves like person in every other respect
    - typically a person record should exist under EITHER `refs/local` OR `refs/heads`
    - if both exist, the `refs/heads` record is used exclusively
<br>
10. `refs/heads/families/{permanames}`:
    - `entity1`: the primary family in a standard JSON form
    - `entityN`: alternate, not-yet-merged family records
    - `entityX.{format}`: the primary person (or an alternate) in an alternate format
    - `entityX.META`: link to intent; link(s) to gedcom permanames
    - `entityX.IDENTITY`: optional "prior-family", "derived-from", "supersedes", "superseded-by", "same-as", and "not-same-as" link attributes; also any arbitrary XFNs to other permanames
    - `entityX.LICENSE`: license for use of this person, copied from gedcom at import time, or from repo's LICENSE\_DEFAULT at non-import-creation time
    - commits on a family can easily be merged/fast-forwarded if history is shared and there are no textual conflicts
    - commits on two families of the same permaname that do NOT share history can undergo family merge
    - merging of META takes a "most recent intent / union" approach
    - merging of IDENTITY takes a "union with conflict detection & manual resolution" approach
    - merging of LICENSE takes the more restrictive license by default
<br>
11. `refs/local/families/{permanames}`:
    - non-public family record, behaves like family in every other respect
    - typically a family record should exist under EITHER `refs/local` OR `refs/heads`
    - if both exist, the `refs/heads` record is used exclusively
<br>
