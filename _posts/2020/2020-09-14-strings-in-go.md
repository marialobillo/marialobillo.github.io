---
layout: post
title:  "Strings in Go"
date:   2020-09-14 11:12:50 +0100
categories: go golang strings programming 
permalink: /strings-in-go
---



# Strings in Go

Disclaimer: This article don't pretend to teach you anything, it's just part of my notes. If it could be helpful for somebody, great :)

Strings are sequences of bytes. And thouse bytes are counted from zero to the lenght of the string minus one.

```Go
var str string = "Notes about string in Golang"
var length int = len(str)
```

## Index operations

We can get the byte in the string with a given index using this 

```Go
var c byte = str[2]
```

## Substrings

We can get a piece from the string

```Go
var c string = str[3:7]
fmt.Println(c)
```

## Substring shortcuts, taking slices

If we want to get all bytes from zero until certain index

```Go
var c byte = str[:6]
```

If we want to get all bytes from certain index until the end of string

```Go
var c byte = str[6:]
```

## After a backslash

```
\a   U+0007 alert or bell
\b   U+0008 backspace
\f   U+000C form feed
\n   U+000A line feed or newline
\r   U+000D carriage return
\t   U+0009 horizontal tab
\v   U+000b vertical tab
\\   U+005c backslash
\'   U+0027 single quote  
\"   U+0022 double quote  
```



## Most Popular String Functions

1. Compare

```Go
fmt.Println(strings.Compare("A", "B"))  // A < B => -1
fmt.Println(strings.Compare("B", "A"))  // B > A  => 1
fmt.Println(strings.Compare("Japan", "Australia")) // J > A => 1
```

2. Contains

```Go
fmt.Println(strings.Contains("Australia", "Aus"))   // Any part of string => true
fmt.Println(strings.Contains("Japan", "JAP")) // Case sensitive => false
```

3. Count

```Go
fmt.Println(strings.Count("Australia", "a")) // => 2
```

4. EqualFold

```Go
fmt.Println(strings.EqualFold("Australia", "aUSTRALIA")) // true
fmt.Println(strings.EqualFold("Australia", "Australia")) // true
fmt.Println(strings.EqualFold("Australia", "Aus")) // true
```

5. Fields
The Fields function breaks a string around each instance of one or more consecutive white space characters into an Array.

```Go
testString := "Australia is a country and continent surrounded by the Indian and Pacific oceans."
testArray := strings.Fields(testString)
for _, v := range testArray {    
    fmt.Println(v)
  } 
```
6. HasPrefix


```Go
fmt.Println(strings.HasPrefix("Australia", "Aus")) // => true
fmt.Println(strings.HasPrefix("Australia", "aus")) // => false
```
  
7. HasSuffix

```Go
fmt.Println(strings.HasSuffix("Australia", "lia")) // => true
fmt.Println(strings.HasSuffix("Australia", "A")) // => false
```

8. Index

The Index function enables searching particular text within a string.

```Go
fmt.Println(strings.Index("Australia", "Aus")) // => 0
fmt.Println(strings.Index("Australia", "aus")) // => -1
```

9. Join

The Join function return a string from the elements of an slice.

```Go
textString := []string{"Australia", "Japan", "Canada"}
fmt.Println(strings.Join(textString, "-")) // => Australia-Japan-Canada
```



## Notes

- String are immutable: once created, it is impossible to change the contents of a string.
- Data type `rune` is equivalent to `int32`.
- String literals are enclosed by double quotes.


## Resources

[https://www.golangprograms.com/golang/string-functions/](https://www.golangprograms.com/golang/string-functions/)

[https://golang.org/pkg/strings/](https://golang.org/pkg/strings/)

[https://golang.org/doc/effective_go.html](https://golang.org/doc/effective_go.html)