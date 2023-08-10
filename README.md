# argoteam-dice-api
Argoteam's repository for DIve CEnter evaluations and database REST API.

# PURPOSE

The purpose of this API is to provide an agnostic backend for dive center evaluations.
It functions has a front for the database and provides a flexible and configurable interface to create evaluation cards.
Evaluation cards are very configurable to provide flexibility and evolution capabilities over time.

# MODEL

The database model is as follow:

```mermaid
erDiagram

User {
	int id PK
	string email
	string password
}

Account {
	int id PK
	int user_id FK "User.id"
	string first_name
	string last_name
}

Agency {
	int id PK
	string name
	string description
}

Certification {
	int id PK
	string name
	int agency_id FK "Agency.id"
}

DiverCertification {
	int id PK
	int user_id FK "User.id"
	int certification_id FK "Certification.id"
	Date date
}

DiveCenter {
	int id PK
	string name
	string country
	string city
	string street_address
	string postal_code
	string email
	string accessible_water
}

DiveCenterContact {
	int id PK
	int diver_center_id FK "DiveCenter.id"
	string name
	string email
	string position
}

DiveCenterAgencyAffiliation {
	int id PK
	int diver_center_id FK "DiveCenter.id"
	int agency_id FK "Agency.id"
}

Evaluation {
	int id PK
	int evaluator FK "User.id"
    int global_note
    string global_comment
}

EvaluationCard {
    int id PK
    int evaluation_id FK "Evaluation.id"
    string name "Name of the card like 'safety' it should be an i18n translatable string"
}

EvaluationCardQuestion {
    int id PK
    int evaluation_card_id FK "EvaluationCard.id"
    string question
    string type "Ex: grade, free_text, etc. This will be interpreted by the client to display the correct UI. For example 'rank' should be displayed as a ranking over 5."
    list possible_answers "Proposal: a comma separated list of possible (translatable) answers."
}

EvaluationCardAnswer {
    int id PK
    int evaluation_card_id FK "EvaluationCard.id"
    int evaluation_card_question_id FK "EvaluationCardQuestion.id"
    data answer "This must be a data type as the type of the answer is tied to the type of the question. The client will use the answer's type to display the result"
    string type "The type of the answer: int, string, text, etc."
}

User ||--|| Account : has
User ||--o{ DiverCertification : has
DiverCertification ||--|| Certification : is
Certification }|--|| Agency : "delivered by"
DiveCenterAgencyAffiliation }|--|| DiveCenter : has
DiveCenterAgencyAffiliation }|--|| Agency : affiliates
DiveCenter ||--|{ DiveCenterContact : has
DiveCenter ||--o{ Evaluation : has
Evaluation ||--|{ EvaluationCard : contains
EvaluationCard ||--|{ EvaluationCardQuestion : contains
EvaluationCard ||--|{ EvaluationCardAnswer : contains
EvaluationCardQuestion o|--|{ EvaluationCardAnswer : contains

```

In short, the user info are decoupled from the account info and from the right management system. The diver profile information are actually held by the diver's certifications. A diver can have multiple certifications from multiple agencies.

Dive centers can have multiple contacts and can be affiliated to multiple agencies. They can also have multiple point of contact. A contact can only be related to one dive center though. A dive center can have multiple evaluations (obviously).

An evaluation only contains global information: a note and a comment. All the specific evaluation data are in the EvaluationCard. The card is dynamically build with a set of EvaluationCardQuestions to which an EvaluationCardAnswer is provided. An evaluation card can contains multiple questions but a question is linked to only one card. An answer belongs to a single question and a single card (answers are specific to a question in a card). Both the question and the answer contains the type that is expected.

There are 2 main questions left on the data model:

 1. Should the answer be serialized as JSON strings or binary data (my personal preference is to strings since binary makes little sense in this context).
 2. Should the possible answers be included in the data model (considering that at this point we do not have multiple answers in the form), and if for future proofing reasons we do include them, shouldn't they be in a EvaluationCardQuestionPossibleAnswer table? The argument for making it a coma separated list for now is that we do not have questions with multiple answers, so it is possible that it will never be useful.

The API does not mandate a specific presentation, it only provides the data for the client (web app, mobile app, etc.) to present them to the end user.

# BUILD / DEVELOP

There's a Makefile at the root of the directory, it contains all the basic commands you will need to contribute/use that API.

The API is written in Python and Flask. Please have a look at the requirements.txt for the list of required dependencies.

## Build the Docker image for the API

```make docker-build```

## Run the Docker image for the API

```make docker-run```

## initialize the database

We use SQLAlchemy for the database.

```make initdb```

## Migrate the database

```make migratedb```

## Upgrade the database

```make upgradedb```
