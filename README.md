# argoteam-dice-api
Argoteam's repository for DIve CEnter evaluations and database REST API.


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
