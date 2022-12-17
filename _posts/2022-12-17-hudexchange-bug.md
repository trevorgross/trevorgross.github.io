---
layout: post
title: 'Execute arbitrary JavaScript on hudexchange.info'
---
A few months ago, I discovered a bug in the search results page of hudexchange.info. The bug allows execution of arbitrary JavaScript on the page.

HUD Exchange is an official website of the Department of Housing and Urban Development. It has mostly reference and training materials for people and organizations implementing HUD programs. 

I reported this bug in detail through the contact mechanism on the HUD Exchange site on September 22, 2022.

## Demo
This `fetch()`es my commit info from Github and parses it, then alerts my name.
<https://www.hudexchange.info/search/?dsp=&ct=&collection=&q=%5C%27%3B%20%2F*%20"%3B%20%2F%2F%20*%2F%20fetch%28%22https%3A//api.github.com/repos/trevorgross/installarch/commits%22%29.then%28response%20%3D%3E%20response.json%28%29%29.then%28commits%20%3D%3E%20alert%28commits%5B0%5D.commit.author.name%29%29%3B%20//>

## Why
A user's improperly sanitized search string is inserted into two JavaScript variables in the search results page. The failure to properly sanitize or escape the strings makes it trivial to execute arbitrary JavaScript.

## How
Go to <https://www.hudexchange.info/search>. Start a search query with `\'; /* "; // */` and end it with `//`. Put whatever JavaScript you want between those strings. To get around the text input length limit, add another leading backslash to the first comment string (so it doesn't get stripped when encoding), wrap your JavaScript in the comment strings, encode it all with encodeURI(), and tack the encoded string on to the search URL after ?q=. E.g. ``encodeURI(`\\'; /* "; // */ [YOUR_JS_HERE] //`);``

##Details
On or about line 433 in the results page source, the search string is inserted between single quotes. The string is not properly sanitized first. On this line, the backslash in the leading `\';` is escaped, effectively enclosing one backslash between single quotes, with the `';` closing the string: `'\\';` Your JavaScript comes after this. The final `//` comments out the `';` that usually closes the single-quote-enclosed search string. The `/* */`-style comment is just a comment on this line. Its contents are included to avoid an error later in the page.

The additional `/* "; // */` comment prevents an error when the search string is inserted into a double-quoted variable on or about line 612. The `";` closes the double-quoted string, and the `//` comments out the rest of this variable string. If you're so inclined, you can add more arbitrary JavaScript between the `";` and the `//` within this comment block, and that JavaScript will execute on this line after the JavaScript that executes on line 433. For an example, paste this into the search box and hit enter: `\'; /* "; alert("Line ~612"); // */ alert("Line ~433"); //`

