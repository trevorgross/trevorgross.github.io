---
layout: post
title: 'Execute arbitrary JavaScript on hudexchange.info'
---
[HUD Exchange](https://www.hudexchange.info/) is an official website of the Department of Housing and Urban Development. It has mostly reference and training materials for people and organizations implementing HUD programs. 

A few months ago, I discovered a bug in the site's search results page. The bug allows execution of arbitrary JavaScript on the page. I reported this bug in detail through the contact mechanism on the site on September 22, 2022.

## Demo
This `fetch()`es my commit info from Github and parses it, then alerts my name.

<https://www.hudexchange.info/search/?dsp=&ct=&collection=&q=%5C%27%3B%20%2F*%20"%3B%20%2F%2F%20*%2F%20fetch%28%22https%3A//api.github.com/repos/trevorgross/installarch/commits%22%29.then%28response%20%3D%3E%20response.json%28%29%29.then%28commits%20%3D%3E%20alert%28commits%5B0%5D.commit.author.name%29%29%3B%20//>

## Why
A user's improperly sanitized search string is inserted into two JavaScript variables in the search results page. The failure to properly sanitize or escape the strings makes it trivial to execute arbitrary JavaScript at either or both variable locations.

## How
Go to <https://www.hudexchange.info/search>. Start a search query with `\'; /* "; // */` and end it with `//`. Put whatever JavaScript you want between those strings. This method executes code at the first variable location.

To get around the text input length limit, URI-encode your JavaScript and the comment bookends and append it directly into the search URL. To do that, wrap your JavaScript in the comment strings, add another leading backslash to the first comment string (so it doesn't get stripped when encoding), and encode it all with `encodeURI()` (e.g. ``encodeURI(`\\'; /* "; // */ [YOUR_JS_HERE] //`);``. Tack the encoded string preceded by `?q=` on to the search page URL.

## Details
User input is stored in JavaScript variables in two places in the results page source.

### Line 433 (or so)
This is the location used in the examples above. The user's search query is inserted as a string variable between single quotes. The string isn't properly sanitized first. The backslash in the leading `\';` is escaped, effectively enclosing one backslash between single quotes, with the `';` closing the string: `'\\';` JavaScript inserted after this will execute. The final `//` comments out the `';` that usually closes the single-quote-enclosed search string. The `/* */`-style comment is just a comment on this line. Its contents are included to avoid an error in the second location where user input is stored in a variable.

### Line 612 (or so)
The additional `/* "; // */` comment prevents an error where the search string is inserted into a double-quoted variable on or about line 612. The `";` closes the double-quoted string, and the `//` comments out the rest of this variable string. JavaScript inserted between the `";` and the `//` within this comment block will execute after any JavaScript that executes on line 433. Here's an example that executes on both lines: [\'; /* "; alert("Line ~612"); // \*/ alert("Line ~433"); //](https://www.hudexchange.info/search/?q=%5C';%20/*%20%22;%20alert(%22Line%20~612%22);%20//%20*/%20alert(%22Line%20~433%22);%20//)
