---
layout: doc
title: Samples
description: Various Hurl samples to show how to run and tests HTTP requests and responses.
section: Getting Started
---
# {{ page.title }}

To run a sample, you can edit a file with the sample content, and use Hurl:

```shell
$ vi sample.hurl

GET https://example.net

$ hurl sample.hurl
```


## Getting Data

A simple GET:

```hurl
GET https://example.net
```

[Doc]({% link _docs/request.md %}#method)

### HTTP Headers

A simple GET with headers:

```hurl
GET https://example.net/news
User-Agent: Mozilla/5.0 
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

[Doc]({% link _docs/request.md %}#headers)

Headers can be used to perform [Basic authentication]. Given a login `bob` 
with password `secret`:

In a shell:

```shell
$ echo -n 'bob:secret' | base64
Ym9iOnNlY3JldA==
```

Then, use [`Authorization` header] to add basic authentication to a request:

```hurl
GET https://example.com/protected
Authorization: Basic Ym9iOnNlY3JldA==
```

Alternatively, one can use [`--user` option].

### Query Params

```hurl
GET https://example.net/news
[QueryStringParams]
order: newest
search: something to search
count: 100
```

Or:

```hurl
GET https://example.net/news?order=newest&search=something%20to%20search&count=100
```

[Doc]({% link _docs/request.md %}#query-parameters)

## Sending Data

### Sending HTML Form Datas

{% raw %}
```hurl
POST https://example.net/contact
[FormParams]
default: false
token: {{token}}
email: john.doe@rookie.org
number: 33611223344
```
{% endraw %}

[Doc]({% link _docs/request.md %}#form-parameters)

### Sending Multipart Form Datas

{% raw %}
```hurl
POST https://example.net/upload
[MultipartFormData]
field1: value1
field2: file,example.txt;
# On can specify the file content type:
field3: file,example.zip; application/zip
```
{% endraw %}

[Doc]({% link _docs/request.md %}#multipart-form-data)

### Posting a JSON Body

With an inline JSON:

```hurl
POST https://api.example.net/tests
{
    "id": "456",
    "evaluate": true
}
```

[Doc]({% link _docs/request.md %}#json-body)

With a local file:

```hurl
POST https://api.example.net/tests
Content-Type: application/json
file,data.json;
```

[Doc]({% link _docs/request.md %}#file-body)

### Templating a JSON / XML Body

Using templates with [JSON body] or [XML body] is not currently supported in Hurl. 
Besides, you can use templates in [raw string body] with variables to send a JSON or XML body:
 
{% raw %}
~~~hurl
PUT https://api.example.net/hits
Content-Type: application/json
```
{
    "key0": "{{a_string}}",
    "key1": {{a_bool}},
    "key2": {{a_null}},
    "key3": {{a_number}}
}
```
~~~
{% endraw %}

Variables can be initialized via command line:

```shell
$ hurl --variable key0=apple \
       --variable key1=true \
       --variable key2=null \
       --variable key3=42 \
       test.hurl
```

Resulting in a PUT request with the following JSON body:

```
{
    "key0": "apple",
    "key1": true,
    "key2": null,
    "key3": 42
}
```

[Doc]({% link _docs/request.md %}#raw-string-body)

## Testing Response

### Testing Response Headers

Use implicit response asserts to test header values:

```hurl
GET http://www.example.org/index.html

HTTP/1.0 200
Set-Cookie: theme=light
Set-Cookie: sessionToken=abc123; Expires=Wed, 09 Jun 2021 10:18:14 GMT
```

[Doc]({% link _docs/asserting-response.md %}#headers)


Or use explicit response asserts with [predicates]:

```hurl
GET https://example.net

HTTP/1.1 302
[Asserts]
header "Location" contains "www.example.net"
```

[Doc]({% link _docs/asserting-response.md %}#header-assert)


### Testing REST Apis

Asserting JSON body response (node values, collection count etc...) with [JSONPath]:

```hurl
GET https://example.org/order
screencapability: low

HTTP/1.1 200
[Asserts]
jsonpath "$.validated" == true
jsonpath "$.userInfo.firstName" == "Franck"
jsonpath "$.userInfo.lastName" == "Herbert"
jsonpath "$.hasDevice" == false
jsonpath "$.links" count == 12
jsonpath "$.state" != null
```

[Doc]({% link _docs/asserting-response.md %}#jsonpath-assert)

Testing status code:

```hurl
GET https://example.org/order/435

HTTP/1.1 200
```

[Doc]({% link _docs/asserting-response.md %}#version-status)

```hurl
GET https://example.org/order/435

# Testing status code is in a 200-300 range
HTTP/1.1 *
[Asserts]
status >= 200
status < 300
```

[Doc]({% link _docs/asserting-response.md %}#status-assert)


### Testing HTML Response

```hurl
GET https://example.com

HTTP/1.1 200
Content-Type: text/html; charset=UTF-8

[Asserts]
xpath "string(/html/head/title)" contains "Example" # Check title
xpath "count(//p)" == 2  # Check the number of p
xpath "//p" count == 2  # Similar assert for p
xpath "boolean(count(//h2))" == false  # Check there is no h2  
xpath "//h2" not exists  # Similar assert for h2
```

[Doc]({% link _docs/asserting-response.md %}#xpath-assert)

### Testing Set-Cookie Attributes

```hurl
GET http://myserver.com/home

HTTP/1.0 200
[Asserts]
cookie "JSESSIONID" == "8400BAFE2F66443613DC38AE3D9D6239"
cookie "JSESSIONID[Value]" == "8400BAFE2F66443613DC38AE3D9D6239"
cookie "JSESSIONID[Expires]" contains "Wed, 13 Jan 2021"
cookie "JSESSIONID[Secure]" exists
cookie "JSESSIONID[HttpOnly]" exists
cookie "JSESSIONID[SameSite]" == "Lax"
```

[Doc]({% link _docs/asserting-response.md %}#cookie-assert)

## Others

### Testing Endpoint Performance

```hurl
GET https://sample.org/helloworld

HTTP/* *
[Asserts]
duration < 1000   # Check that response time is less than one second
```

[Doc]({% link _docs/asserting-response.md %}#duration-assert)

### Using SOAP Apis

```hurl
POST https://example.net/InStock
Content-Type: application/soap+xml; charset=utf-8
SOAPAction: "http://www.w3.org/2003/05/soap-envelope"
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope" xmlns:m="http://www.example.org">
  <soap:Header></soap:Header>
  <soap:Body>
    <m:GetStockPrice>
      <m:StockName>GOOG</m:StockName>
    </m:GetStockPrice>
  </soap:Body>
</soap:Envelope>

HTTP/1.1 200
```

[Doc]({% link _docs/request.md %}#xml-body)

### Capturing and Using a CSRF Token

{% raw %}
```hurl
GET https://example.net

HTTP/* 200
[Captures]
csrf_token: xpath "string(//meta[@name='_csrf_token']/@content)"

POST https://example.net/login?user=toto&password=1234
X-CSRF-TOKEN: {{csrf_token}}

HTTP/* 302
```
{% endraw %}

[Doc]({% link _docs/capturing-response.md %}#xpath-capture)

### Checking Byte Order Mark (BOM) in Response Body

```hurl
GET https://example.net/data.bin

HTTP/* 200
[Asserts]
bytes startsWith hex,efbbbf;
```

[Doc]({% link _docs/asserting-response.md %}#bytes-assert)


[JSON body]: {% link _docs/request.md %}#json-body
[XML body]: {% link _docs/request.md %}#xml-body
[raw string body]: {% link _docs/request.md %}#raw-string-body
[predicates]: {% link _docs/asserting-response.md %}#predicates
[JSONPath]: https://goessner.net/articles/JsonPath/
[Basic authentication]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#basic_authentication_scheme
[`Authorization` header]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization
[`--user` option]: {% link _docs/man-page.md %}#user
