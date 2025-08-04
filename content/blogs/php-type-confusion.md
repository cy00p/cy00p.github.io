---
title: PHP Type Juggling Vulnerability
slug: php-type-coercion
description: php type juggling, a common and dangerous vulnerability
longDescription: a dive through type juggling, a subtle PHP or JS vulnerability arising from simple coding mistakes
cardImage: "https://cy00p.github.io/cy00p.webp"
tags: ['web','php']
readTime: 4
featured: true
rating: advanced
timestamp: 2022-12-16T01:01:01+00:00
---

Consider the following program:
```php
	$example_int = 7
	$example_str = "7"

	if ($example_int == $example_str) {
		echo("successful comparison")
	}
```
This program will run without errors and output 'successful comparison'.  Somewhat just like Javascript behaviour in int and str comparisons

Here's more:
```php
	("7 puppies" == 7) // True
	("Puppies" == 0) // True
	(0 == "Admin_Password") // True
```

## Conditions of Exploitation
- It's not always exploitable, most often needs to be combined with a *deserialization flaw*. 
- That's because POST, GET parameters and Cookie values are, most of the time, passed as strings or arrays into the program. If a POST parameter is passed as a string, no type conversion would be needed i.e.
```php
	("0" == "Admin_Password") // False
```
- However, if an app accepts input via functions like *json_decode()* or *unserialize()*, it's possible for the end-user to specify the type of input passed in
```php
	{"password": "0"}
	{"password": 0}		// Here the attacker exploits type juggling
```

## Mitigation
1. Just like in Javascript, use strict comparison operators
```php
	// loose comparison operator vs strict comparison operator
	(7 == "7") // True
	(7 === "7") // False
```
2. Specify the 'strict' option for functions that compare e.g. *in_array()* uses loose comparison by default but can be switched to type-safe comparison by specifying the strict option
3. Avoid typecasting before comparison
```php
	(7 === (int)"7_string") // True
```