---
layout: post
title: "Course notes: Udemy google sheets"
excerpt: "Useful course. But makes you jealous of the author for creating a popular course that just mostly references official documentation"
---

Got this course on a sale from Udemy: "Master Google Sheets (and see why it's better than Excel)"

It was useful, but only at a basic level I guess. The author could've spent time exploring more advanced topics.

So, the notes...

can protect a single sheet of a spreadsheet file alone. click the sheet menubar arrow button present next to the sheet name.

conditional formatting.

filtering. funnel icon, save as filter, filter view dropdown.

File -> See revision history

cell reference wrt formulas: relative(A1) vs absolute($A$1)

formula: 
SWITCH: `=SWITCH(B2:B11, "abc", "YUP", "def", "NOP", "UGH")`
IFS: `=ifs(B9="abc", "FOO", B9="def", "BAR")`
IFERROR: 
* return null if error: `=iferror(ifs(B3="abc", "FOO", B3="def", "BAR"))`
* return custom val if error: `=iferror(ifs(B3="abc", "FOO", B3="def", "BAR"), "SORRY")`


vlookup/hlookup:
`=VLOOKUP(B3, PartCodes!$A$2:$B$4, 2)`
It does table joining using a specific column, just like in sql.


ImportXml, ImportHTML:
Pull data from any structured data from internet.

`=IMPORTXML("https://en.wikipedia.org/wiki/Kamal_Haasan", "//a/@href")`


Eg: pull first page search result links for any search term into a sheet using importxml:
`=IMPORTXML($C$1, "//h3[@class='r']/a/@href")`
Where the C1 url is: https://encrypted.google.com/search?q=prasanna+natarajan+ruby
