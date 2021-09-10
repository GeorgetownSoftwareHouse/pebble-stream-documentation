# pebble-stream-documentation
Pebble Stream Documentation



Directives are either [transformative](#transformative-directives) (`pebblestream:from(Sheet1)`) or [annotative](#annotative-directives) (`pebblestream:sorted:asc`).

# Transformative directives
Transformative directives look like this:
```
pebblestream:from(Accounts)
pebblestream:filter("Amount", ">0")
pebblestream:group-sum("Account ID", "Amount")
```
and go in a comment in the top-left cell of a sheet.  

<details>
  <summary>
  Multi-directive example
  </summary>

  [Multi-directive example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Multi-directive%20example.xlsx)
  ```
Purchases           
From To Amount Verified
0    1  30     TRUE
5    1  10     FALSE
9    2  20     TRUE
4    1  50     TRUE
5    2  90     FALSE

Transactions
Amount From To Verified
60     9    2  TRUE
-10    5    1  TRUE

Total
pebblestream:from(Purchases, Transactions)
pebblestream:filter("Verified")
pebblestream:sort("To")
pebblestream:group-sum("To", "Amount")
To Amount
1  70
2  80
  ```
</details>

The directives are executed in this order:  
1. [from](#from)
1. [cartesian-product](#cartesian-product)
1. [join](#join)
1. [outer-join](#outer-join)
1. [filter](#filter)
1. [sort](#sort)
1. [group](#group)
    * [coalesce](#coalesce)
    * [sum](#sum)
	* [ratio](#ratio)
    * [mult](#mult)
    * [equal](#equal)
    * [regex](#regex)
1. [group-unique](#group-unique)
1. [group-sum](#group-sum)
1. [group-ratio](#group-ratio)
1. [header-map](#header-map)
1. [split](#split)

Column arguments are header value strings in double quotes (`"Account"`, `"Amount"`).  
Sheet arguments are either in excel notation (`Sheet1`, `'2010''s Data'`) or double quotes (`"Sheet1"`, `"2010's Data"`).

## from
`pebblestream:from(Sheet1, ...)`  
Stitches the sheets together (headers only appear once). Only the columns with headers the sheets have in common are included.  
<details>
  <summary>Example</summary>
  
  [From example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/From%20example.xlsx)
  ```
Regional
Name ID Account
Alex 17 Checking
John 2  Savings

Local
ID Account
3  Checking
64 Brokerage
5  Brokerage

District
Name ID Balance Last Name
John 4  545     Jones

IDs
pebblestream:from(Regional, Local, District)
ID
17
2
3
64
5
4
  ```
</details>

_O(cells)_

## cartesian-product
`pebblestream:cartesian-product(Sheet1, ...)`  
Output will have a row for every combination of rows in the input sheets.  
<details>
  <summary>Example</summary>
  
  [Cartesian-product example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Cartesian-product%20example.xlsx)
  ```
Customers
Customer ID Balance
c0          45
c1          3
c2          0

Discounts
Discount ID Percentage
d0          0.5
d1          0.1

Customers x Discounts
pebblestream:from(Customers)
pebblestream:cartesian-product(Discounts)
Customer ID Balance Discount ID Percentage
c0          45      d0          0.5
c0          45      d1          0.1
c1          3       d0          0.5
c1          3       d1          0.1
c2          0       d0          0.5
c2          0       d1          0.1
  ```
</details>

_O(rows^sheets)_

## join
`pebblestream:join(SheetRight, [ ["Key Left", "Key Right",] ...])`  
Does an inner join with SheetRight keyed on either the header names they have in common or the arguments, which come in pairs of first the left's header name and then the right's header name.  
<details>
  <summary>Example 1</summary>
  
  [Join example 1.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Join%20example%201.xlsx)
  ```
Customers
Customer ID Name
0           John
1           James
2           Alex

Orders
Order ID Amount Customer ID
0        5      0
1        1      0
2        8      2
3        6      1

Customers + Orders
pebblestream:from(Customers)
pebblestream:join(Orders)
Name  Customer ID Order ID Amount
John  0           0        5
John  0           1        1
James 1           3        6
Alex  2           2        8
  ```
</details>
<details>
  <summary>Example 2</summary>
  
  [Join example 2.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Join%20example%202.xlsx)
  ```
Transfers
From To Amount
A0   A1 24
A0   A1 576
A2   A1 13
A1   A0 42

Taxable
Account From Account To Taxable
A0           A1         TRUE
A0           A2         FALSE
A1           A0         FALSE
A1           A2         TRUE
A2           A0         FALSE
A2           A1         TRUE

Transfers + Taxable
pebblestream:from(Transfers)
pebblestream:join(Taxable, "From", "Account From", "To", "Account To")
Amount From To Taxable
24     A0   A1 TRUE
576    A0   A1 TRUE
13     A2   A1 TRUE
42     A1   A0 FALSE
  ```
</details>

_O(cells)_

## outer-join
`pebblestream:outer-join(SheetRight, [ ["Key Left", "Key Right",] ...])`  
Does a left outer join with SheetRight keyed on either the header names they have in common or the arguments, which come in pairs of first the left's header name and then the right's header name.  
<details>
  <summary>Example 1</summary>
  
  [Outer-join example 1.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Outer-join%20example%201.xlsx)
  ```
Customers
Customer ID Name
0           John
1           James
2           Alex
3           Linda

Orders
Order ID Amount Customer ID
0        5      0
1        1      0
2        8      2
3        6      1
4        2      7

Customers + Orders
pebblestream:from(Customers)
pebblestream:outer-join(Orders)
Name  Customer ID Order ID Amount
John  0           0        5
John  0           1        1
James 1           3        6
Alex  2           2        8
Linda 3
  ```
</details>
<details>
  <summary>Example 2</summary>
  
  [Outer-join example 2.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Outer-join%20example%202.xlsx)
  ```
Transfers
From To Amount
A0   A1 24
A0   A1 576
A2   A1 13
A1   A0 42
A3   A0 78

Taxable
Account From Account To Taxable
A0           A1         TRUE
A0           A2         FALSE
A1           A0         FALSE
A1           A2         TRUE
A2           A0         FALSE
A2           A1         TRUE
A1           A3         FALSE

Transfers + Taxable
pebblestream:from(Transfers)
pebblestream:outer-join(Taxable, "From", "Account From", "To", "Account To")
Amount From To Taxable
24     A0   A1 TRUE
576    A0   A1 TRUE
13     A2   A1 TRUE
42     A1   A0 FALSE
78     A3   A0
  ```
</details>

_O(cells)_

## filter
`pebblestream:filter("Active")`  
Only includes rows where `"Active"` is true.

`pebblestream:filter("Amount", ">0", "Color", "blue", "Amount", "<="&"Balance", ...)`  
Only includes rows where `"Amount"` is greater than 0, `"Color"` is "blue", and `"Amount"` is less than or equal to the value in that row's `"Balance"` column.

If `filter()` appears multiple times, it includes rows where any of the `filter()` directives qualify (OR).  
<details>
  <summary>Simple example</summary>
  
  [Filter simple example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Filter%20simple%20example.xlsx)
  ```
Accounts
ID Verified Balance
7  TRUE     0
1  TRUE     1
5  FALSE    54
17 TRUE     35

Verified
pebblestream:from(Accounts)
pebblestream:filter("Verified")
ID
7
1
17
  ```
</details>
<details>
  <summary>Complex example</summary>
  
  [Filter complex example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Filter%20complex%20example.xlsx)
  ```
  Heights
Height Age Age last transaction
190    24  18
300    7   7
100    3   3
40     1   2
400    16  16
167    40  38
50     22  20

Unusual
pebblestream:from(Heights)
pebblestream:filter("Height", ">200")
pebblestream:filter("Height", "<60", "Age", ">3")
pebblestream:filter("Age last transaction", ">"&"Age")
Height Age Age last transaction
300    7   7
40     1   2
400    16  16
50     22  20
  ```
</details>

_O(cells)_

## sort
`pebblestream:sort("Last name":desc, "First name":asc, ...)`  
Sorts the rows by `"Last name"` descending with ties broken by `"First name"` ascending. Default is ascending.  
Can't be used with group.  
<details>
  <summary>Simple example</summary>
  
  [Sort simple example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Sort%20simple%20example.xlsx)
  ```
Accounts
ID Balance
5  44
1  40
8  848
3  12

Sorted
pebblestream:from(Accounts)
pebblestream:sort("ID")
ID Balance
1 40
3 12
5 44
8 848
  ```
</details>
<details>
  <summary>Complex example</summary>
  
  [Sort complex example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Sort%20complex%20example.xlsx)
  ```
Transactions
First Last      Date
John  Jones     9/2/2020
Adam  Jones     1/1/2000
Cathy Albertson 5/7/1990
John  Jones     5/4/2013

Sorted
pebblestream:from(Transactions)
pebblestream:sort("Last", "First", "Date":desc)
First Last      Date
Cathy Albertson 5/7/1990
Adam  Jones     1/1/2000
John  Jones     9/2/2020
John  Jones     5/4/2013
  ```
</details>

_O(cells*log(cells))_

## group
`pebblestream:group("Account ID", "Date", ...)`  
All rows with matching `"Account ID"` and `"Date"` values are put next to each other.  
Any of the following directives are calculated relative to their group:
  * [sum](#sum)
  * [mult](#mult)
  * [equal](#equal)
  * [regex](#regex)

Can't be used with sort, group-unique, group-sum, or group-ratio.  
<details>
  <summary>Multi-directive example</summary>
  
  [Group multi-directive example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Group%20multi-directive%20example.xlsx)
  ```
Transactions
Account ID Account User # Amount Coupon factor IP Address  Tags
1          2              894    0.9           127.0.38.9  home verified
1          1              45981  0.8           127.0.5.8   office verified
1          1              15     0.95          127.0.5.8   home
1          2              5958   1             127.0.48.2  verified home
2          1              255    0.7           127.0.59.12 verified mobile

Grouped
pebblestream:from(Transactions)
pebblestream:group("Account ID", "Account User #")
pebblestream:sum("Amount")
pebblestream:mult("Coupon factor")
pebblestream:equal("IP Address", "Same IP?")
pebblestream:regex("Tags", ".*\\bverified\\b.*", "Verified?")
pebblestream:coalesce()
Account ID Account User # Amount Coupon factor Same IP? Verified?
1          2              6852   0.9           FALSE    TRUE
1          1              45996  0.76          TRUE     FALSE
2          1              255    0.7           TRUE     TRUE
  ```
</details>
<details>
  <summary>Solo example</summary>
  
  [Group solo example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Group%20solo%20example.xlsx)
  ```
Logins
Account ID Account User # Date
1          1              1/1/2015
3          1              5/6/2015
2          1              6/8/2015
1          2              2/20/2018
1          1              3/15/2020
2          1              4/15/2020

Logins by user
pebblestream:from(Logins)
pebblestream:group("Account ID", "Account User #")
Account ID Account User # Date
1          1              1/1/2015
1          1              3/15/2020
3          1              5/6/2015
2          1              6/8/2015
2          1              4/15/2020
1          2              2/20/2018

Users
pebblestream:from(Logins)
pebblestream:group("Account ID", "Account User #")
pebblestream:coalesce()
Account ID Account User #
1          1
3          1
2          1
1          2
  ```
</details>

_O(cells)_

### coalesce
`pebblestream:coalesce()`  
After group calculations are performed, coalesce each group into a single row.  
Requires [group](#group).  
See "Group solo example" for an example.  
_O(1)_

### sum
`pebblestream:sum("Amount"[, "Result"])`  
The `"Amount"` rows are summed together. If `"Result"` is provided, a new column called `"Result"` is created with the value. Otherwise, the `"Amount"` column is updated.  
Requires [group](#group).  
<details>
  <summary>Example</summary>
  
  [Sum example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Sum%20example.xlsx)
  ```
Transactions
First Last      Transfer
John  Jones     -60
Adam  Jones     186
Cathy Albertson 48794
John  Jones     545
Cathy Albertson 8777

Totals
pebblestream:from(Transactions)
pebblestream:group("Last", "First")
pebblestream:sum("Transfer")
pebblestream:coalesce()
First Last      Transfer
John  Jones     485
Adam  Jones     186
Cathy Albertson 57571
  ```
</details>

_O(cells)_

### mult
`pebblestream:mult("Factor"[, "Result"])`  
The `"Factor"` rows are multiplied together. If `"Result"` is provided, a new column called `"Result"` is created with the value. Otherwise, the `"Factor"` column is updated.  
Requires [group](#group).  
<details>
  <summary>Example</summary>
  
  [Mult example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Mult%20example.xlsx)
  ```
Discounts
Discount ID Account ID Discount
1           1          0.9
2           2          0.8
3           1          0.95

Account discounts
pebblestream:from(Discounts)
pebblestream:group("Account ID")
pebblestream:mult("Discount")
pebblestream:coalesce()
Account ID Discount
1          0.855
2          0.8
  ```
</details>

_O(cells)_

### equal
`pebblestream:equal("Amount"[, "Result"])`  
The `"Amount"` rows are checked for whether they are all equal. If `"Result"` is provided, a new column called `"Result"` is created with the value. Otherwise, the `"Amount"` column is updated.  
Requires [group](#group).  
<details>
  <summary>Example</summary>
  
  [Equal example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Equal%20example.xlsx)
  ```
Logins
Account ID Account User # IP Address
1          2              127.0.38.9
1          1              127.0.5.8
1          1              127.0.5.8
1          2              127.0.48.2
2          1              127.0.59.12

Same IP
pebblestream:from(Logins)
pebblestream:group("Account ID", "Account User #")
pebblestream:equal("IP Address", "Same IP?")
pebblestream:coalesce()
Account ID Account User # Same IP?
1          2              FALSE
1          1              TRUE
2          1              TRUE
  ```
</details>

_O(cells)_

### ratio
`pebblestream:ratio("Amount"[, "Result"])`  
The `"Amount"` rows are summed together and the ratio of each row compared to the total calculated. If `"Result"` is provided, a new column called `"Result"` is created with the value. Otherwise, the `"Amount"` column is updated.  
Requires [group](#group).  
Incompatible with [coalesce](#coalesce).  
<details>
  <summary>Example</summary>
  
  [Ratio example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Ratio%20example.xlsx)
  ```
Transactions
First Last      Value
John  Jones     3
Adam  Jones     5
John  Jones     1
Cathy Albertson -1
Cathy Albertson 1
John  Jones     0

Ratio
pebblestream:from(Transactions)
pebblestream:group("Last", "First")
pebblestream:ratio("Value", "Ratio")
First Ratio
John  0.75
John  0.25
John  0.0
Adam  1.0
Cathy 0
Cathy 0
  ```
</details>

_O(cells)_

### regex
`pebblestream:regex("Name", "[0-9]*MyRegex.*"[, "Result"])`  
Checks whether every row in `"Name"` matches the provided regular expression. If `"Result"` is provided, a new column called `"Result"` is created with the result. Otherwise, the `"Name"` column is updated.  
Requires [group](#group).  
<details>
  <summary>Example</summary>
  
  [Regex example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Regex%20example.xlsx)
  ```
Transactions
Account ID Account User # Amount Tags
1          2              894    home verified
1          1              45981  office verified
1          1              15     home
1          2              5958   verified home
2          1              255    verified mobile

Transactions with verify info
pebblestream:from(Transactions)
pebblestream:group("Account ID", "Account User #")
pebblestream:regex("Tags", ".*\\bverified\\b.*", "User always verified?")
Account ID Account User # Amount User always verified?
1          2              894    TRUE
1          2              5958   TRUE
1          1              45981  FALSE
1          1              15     FALSE
2          1              255    TRUE
  ```
</details>

_O(cells)_

## group-unique
`pebblestream:group-unique("Last name", "First name", ...)`  
All rows with the same `"Last name"` and `"First name"` are condensed to one and only columns `"Last name"` and `"First name"` are included.  
Can't be used with group, group-sum, or group-ratio.  
<details>
  <summary>Example</summary>
  
  [Group-unique example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Group-unique%20example.xlsx)
  ```
Transactions
First Last      Date
John  Jones     9/2/2020
Adam  Jones     1/1/2000
Cathy Albertson 5/7/1990
John  Jones     5/4/2013
Cathy Albertson 12/12/1999

Names
pebblestream:from(Transactions)
pebblestream:group-unique("Last", "First")
First Last
John  Jones
Adam  Jones
Cathy Albertson
  ```
</details>

_O(cells)_

## group-sum
`pebblestream:group-sum("Last name", "First name", ... "Amount")`  
All rows with the same `"Last name"` and `"First name"` are condensed to one with their `"Amount"`'s summed together. Only columns `"Last name"`, `"First name"`, and `"Amount"` are included.  
Can't be used with group, group-unique, or group-ratio.  
<details>
  <summary>Example</summary>
  
  [Group-sum example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Group-sum%20example.xlsx)
  ```
Transactions
First Last      Transfer
John  Jones     -60
Adam  Jones     186
Cathy Albertson 48794
John  Jones     545
Cathy Albertson 8777

Totals
pebblestream:from(Transactions)
pebblestream:group-sum("Last", "First", "Transfer")
First Last      Transfer
John  Jones     485
Adam  Jones     186
Cathy Albertson 57571
  ```
</details>

_O(cells)_

## group-ratio
`pebblestream:group-ratio("Last name", "First name", ... "Amount")`  
All rows with the same `"Last name"` and `"First name"` are put next to each other with their `"Amount"` values replaced by their `"Amount"` value divided by their group's summed `"Amount"` value (0 if sum is 0).  
Can't be used with group, group-unique, or group-sum.  
<details>
  <summary>Example</summary>
  
  [Group-ratio example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Group-ratio%20example.xlsx)
  ```
Deposits
First Last      Deposit
John  Jones     20
Adam  Jones     186
Cathy Albertson 48794
John  Jones     545
Cathy Albertson 8777

Ratio deposited
pebblestream:from(Deposits)
pebblestream:group-ratio("Last", "First", "Deposit")
First Last      Deposit
John Jones      0.03539823
John Jones      0.96460177
Adam Jones      1
Cathy Albertson 0.847544771
Cathy Albertson 0.152455229
  ```
</details>

_O(cells)_

## header-map
`pebblestream:header-map("Header Old", "Header New"[, ...])`  
`Header Old` is renamed to `Header New`, specified in pairs.  
<details>
  <summary>Example</summary>
  
  [Header-map example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Header-map%20example.xlsx)
  ```
Raw Accounts
ID Balance Last Name
0  4233    Jones
1  21      Smith
2  0       George
3  33      Jones

DB-friendly Accounts
pebblestream:from("Raw Accounts")
pebblestream:header-map("Balance", "VALUE", "Last Name", "OWNER_LAST")
ID OWNER_LAST VALUE
0  Jones      4233
1  Smith      21
2  George     0
3  Jones      33
  ```
</details>

_O(cols)_

## split
`pebblestream:split("Last name", "First name", ... SheetSplit)`  
SheetSplit in the output will be a list of sheets where each has a group of rows with the same `"Last name"` and `"First name"` values.  
<details>
  <summary>Example</summary>
  
  [Split example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Split%20example.xlsx)
  ```
Transactions
First Last      Date
John  Jones     9/2/2020
Adam  Jones     1/1/2000
Cathy Albertson 5/7/1990
John  Jones     5/4/2013
Cathy Albertson 12/12/1999

Splitter
pebblestream:from(Transactions)
pebblestream:split("First", "Last", SheetPerName)
First Last      Date
John  Jones     9/2/2020
Adam  Jones     1/1/2000
Cathy Albertson 5/7/1990
John  Jones     5/4/2013
Cathy Albertson 12/12/1999

SheetPerName (array of sheets)
[First Last  Date
 John  Jones 9/2/2020
 John  Jones 5/4/2013,
 
 First Last  Date
 Adam  Jones 1/1/2000,
 
 First Last      Date
 Cathy Albertson 5/7/1990
 Cathy Albertson 12/12/1999]
  ```
</details>

_O(cells)_

# Annotative directives

Annotative directives look like `pebblestream:sorted:asc` or `pebblestream:id` and go in a comment in the top row of a sheet. Depending on the directive, it may matter which column it's placed in.  

* [allowed-values](#allowed-values)
* [div/0](#div0)
* [do-not-compile](#do-not-compile)
* [fail](#fail)
* [grouped](#grouped)
* [header-count](#header-count)
* [hint](#hint)
* [id](#id)
* [identical-columns](#identical-columns)
* [ignore](#ignore)
* [match-rows](#match-rows)
* [not-blank](#not-blank)
* [out](#out)
* [sorted](#sorted)
* [stop](#stop)
* [stop-immediate](#stop-immediate)
* [unique](#unique)
* [warn](#warn)

## allowed-values
`pebblestream:allowed-values[:value]+`  
An error will be thrown if this column has a value besides those listed. If `value` is a string, it should be written between `"`'s. For blank, put `blank`.  
[Allowed-values example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Allowed-values%20example.xlsx)  
_O(rows)_

## div/0
`pebblestream:div/0:(spreadsheet|worksheet|column)[:42]`  
Whenever a #DIV/0! would happen, 0 is returned. `spreadsheet`, `worksheet`, and `column` apply only to the spreadsheet, worksheet, or column in which they are present. If `:<number>` is provided at the end, that number is returned instead of 0.  
[Div/0 example 1.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Div0%20example%201.xlsx)  
[Div/0 example 2.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Div0%20example%202.xlsx)  
_O(1)_

## do-not-compile
`pebblestream:do-not-compile`  
This spreadsheet will not exist to the compiler. Any sheet referencing it will cause a compile error.  
[Do-not-compile example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Do-not-compile%20example.xlsx)  
_O(1)_

## fail
`pebblestream:fail`  
If any value in this column is TRUE, the runtime will throw an exception.  
[Fail example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Fail%20example.xlsx)  
_O(1)_

## grouped
`pebblestream:grouped`  
If any value in this column appears multiple times but not next to each other, the runtime will throw an exception.  
[Grouped example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Grouped%20example.xlsx)  
_O(rows)_

## header-count
`pebblestream:header-count:2`  
Treats this sheet as having 2 header rows.  
[Header-count example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Header-count%20example.xlsx)  
_O(1)_

## hint
`pebblestream:hint:key[:value]`  
Stores a map of Sheet name -> `key` -> list of values for each hint directive found in the sheet. If `value` isn't specified, uses `nil`.  
[Hint example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Hint%20example.xlsx)  
_O(1)_

## id
`pebblestream:id[:name]`  
Every row has to be unique when looking at the cells in columns with `id` and the same `name` (where no `name` is also a name).  
[ID example 1.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/ID%20example%201.xlsx)  
[ID example 2.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/ID%20example%202.xlsx)  
_O(rows)_

## identical-columns
`pebblestream:identical-columns:Sheet1[: ...]`  
Every column in `Sheet1 ...` with the same header name as this one must have the same values as this one.  
[Identical-columns example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Identical-columns%20example.xlsx)  
_O(rows)_

## ignore
`pebblestream:ignore`  
This sheet won't be generated by the runtime. Can only be used on isolated worksheets.  
[Hint example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Hint%20example.xlsx)  
_O(1)_

## match-rows
`pebblestream:match-rows:Sheet1`  
This sheet's last row will be dragged down by the runtime to match the number of rows in `Sheet1`.  
[Match-rows example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Match-rows%20example.xlsx)  
_O(rows_matching)_

## not-blank
`pebblestream:not-blank`  
The runtime will throw an exception if any value in this column is blank.  
[Not-blank example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Not-blank%20example.xlsx)  
_O(rows)_

## out
`pebblestream:out`  
This sheet will be included in the output sheets when run by the runtime instead of whatever would have been output by default.  
[Out example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Out%20example.xlsx)  
_O(1)_

## sorted
`pebblestream:sorted[:(asc|desc)][:<number>]`  
The runtime will throw an exception if this column is not sorted, ascending by default. If multiple are placed in the same sheet, the columns will be considered multiple keys for the same sorted check, and they should each have a number to indicate their order for breaking ties (lower = earlier).  
[Sorted example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Sorted%20example.xlsx)  
_O(rows)_

## stop
`pebblestream:stop`  
The runtime will drag down this worksheet row by row until a cell in this column is `TRUE`.  
[Stop example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Stop%20example.xlsx)  
_O(rows_until_stop)_

## stop-immediate
`pebblestream:stop-immediate`  
Same as `pebblestream:stop`, except when this column is `TRUE`, the rest of the row to the right is not completed.  
[Stop-immediate example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Stop-immediate%20example.xlsx)  
_O(rows_until_stop)_

## unique
`pebblestream:unique`  
Every row must have a unique value.  
[Unique example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Unique%20example.xlsx)  
_O(rows)_

## warn
`pebblestream:warn`  
When this column is `TRUE`, its address will be included in a `:warnings` entry of the runtime's returned output context.  
[Warn example.xlsx](https://github.com/GeorgetownSoftwareHouse/pebble-stream-documentation/raw/main/examples/Warn%20example.xlsx)  
_O(1)_
    
