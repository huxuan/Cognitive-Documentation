<!-- 
NavPath: Academic Knowledge API/Knowledge Exploration/Entity Attributes
LinkLabel: Common Entity Attributes
Url: Academic-Knowledge-API/documentation/KnowledgeExploration/EntityAttributes/CommonEntityAttributes
Weight: 695
-->

#Entity Attributes

The academic graph is composed of 7 types of entity. All entities will have a Entity ID and a Entity type.

### Common Entity Attributes
Name	|Description	            |Type       | Operations
------- | ------------------------- | --------- | ----------------------------
Id		|Entity ID					|Int64		|Equals
Ty 		|Entity type 				|enum	|Equals

### Entity type enum
Name 															|value
----------------------------------------------------------------|-----
[Paper](PaperEntity.md)								|0
[Author](AuthorEntity.md)								|1
[Journal](JournalEntity.md)	 						|2
[Conference Series](ConferenceSeriesEntity.md)					|3
[Conference Instance](ConferenceInstanceEntity.md)	|4
[Affiliation](AffiliationEntity.md)					|5
[Field Of Study](FieldsOfStudy.md)						|6

