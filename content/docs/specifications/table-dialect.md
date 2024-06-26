---
title: Table Dialect
sidebar:
  order: 3
---

<table>
  <tr>
    <th>Authors</th>
    <td>Rufus Pollock, Paul Walsh, Adam Kariv, Evgeny Karev, Peter Desmet, Data Package Working Group</td>
  </tr>
  <tr>
    <th>Profile</th>
    <td><a href="/profiles/2.0/tabledialect.json">https://datapackage.org/profiles/2.0/tabledialect.json</a></td>
  </tr>
</table>

Table Dialect describes how tabular data is stored in a file. It supports delimited text files like CSV, semi-structured formats like JSON and YAML, and spreadsheets like Microsoft Excel. The specification is designed to be expressible as a single JSON-compatible descriptor.

## Language

The key words `MUST`, `MUST NOT`, `REQUIRED`, `SHALL`, `SHALL NOT`, `SHOULD`, `SHOULD NOT`, `RECOMMENDED`, `MAY`, and `OPTIONAL` in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Introduction

Table Dialect defines set of properties that can be used by data producers and data consumers to ensure data interoperability in various [Tabular Data](../glossary/#tabular-data) formats such as CSV, JSON, or Excel. The main goal of this specification is to define a common language for defining tabular data dialects. It is not expected that all the properties are supported by all the Data Package implementations. An implementation `MUST` choose the most suitable strategy for communicating to the users if some relevant feature is not supported.

Table Dialect is useful for programmes which might have to deal with multiple dialects of tabular files, but which can rely on being told out-of-band which dialect will be used in a given input stream. This reduces the need for heuristic inference of dialects, and simplifies the implementation of readers, which must juggle dialect inference, schema inference, unseekable input streams, character encoding issues, and the lazy reading of very large input streams.

:::note[Backward Compatibility]
Table Dialect supersedes [CSV Dialect](https://specs.frictionlessdata.io/csv-dialect/). Except for `caseSensitiveHeader` and `csvddfVersion` which are removed, all CSV Dialect properties are included in Table Dialect and can be used without changes.
:::

## Descriptor

Table Dialect descriptor `MUST` be a descriptor as per [Descriptor](../glossary/#descriptor) definition. A list of standard properties that can be included into a descriptor is defined in the [Properties](#properties) section.

An example of a Table Dialect descriptor:

```json
{
  "header": false,
  "delimiter": ";",
  "quoteChar": "'"
}
```

## Properties

Table dialect defines individual properties that regulates data producing and consuming for different groups of targeted formats, as well, as general rules that applies for multiple data format groups. Note, that property grouping is only illustrative, if a property is suitable for a format group that is not mentioned in this specification it still can be used as far as the property definition and semantics are respected.

A property `MUST` be ignored if it is no applicable for an arbitrary data format. For example, SQL databases do not have a concept of a header row.

For the sake of simplicity, most of examples are written in the CSV data format. For example, this data file without providing any Table Dialect properties:

```csv
id,name
1,apple
2,organe
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple"}
{id: 2, name: "orange"}
```

### General

General properties are format-agnostic. Usually, there are useful for defining dialects for delimiter-based and spreadsheet-based formats like CSV or Excel.

#### `$schema`

A root level Table Dialect descriptor `MAY` have a `$schema` property that `MUST` be a profile as per [Profile](../glossary/#profile) definition that `MUST` include all the metadata constraints required by this specification.

The default value is `https://datapackage.org/profiles/1.0/tabledialect.json` and the recommended value is `https://datapackage.org/profiles/2.0/tabledialect.json`.

#### `header`

A Table Dialect descriptor `MAY` have the `header` property that `MUST` be boolean with default value `true`. This property indicates whether the file includes a header row. If `true` the first row in the file `MUST` be interpreted as a header row, not data.

For example, this data file:

```csv
1,apple
2,orange
```

With this dialect definition:

```json
{
  "header": false
}
```

`SHOULD` output this data:

```javascript
{field1: 1, field2: "apple"}
{field1: 2, field2: "orange"}
```

Where `field1` and `field2` names are implementation-specific and used here only for illustrative purpose.

#### `headerRows`

A Table Dialect descriptor `MAY` have the `headerRows` property that `MUST` be an array of positive integers starting from 1 with default value `[1]`. This property specifies the row numbers for the header. It is `RECOMMENDED` to be used for multiline-header files.

For example, this data file:

```csv
fruit
id,name
1,apple
2,orange
```

With this dialect definition:

```json
{
  "headerRows": [1, 2]
}
```

`SHOULD` output this data:

```javascript
{"fruit id": 1, "fruit name": "apple"}
{"fruit id": 2, "fruit name": "orange"}
```

#### `headerJoin`

A Table Dialect descriptor `MAY` have the `headerJoin` property that `MUST` be a string with default value `" "`. This property specifies how multiline-header files have to join the resulting header rows.

For example, this data file:

```csv
fruit
id,name
1,apple
2,orange
```

With this dialect definition:

```json
{
  "headerRows": [1, 2],
  "headerJoin": "-"
}
```

`SHOULD` output this data:

```javascript
{"fruit-id": 1, "fruit-name": "apple"}
{"fruit-id": 2, "fruit-name": "orange"}
```

#### `commentRows`

A Table Dialect descriptor `MAY` have the `commentRows` property that `MUST` be an array of positive integers starting from 1; undefined by default. This property specifies what rows have to be omitted from the data.

For example, this data file:

```csv
id,name
#fruits
1,apple
2,orange
```

With this dialect definition:

```json
{
  "commentRows": [2]
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple"}
{id: 2, name: "orange"}
```

#### `commentChar`

A Table Dialect descriptor `MAY` have the `commentChar` property that `MUST` be a string of one or more characters; undefined by default. This property specifies what rows have to be omitted from the data based on the row's first characters.

For example, this data file:

```csv
id,name
#fruits
1,apple
2,orange
```

With this dialect definition:

```json
{
  "commentChar": "#"
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple"}
{id: 2, name: "orange"}
```

### Delimited

Delimited formats is a group of textual formats such as CSV and TSV.

#### `delimiter`

A Table Dialect descriptor `MAY` have the `delimiter` property that `MUST` be a string; with default value `,` (comma). This property specifies the character sequence which separates fields in the data file.

For example, this data file:

```csv
id|name
1|apple
2|orange
```

With this dialect definition:

```json
{
  "delimiter": "|"
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple"}
{id: 2, name: "orange"}
```

#### `lineTerminator`

A Table Dialect descriptor `MAY` have the `lineTerminator` property that `MUST` be a string; with default value `\r\n`. This property specifies the character sequence which terminates rows.

For example, this data file:

```csv
id,name;1,apple;2,orange
```

With this dialect definition:

```json
{
  "lineTerminator": ";"
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple"}
{id: 2, name: "orange"}
```

#### `quoteChar`

A Table Dialect descriptor `MAY` have the `quoteChar` property that `MUST` be a string of one character length with default value `"` (double quote). This property specifies a character to use for quoting in case the `delimiter` needs to be used inside a data cell.

For example, this data file:

```csv
id,name
1,'apple,fruits'
2,'orange,fruits'
```

With this dialect definition:

```json
{
  "quoteChar": "'"
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple,fruits"}
{id: 2, name: "orange,fruits"}
```

#### `doubleQuote`

A Table Dialect descriptor `MAY` have the `doubleQuote` property that `MUST` be boolean with default value `true`. This property controls the handling of `quoteChar` inside data cells. If true, two consecutive quotes are interpreted as one.

For example, this data file:

```csv
id,name
1,"apple""fruits"
2,"orange""fruits"
```

With this dialect definition:

```json
{
  "doubleQuote": true
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: 'apple"fruits'}
{id: 2, name: 'orange"fruits'}
```

#### `escapeChar`

A Table Dialect descriptor `MAY` have the `escapeChar` property that `MUST` be a string of one character length; undefined by default. This property specifies a one-character string to use for escaping, for example, `\`, mutually exclusive with `quoteChar`.

For example, this data file:

```csv
id,name
1,apple|,fruits
2,orange|,fruits
```

With this dialect definition:

```json
{
  "escapeChar": "|"
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple,fruits"}
{id: 2, name: "orange,fruits"}
```

#### `nullSequence`

A Table Dialect descriptor `MAY` have the `nullSequence` property that `MUST` be a string; undefined by default. This property specifies specifies the null sequence, for example, `\N`.

For example, this data file:

```csv
id,name
1,apple
2,NA
```

With this dialect definition:

```json
{
  "nullSequence": "NA"
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple"}
{id: 2, name: null}
```

#### `skipInitialSpace`

A Table Dialect descriptor `MAY` have the `skipInitialSpace` property that `MUST` be boolean with default value `false`. This property specifies how to interpret whitespace which immediately follows a delimiter; if `false`, it means that whitespace immediately after a delimiter is treated as part of the following field.

For example, this data file:

```csv
id, name
1, apple
2, orange
```

With this dialect definition:

```json
{
  "skipInitialSpace": true
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple"}
{id: 2, name: "orange"}
```

### Structured

Structured formats is a group of structured or semi-structured formats such as JSON and YAML.

#### `property`

A Table Dialect descriptor `MAY` have the `property` property that `MUST` be a string; undefined by default. This property specifies where a data array is located in the data structure.

For example, this data file:

```json
{
  "rows": [
    { "id": 1, "name": "apple" },
    { "id": 2, "name": "orange" }
  ]
}
```

With this dialect definition:

```json
{
  "property": "rows"
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple"}
{id: 2, name: "orange"}
```

#### `itemType`

A Table Dialect descriptor `MAY` have the `itemType` property that `MUST` be a string with value `array` or `object`; undefined by default. This property specifies whether the data `property` contains an array of arrays or an array of objects.

For example, this data file:

```json
[
  ["id", "name"],
  [1, "apple"],
  [2, "orange"]
]
```

With this dialect definition:

```json
{
  "itemType": "array"
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple"}
{id: 2, name: "orange"}
```

#### `itemKeys`

A Table Dialect descriptor `MAY` have the `itemKeys` property that `MUST` be array of strings; undefined by default. This property specifies the way of extracting rows from data arrays with `itemType` is `object`.

For example, this data file:

```json
[
  { "name": "apple", "id": 1, "count": 2 },
  { "id": 2, "name": "orange", "count": 5 }
]
```

With this dialect definition:

```json
{
  "itemKeys": ["id", "name"]
}
```

`SHOULD` output this data:

```javascript
{id: 1, name: "apple"}
{id: 2, name: "orange"}
```

### Spreadsheet

Spreadsheet formats is a group of sheet-based formats such as Excel or ODS.

#### `sheetNumber`

A Table Dialect descriptor `MAY` have the `sheetNumber` property that `MUST` be an integer with default value `1`. This property specifies a sheet number of a table in the spreadsheet file.

For example, this data file:

```txt
Sheet 1
Sheet 2
```

With this dialect definition:

```json
{
  "sheetNumber": 2
}
```

`SHOULD` output the data from the second sheet.

#### `sheetName`

A Table Dialect descriptor `MAY` have the `sheetName` property that `MUST` be a string; undefined by default. This property specifies a sheet name of a table in the spreadsheet file.

For example, this data file:

```txt
Sheet 1
Sheet 2
```

With this dialect definition:

```json
{
  "sheetName": "Sheet 2"
}
```

`SHOULD` output the data from the second sheet.

### Database

Database formats is a group of formats accessing data from databases like SQLite.

#### `table`

A Table Dialect descriptor `MAY` have the `table` property that `MUST` be a string; undefined by default. This property specifies a name of the table in the database.

For example, the database with the tables below:

```txt
table1
table2
```

With this dialect definition:

```json
{
  "table": "table2"
}
```

`SHOULD` output the data from the second table.

## Example

An example of a well-defined Table Dialect descriptor for a CSV format:

```json
{
  "header": false,
  "commentChar": "#"
  "delimiter": ";",
  "doubleQuote": true,
  "lineTerminator": "\r\n",
  "quoteChar": "\"",
  "skipInitialSpace": true,
}
```

## Excluded

Table Dialect has nothing to do with the names, contents or types of the headers or data within the CSV file, only how it is formatted. However, CSV Dialect does allow the presence or absence of a header to be specified, similarly to [RFC4180](http://www.ietf.org/rfc/rfc4180.txt).

Table Dialect is also orthogonal to the character encoding used in the CSV file. Note that it is possible for files in CSV format to contain data in more than one encoding.

## References

Some related work can be found in [similar JSON format](http://panda.readthedocs.org/en/latest/api.html#data-uploads) and in Python's [PEP 305](http://www.python.org/dev/peps/pep-0305/).
