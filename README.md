# Payment review API (practice project)

This is a small API that simulates importing and storing transaction data from an upstream API and handling workflow/review management of the transactions (such as notes, approval or rejection). 

Next step would be to return data (f.ex APPROVED) to the upstream API and refresh Spanner state, but that is not implemented here. It's meant as a small practice project to learn the tech stack and not for production. 

## Architecture / data ownership
- Source of truth
	- In actual production the ultimate source of truth would be from an upstream bank API. In this practice project, generated transactions simulate the data that would normally come from the upstream API, and the service imports the simulated data into Spanner.
- OpenAPI
	- Definitions of API contracts 
- Spanner 
	- Storage of imported transaction data
- Firestore
	- Stores a flexible document related to each imported transaction with for example review, workflow and timeline related data

## Endpoint design
- `POST /v1/dev/generate-transactions`
	- Generates N random transactions (at least 2)
	- Marks a random amount as requiring review (at least 1, otherwise up to 20% of total)
	- Writes the generated transactions to Spanner
	- Creates matching Firestore review docs for transactions that require review
- `GET /v1/transactions`
	- Returns a list of imported transactions from Spanner
- `GET /v1/transactions/{id}`
	- Reads canonical imported transaction data from Spanner
- `POST /v1/transactions/{id}/review-notes`
	- Append a note/reason code etc. in Firestore
- `GET /v1/transactions/{id}/review`
	- Reads the flexible workflow/review data from Firestore
- `POST /v1/transactions/{id}:approve`
	- Updates internal review state for the transaction in Firestore
- `POST /v1/transactions/{id}:reject`
	- Updates internal review state for the transaction in Firestore

## Database 

Spanner tables:

- `transactionId`
- `fromAccountId`
- `toAccount`
- `amount`
- `currency`
- `externalStatus` (`RECEIVED`, `BOOKED`, `SETTLED`)
- `createdAt`
- `importedAt`

Firestore collections:

- `transactionId`
- `requiresReview`
- `reviewStatus` (`PENDING_REVIEW`, `APPROVED`, `REJECTED`)
- `reasonCodes`
- `notes`
- `lastReviewedBy`
- `timeline`