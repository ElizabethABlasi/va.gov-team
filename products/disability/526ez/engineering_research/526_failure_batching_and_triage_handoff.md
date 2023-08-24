# 526 Failure re-try batching and triage handoff

## Purpose

A collection of key take aways from the knowledge transfer between Kyle Soskin
and Sam Stuckey, specificaly relevant to the work of batch retrying historical
failed 526 submissions, as well as on going triage.

## Context

### The Work

The following detail what happened, how it was being addressed, and why we are now transfering this work to a new team
- [Historical Epic](https://app.zenhub.com/workspaces/benefits-team-1-6138d7b57a2631001a4b7562/issues/gh/department-of-veterans-affairs/va.gov-team/44994)
- [Transitional Ticket](https://app.zenhub.com/workspaces/benefits-team-1-6138d7b57a2631001a4b7562/issues/gh/department-of-veterans-affairs/va.gov-team/62593)

### TL;DR
In mid 2022 it was discovered that we had roughly ~150k failed user submissions
in our 526 form that had never been addressed.  These failures had presented to
end users as non-failures with a message to the effect of "We cqn't submit this
now, but we will keep trying.  Due to a lack of cohesive monitoring these
failures quitely piled up where no one was looking, leaving vetterans in the
dark about the status of their claims.  THis problem is coliquially known to the team (now) as the "black hole" problem, as
these submissions had effectively vanished.  See the afforementioned tickets for more context
on the bug and the initial strategy to address it.

For these historical failures, after de-duping the original 150k (Some vets had
tried multiple submissions), Kyle Soskin from veteran-benefits-team-1 was tasked with
a multi faceted project to begin correcting this problem. A major part of this
work has been enabling the VBA to manually resubmit these forms. Our part of this process (or more specificially, Kyle's part until now)
was / is to run a time consuming and semi-manual process wherin we use
 our existing code (via a production rails console) to generate PDFs which are in turn sent in
bacthes to VBA for re-submission (how they actually leverage these files towards
resubmitting is a black box and beyond the scope of our work.  
This process requires enhanced security credentials, 
knowledge of our existing available infrastructure for generating these PDFs,
and some proprietery scripting by kyle. Until now, the onus for this work, as
well as related or adjacent triage, was falling soly on him due to the siloing
of this knowledge.

The purpouse of this knowledge transfer is to
1. Allow Kyle to be moved off on to other work, and Sam to pick up where he left
   off.
2. Allow Sam the insight needed to document and potentially automate this
   process.
3. Ultimately De-silo the knowledge and hopefully remove the need for dangrous production access

## The Batch Resbumission Flow

This 'batching' work described above, roughly entails:
1. getting / building a list of submission IDs to target for the batch
2. Login to a Production connected Rails Console
3. Import the IDs from step 1.
4. Enque the jobs using Sidekiq and existing application objects
5. Pull a list of successfully uploaded filenames from the S3 bucket
6. generate signed links to these batches in S3
This allows our VBA counterparts to easily pull these batched fils

**NOTE:**
Downstream services are
- EVSS 526 PDF document generation
    - `<evss url>/rest/form526/v2/getPDF`
- Lighthouse Backup submission endpoint
    - `asdf`

TODO: break down each of these steps with code

### 1. Create a list of submission ids to process
At the moment, this is simply taking a subsection of the yet to be unprocessed
submission failures from the original ~40k

### 2. Login to a production connected Rails Console

### 3. Import the IDs from step 1
### 4. Enqueue the jobs using application logic

This work can be broken into 3 parts
1. Create a sidekiq batch
2. Loop over the submission Ids
3. Enqueue a Job (as a part of the batch) for each ID.  That code looks like this

```

```

This code will handle the PDF generation, ziping, and upload of each new
submission.  On the VBA end, they will download this file (using the links we
will generate in step 6) and run their manual process on it.

Additionally, by creating a sidekiq batch we are able to view the progress of
this work in real time at https://api.va.gov/sidekiq/batches 
    NOTE: viewing sidekiq requires special access.  If you do not have access
    [follow the steps here to get it.](https://depo-platform-documentation.scrollhelp.site/developer-docs/sidekiq-ui-access)



### 5. Pull a list of filenames from s3

```
aws s3 ls $bucket > my_local_file.txt
```

### 6. Generate signed links for each S3 upload

This allows our VBA counterparts to easily pull these batched fils.  We are currently using the following script

