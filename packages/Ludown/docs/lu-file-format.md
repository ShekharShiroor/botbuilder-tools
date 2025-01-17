# .lu File Format
.lu files contain markdown-like, simple text based definitions for [LUIS](http://luis.ai) or [QnAmaker.ai](http://qnamaker.ai) concepts. 

## Intent
An intent represents an action the user wants to perform. The intent is a purpose or goal expressed in a user's input, such as booking a flight, paying a bill, or finding a news article. You define and name intents that correspond to these actions. A travel app may define an intent named "BookFlight."

Here's a simple .lu file that captures a simple 'Greeting' intent with a list of example utterances that capture ways users can express this intent. You can use - or + or * to denote lists. Numbered lists are not supported.

```markdown
# Greeting
- Hi
- Hello
- Good morning
- Good evening
```

'#\<intent-name\>' describes a new intent definition section. Each line after the intent definition are example utterances that describe that intent.

You can stitch together multiple intent definitions in a single file like this:

```markdown
# Greeting
- Hi
- Hello
- Good morning
- Good evening

# Help
- help
- I need help
- please help
```
Each section is identified by #\<intent name\> notation. Blank lines are skipped when parsing the file.

## Entity
An entity represents detailed information that is relevant in the utterance. For example, in the utterance "Book a ticket to Paris", "Paris" is a location. 

|Sample user utterance|entity|
|--------------------------|----------|
|"Book a flight to **Seattle**?"|Seattle|
|"When does your store **open**?"|open|
|"Schedule a meeting at **1pm** with **Bob** in Distribution"|1pm, Bob|

Entity in .lu file is denoted using {\<entityName\>=\<labelled value\>} notation. Here's an example: 

```markdown
# CreateAlarm
- book a flight to {toCity=seattle}
- book a flight from {fromCity=new york} to {toCity=seattle}
```

LUDown tool supports the following [LUIS entity types](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-concept-entity-types)
- Prebuilt ("datetimeV2", "age", "dimension", "email", "money", "number", "ordinal", "percentage", "phoneNumber","temperature", "url", "datetime", "keyPhrase")
- List
- Simple
- RegEx

LUDown tool **does not** support the following LUIS entity types:
- Hierarchical

You can define: 
- [Simple](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-quickstart-primary-and-secondary-data) entities by using $\<entityName\>:simple notation. Note that the parser defaults to simple entity type.
- [PREBUILT](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/pre-builtentities) entities by using $PREBUILT:\<entityType\> notation. 
- [List](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-quickstart-intent-and-list-entity) entities by using $\<entityName\>:\<CanonicalValue\>**=**<List of values> notation.
- [Composite](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-tutorial-composite-entity) entities using $\<entityName\>:[childEntity1, childEntity2, ...] notation. See [here](../examples/compositeEntities.lu) for an example.

**Note:**
```markdown
Entity names in Ludown cannot include a colon ':'. E.g. $Country:Office:Simple is invalid.  
```

Here's an example: 

```markdown
# CreateAlarm
- create an alarm for {alarmTime=7AM}
- set an alarm for {alarmTime=7AM}

$alarmTime:Simple
```
Pre-built entities only need to be defined once and are applicable to your entire application. Here's an example: 
```markdown 
# Add
- 1 + 1

> 1's in the "1 + 1" utterance will automatically be picked up as numbers by LUIS
$PREBUILT:number 

# BookTable
- book a table for tomorrow
- book a table for 3pm
- book a table for next thursday 4pm

> all date or time or date and time in utterances will automatically be picked by LUIS as datetime values
$PREBUILT:datetimeV2
```

You can describe list entities using the following notation:
$listEntity:\<normalized-value\>=
    - \<synonym1\>
    - \<synonym2\>

When using list entity, you should include a value from the list directly in the utterance, not an entity label or any other value. Here's an example definition of a list entity: 

```markdown
# CommunicationPreference
- set phone call as my communication preference
- I prefer to receive text message

$commPreference:call=
	- phone call
	- give me a ring
	- ring
	- call
	- cell phone
	- phone
$commPreference:text=
	- message
	- text
	- sms
	- text message
```

[RegEx entities](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-quickstart-intents-regex-entity) are defined by a regular expression the user provides as part of the entity definition.

In the .lu file format, these are represented using 
$\<entity-name\>:/regExPattern/ notation

Note that the regEx pattern needs to be enclosed within forward slashs - '/'

Here's an example of regex entity: 

```markdown
$HRF-number:/hrf-[0-9]{6}/
```

[Composite entities](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-tutorial-composite-entity) enable grouping related entities into a parent category entity. 

```markdown
# setThermostat
> This utterance labels ‘thermostat to 72’ as composite entity deviceTemperature
    - Please set {deviceTemperature = thermostat to 72}
> This is an example utterance that labels ‘owen’ as customDevice (simple entity) and wraps ‘owen to 72’ with the ‘deviceTemperature’ composite entity
    - Set {deviceTemperature = {customDevice = owen} to 72}

> Define a composite entity ‘deviceTemperature’ that has device (list entity), customDevice (simple entity), temperature (pre-built entity) as children
$deviceTemperature: [device, customDevice, temperature]

$device : thermostat=
    - Thermostat
    - Heater
    - AC
    - Air conditioner

$device : refrigerator=
    - Fridge
    - Cooler

$customDevice : simple

$PREBUILT : temperature
```

### Roles

Every entity type except Phrase Lists can have [roles](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-roles). Roles are named, contextual subtypes of an entity. 

Roles in .lu file format can be explicitly or implicity defined. 

Explicit definition follow the following notation - $\<entityName>:\<entityType> Roles=role1, role2, ...
```markdown
> # Simple entity definition with roles

$userName:simple Roles=firstName,lastName
```

Implicit definition: You can refer to roles directly in patterns as well as in labelled utterances via {\<entityName>:\<roleName>} format. 
```markdown
# AskForUserName
- {userName:firstName=vishwac} {userName:lastName=kannan}
- I'm {userName:firstName=vishwac}
- my first name is {userName:firstName=vishwac}
- {userName=vishwac} is my name

> This definition is same as including an explicit defintion for userName with 'lastName', 'firstName' as roles
> $userName : simple Roles=lastName, firstName
```

## Phrase List features

You can enhance LUIS understanding of your model using [PhraseLists](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-tutorial-interchangeable-phrase-list).

You can describe Phrase List entities using the following notation:
$\<entityName\>:PhraseList[!interchangeable]
    - \<synonym1\>
    - \<synonym2\>

Here's an example of a phrase list definition:

```markdown
$Want:PhraseList
    - require, need, desire, know

> You can also break up the phrase list values into an actual list

$Want:PhraseList
    - require
	- need
	- desire
	- know
```
By default synonyms are set to be **not interchangeable** (matches with the portal experience). You can optionally set the synonyms to be **interchangeable** as part of the definition. Here's an example:

```markdown
$question:PhraseList interchangeable
    - are you
    - you are
```

## Patterns
Patterns are a new feature in LUIS that allows you to define a set of rules that augment the machine learned model. You can define patterns in the .lu file simply by defining an entity in an utterance without a labelled value. 

As an example, this would be treated as a pattern with alarmTime set as a Pattern.Any entity type:
```markdown
# DeleteAlarm
- delete the {alarmTime} alarm
``` 
This example would be treated as an utterance since it has a labelled value with 7AM being the labelled value for entity alarmTime:
```markdown
# DeleteAlarm
- delete the {alarmTime=7AM} alarm
```

Note: By default any entity that is left un-described in a pattern will be mapped to Pattern.Any entity type.

## Roles
[Roles](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-roles) are named, contextual subtypes of an entity used only in [Patterns](#Patterns).

For example, in the utterance buy a ticket from New York to London, both New York and London are cities but each has a different meaning in the sentence. New York is the origin city and London is the destination city.

Roles give a name to those differences:

|Entity	|Role	|Purpose|
|-------|-------|-------|
|Location	|origin	|where the plane leaves from|
|Location	|destination	|where the plane lands|

In patterns, you can use roles using the {\<entityName\>:\<roleName\>} notation. Here's an example: 

```markdown
# getUserName
- call me {name:userName}
- I'm {name:userName}
- my name is {name:userName}
```

You can define multiple roles for an entity in patterns and the ludown parser will do rest! 

```markdown
# BookFlight
> roles can be specified for list entity types as well - in this case fromCity and toCity are added as roles to the 'city' list entity defined further below
- book flight from {city:fromCity} to {city:toCity}
- [can you] get me a flight from {city:fromCity} to {city:toCity}
- get me a flight to {city:toCity}
- I need to fly from {city:fromCity}

$city:Seattle=
- Seattle
- Tacoma
- SeaTac
- SEA

$city:Portland=
- Portland
- PDX
```

See this [example](..\examples\patterns.lu) for more details. 

## Question and Answer pairs
.lu file (and the parser) supports question and answer definitions as well. You can this notation to describe them:

```markdown
# ? Question
    [list of question variations]
    ```markdown
    Answer
    ```
```

Here's an example of question and answer definitions. The LUDown tool will automatically separate question and answers into a qnamaker JSON file that you can then use to create your new [QnaMaker.ai](http://qnamaker.ai) knowledge base article.

```markdown
> # QnA Definitions
> 
### ? who is the ceo?
	```markdown
	You can change the default message if you use the QnAMakerDialog. 
	See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
	```

### ? How do I programmatically update my KB?
	```markdown
	You can use our REST apis to manage your KB. 
	\#1. See here for details: https://westus.dev.cognitive.microsoft.com/docs/services/58994a073d9e04097c7ba6fe/operations/58994a073d9e041ad42d9baa
	```
```

You can add multiple questions to the same answer by simply adding variations to questions:

```markdown
### ? Who is your ceo?
- get me your ceo info
	```markdown
		Vishwac
	```
```

## External references

Few different references are supported in the .lu file. These follow Markdown link syntax.
- Reference to another .lu file via `\[link name](\<.lu file name\>)`. Reference can be an absolute path or a relative path from the containing .lu file.
- Reference to a folder with other .lu files is supported through 
	- `\[link name](\<.lu file path\>/*)` - will look for .lu files under the specified absolute or relative path
	- `\[link name](\<.lu file path\>/**)` - will recursively look for .lu files under the specified absolute or relative path including sub-folders.
- Reference to URL for QnAMaker to ingest during KB creation via `\[link name](\<URL\>)`
- You can also add references to utterances defined in a specific file under an Intent section or as QnA pairs.
	- `\[link name](\<.lu file path\>#\<INTENT-NAME\>) will find all utterances found under \<INTENT-NAME\> in the .lu file and add them to the list of utterances where this reference is specified
	- `\[link name](\<.lu file path\>#*utterances*) will find all utterances in the .lu file and add them to the list of utterances where this reference is specified
    - `\[link name](\<.lu file path\>#*patterns*) will find all patterns in the .lu file and add them to the list of utterances where this reference is specified
	- `\[link name](\<.lu file path\>#*utterancesAndPatterns*) will find all utterances and patterns in the .lu file and add them to the list of utterances where this reference is specified
	- `\[link name](\<.lu file path\>#?) will find questions from all QnA pairs defined in the .lu file and add them to the list of utterances where this reference is specified.
	- `\[link name](\<.lu folder\>/*#?) will find all questions from all .lu files in the specified folder and add them to the list of utterances where this reference is specified. 


Here's an example of those references: 

```markdown
[QnaURL](https://docs.microsoft.com/en-in/azure/cognitive-services/qnamaker/faqs)

[none intent definition](./none.lu)

> Look for all .lu files under a path
[Cafe model](./cafeModels/*)

> Recursively look for .lu files under a path including sub-folders.
[Chit chat](../chitchat/resources/**)
```

Take a look at these additional example .lu files to learn more about external references.

- [Simple external reference](../examples/luFileReference1.lu)
- [References with wild cards](../examples/luFileReference2.lu)
- [Deep reference to an intent](../examples/luFileReference3.lu)
- [Deep reference to QnA questions](../examples/luFileReference4.lu)
- [Deep reference to QnA questions with wild card](../examples/luFileReference5.lu)

## QnAMaker Filters
Filters in QnA Maker are simple key value pairs that can be used to narrow search results, boost answers and store context. You can add filters using the following notation: 
```markdown
***Filters:***
- name = value
- name = value 
```

Here's an example usage: 
```markdown
### ? Where can I get coffee? 
- I need coffee

**Filters:**
- location = seattle

    ```markdown
    You can get coffee in our Seattle store at 1 pike place, Seattle, WA
    ```

### ? Where can I get coffee? 
- I need coffee

**Filters:**
- location = portland

    ```markdown
    You can get coffee in our Portland store at 52 marine drive, Portland, OR
    ```
```

## QnA Maker alterations
QnA Maker supports [word alterations](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices#use-synonyms) as a way to improve the likelihood that a given user query is answered with an appropriate response. You can use this feature to add synonyms to keywords that take different form. 

You can describe word alterations/ synonyms list in .lu files using the following notation - 
```markdown
$<synonym word>:qna-alteration=
- <list of synonyms>
```

Here's an example: 
```markdown
$botframework : qna-alterations=
- bot framework
- Microsoft bot framework
```

## QnA Maker pdf file ingestion
QnA Maker also supports ingesting pdf files during KB creation. You can add files for QnA maker to ingest using the URL reference scheme. If the URI's content-type is not text/html, then the ludown parser will add it to files collection for QnA Maker to ingest. 

```markdown
[SurfaceManual.pdf](https://download.microsoft.com/download/2/9/B/29B20383-302C-4517-A006-B0186F04BE28/surface-pro-4-user-guide-EN.pdf)
```
## Adding comments
You can add comments to your .lu document by prefixing the comment with >. Here's an example: 

```markdown
> This is a comment and will be ignored

# Greeting
- hi
- hello
```

## Adding ID to QnA maker Questions
You can add Id to question in QnA ludown file using the decorator '@', necessary for the next feature which is add followup prompts in QnA Ludown file

```markdown
### ? Where can I get coffee? 
@ 1 
- I need coffee
- I want coffee

```


## Adding Follow Up prompts to QnA maker Questions
You can add follow up prompts to QnA questions in QnA ludown file using the decorator '%'. Please see the usage below

```markdown
### ? Where can I get coffee? 
@ 1 
- I need coffee
- I want coffee
QnA ans here in markdown(same as before)

### ? Where can I get Tea? 
@ 2 
- I need tea
- I want tea
% 1     (ID of the question(s) goes here and it will be added as followup prompt to this question)
% 3
% 4
QnA ans here in markdown (same as before)

```

Note:- 
1) You cannot have circular references eg:-   1->2->1 , 1->2->3->1,  etc. where 1,2... are the QnA Question pairs 
2) You can have chain like 1->2->3->4......
2) You can use Replace knowledgebase QnA maker API call and you will see the changes reflected Link:-  https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/knowledgebases_publish