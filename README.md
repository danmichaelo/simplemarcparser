SimpleMarcParser
===============

[![Build Status](http://img.shields.io/travis/scriptotek/simplemarcparser.svg?style=flat-square)](https://travis-ci.org/scriptotek/simplemarcparser)
[![Coverage Status](http://img.shields.io/coveralls/scriptotek/simplemarcparser.svg?style=flat-square)](https://coveralls.io/r/scriptotek/simplemarcparser?branch=master)
[![Code Quality](http://img.shields.io/scrutinizer/g/scriptotek/simplemarcparser/master.svg?style=flat-square)](https://scrutinizer-ci.com/g/scriptotek/simplemarcparser/?branch=master)
[![StyleCI](https://styleci.io/repos/14246253/shield)](https://styleci.io/repos/14246253)
[![Latest Stable Version](http://img.shields.io/packagist/v/scriptotek/simplemarcparser.svg?style=flat-square)](https://packagist.org/packages/scriptotek/simplemarcparser)
[![Total Downloads](http://img.shields.io/packagist/dt/scriptotek/simplemarcparser.svg?style=flat-square)](https://packagist.org/packages/scriptotek/simplemarcparser)

`SimpleMarcParser` is currently a minimal MARC21/XML parser for use with `QuiteSimpleXMLElement`,
with support for the MARC21 Bibliographic, Authority and Holdings formats.

## Example:

```php
require_once('vendor/autoload.php');

use Danmichaelo\QuiteSimpleXMLElement\QuiteSimpleXMLElement,
    Scriptotek\SimpleMarcParser\Parser;

$data = file_get_contents('http://sru.bibsys.no/search/biblio?' . http_build_query(array(
	'version' => '1.2',
	'operation' => 'searchRetrieve',
	'recordSchema' => 'marcxchange',
	'query' => 'bs.isbn="0-521-43291-x"'
)));

$doc = new QuiteSimpleXMLElement($data);
$doc->registerXPathNamespaces(array(
        'srw' => 'http://www.loc.gov/zing/srw/',
        'marc' => 'http://www.loc.gov/MARC21/slim',
        'd' => 'http://www.loc.gov/zing/srw/diagnostic/'
    ));

$parser = new Parser();

$record = $parser->parse($doc->first('/srw:searchRetrieveResponse/srw:records/srw:record/srw:recordData/marc:record'));

print $record->title;

foreach ($record->subjects as $subject) {
	print $subject['term'] . '(' . $subject['system'] . ')';
}
```

# Transformation/normalization

This parser is aimed at producing machine actionable output, and does some non-reversible 
transformations to achieve this. Transformation rules expect AACR2-like records, and are
tested mainly against the Norwegian version of AACR2 (*Norske katalogregler*), but might
work well with other editions as well.

Examples:

 - `title` is a combination of 300 $a and $b, separated by ` : `.
 - `year` is an integer extracted from 260 $c by extracting the first four digit integer found
   (`c2013` → `2013`, `2009 [i.e. 2008]` → `2009` (this might be a bit rough…))
 - `pages` is an integer extracted from 300 $a. The raw value, useful for e.g. non-verbal content,
   is stored in `extent`
 - `creators[].name` are transformed from '<Lastname>, <Firstname>' to '<Firstname> <Lastname>'

# Form and material

Form and material is encoded in the leader and in control fields 006, 007 and 008.
Encoding this information in a format that makes sense is a *work-in-progress*.

Electronic and printed material is currently distinguished using the boolean valued `electronic` key.

Printed book:

```json
{
	"material": "book",
	"electronic": false
}
```

Electronic book:

```json
{
	"material": "book",
	"electronic": true
}
```
