---
title: "Json Processing in Go Marshal or Encode"
date: 2022-04-03T22:08:55+09:00
draft: true
---

intro

### Marshal() and Unmarshal()

blah

### NewEncoder().Encode() and NewDecoder().Decode()

blah

### Which One Should I Use?

blah

All JSON files are sourced from https://jsonplaceholder.typicode.com.

large-file.json is sources from [test-data/large-file.json at master · json-iterator/test-data · GitHub](https://github.com/json-iterator/test-data/blob/master/large-file.json)



- JSON processing: marshal/unmarshal vs. encoder/decoder
  - json.Marshal() and json.Unmarshal()
    - accepts v and returns a byte slice of the encoded JSON
    - 
  - key difference: marshal -> string, encode -> stream
    - the code isn't very different. both use e.marshal. the only difference for the end user will be the parameters.
    - it's not worth talking about it too much...
    - if there is a writer or a reader you can work with, aka in API controller functions, it's more convenient to use decode.
    - if there is a byte slice to work with, just use marshal.
