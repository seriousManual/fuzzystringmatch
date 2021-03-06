# fuzzystringmatch
fuzzystringmatch is a small library that creates a in-memory index for a fast and fuzzy lookup of search terms.

## Example
The following country names have been put into the index:
````
United States of America
United Kingdom of Great Britain
Germany
France
Japan
````

When searching for the term `Unted`, the entries `United States of America` and `United Kingdom of Great Britain` will be returned.
(see the [example section](https://github.com/cookiefactory/fuzzystringmatch/tree/master/example))

## Installation
````bash
npm install fuzzystringmatch
````

## Setup
fuzzystringmatch consists of 3 parts that work together:

### Subject
Each Subject instance describes one index entry (e.g. one of the country names in the example above).
````javascript
var mySubject = new Subject('United States of America')
````

If you want to decorate the Subject with additional meta data (e.g. an external ranking factory that will be used to sort the list of entries),
a child class of Subject can be created:

````javascript
var Subject = require('fuzzystringmatch').Subject

class RankedSubject extends Subject {
    constructor(term, rank) {
        super(term) //dont forget the parent constructor call
        this._rank = rank
    }
    
    getRank() {
        return this._rank
    }
}

var mySubject = new RankedSubject('Germany', 42)
````

### Digester
The Digester is the instance that receives all the base data (e.g. the country names from the example above) and build up an index.

The Digester can be supplied with a raw string:
````javascript
var digester = new require('fuzzystringmatch').Digester()

digester.feed('United States of America')
````
or can be supplied with an instance of Subject:
````javascript
var digester = new require('fuzzystringmatch').Digester()

digester.feed(new Subject('United States of America'))
````

### Matcher
The Matcher uses the index created by the digester to look up search terms:
````javascript
var digester = new require('fuzzystringmatch').Digester()
var matcher = new require('fuzzystringmatch').Matcher(digester)

digester.feed('France')
digester.feed('Japan')

console.log(matcher.match('jpan'))
````

The result of the `match` call is a list of `ResultEntry` instances.
Each `ResultEntry` represents a match regarding the search term and holds a reference to the subject from the index.

## Complete Example
````javascript
var fuzzyStringmatch = require('fuzzystringmatch')

var digester = new fuzzyStringmatch.Digester
var matcher = new fuzzyStringmatch.Matcher(digester)

digester.feed('United States of America')
digester.feed(new fuzzyStringmatch.Subject('United Kingdom of Great Britain'))
digester.feed('Germany')
digester.feed('France')
digester.feed('Japan')

matcher
    .match('grmany')
    .forEach((resultEntry) => {
        var subject = resultEntry.getSubject()
        console.log(`${subject.getTerm()}, Matchscore: ${resultEntry.getMatchRelation()}`)
    })

````

## API Reference

### Digester

#### constructor([configuration])
Creates the Digester instance

#### Digester.feed(term)
Takes a search term that will be included into the index, can be a raw String or a Subject instance

### Matcher

#### constructor(digester[, configuration])
Creates the Matcher instance, takes a digester that holds the index that should be searched.

#### Matcher.match(term[, overallCount])
Matches a certain search term against the index.
* *term*: the raw string that should be searched
* *overallCount*: defines the maximum number of result entries, optional, defaults to 150

### Subject

#### constructor(term)
Creates the Subject instance, takes the term that should be represented by the Subject

#### Subject.getTerm()
Returns the term that is represented

### ResultEntry

#### ResultEntry.getSubject()
Returns the Subject the ResultEntry is in relation with

#### ResultEntry.getMatchRelation()
Returns the relation between the number of matched chunks and the number of searched chunks.
Acts as a quality factor for the search result.
