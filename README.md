# Bannerbear Signed URLs

[Signed URLs](https://www.bannerbear.com/integrations/signed-urls/) are a way to use Bannerbear to generate images on-the-fly using only URL parameters.

The standard Bannerbear product is a [REST API](https://www.bannerbear.com/product/image-generation-api/) accessed using an API key. 

Signed URLs *do not use the API key* in the URL params as by nature these params will be publicly visible. Instead, the Signed URL uses an encrypted signature so that the API key is never publicly exposed.

### Example Image

This image url is an example signed url using the Base64 encoding method described further down this README. Notice that if you try to change any of the parameters, the response becomes invalid.

```
https://cdn.bannerbear.com/signedurl/29oZzmYy7Qe7jxDORa/image.jpg?m[][name]=cGhvdG8&m[][image_url]=aHR0cHM6Ly93d3cuYmFubmVyYmVhci5jb20vaW1hZ2VzL2Jsb2cvcGhvdG8tMTQ5NTYzOTg2NzM4Ny01NDIzZDY4MTE1ODMtMS5qcGVn&m[][name]=dGl0bGU&m[][text]=V2lsbCBBSSBFdmVyIFJlcGxhY2UgRGVzaWduZXJzPw&m[][name]=cmVhZGluZw&m[][text]=OCBtaW51dGUgcmVhZA&m[][name]=YXZhdGFy&m[][image_url]=aHR0cHM6Ly93d3cuYmFubmVyYmVhci5jb20vaW1hZ2VzL2F1dGhvcl95b25nZm9vay5qcGc&m[][name]=bmFtZQ&m[][text]=Sm9uIFlvbmdmb29r&m[][name]=ZGF0ZQ&m[][text]=Tm92ZW1iZXIgMjAxOQ&base64=true&s=52f90de6cf09abbd3b58828046cd726e
```

## Table of Contents

- [Create a Signed URL Base](#create-a-signed-url-base)
- [Building the Query String](#building-the-query-string)
- [Signing the URL](#signing-the-url)


## Create a Signed URL Base

Signed URLs are attached at the template level in Bannerbear. To start using Signed URLs, create a Signed URL Base. This is a unique endpoint associated with a template. You can create as many Bases as you want for each template. For most cases you will only need to create one Base for a template, but for flexibility Bannerbear allows you to create multiple Bases per template.

## Building the Query String

Bannerbear expects all modifications to appear in the url parameter `m`

`m` is the url-parameterized version of the JSON modifications that you will find in the template API console.

For example, if your JSON modifications looks like this:
```json
"modifications": [
  {
    "name": "message",
    "text": "Hello World"
  },
  {
    "name": "face",
    "image_url": "https://cdn.bannerbear.com/sample_images/welcome_bear_photo.jpg"
  }
]
```
The resulting query would look like this:

`?m[][name]=message&m[][text]=Hello+World&m[][name]=face&m[][image_url]=https%3A%2F%2Fcdn.bannerbear.com%2Fsample_images%2Fwelcome_bear_photo.jpg`

Note the escaped, url-safe parameters.

### Query String Tips

- Define all modifications in the paramater array `m[]`
- Each modification should specify a layer name e.g. `m[][name]=`
- Followed by the text you want to change e.g. `m[][text]=`
- Or if the layer is an image, the url to an image e.g. `m[][image_url]=`
- Remember to escape your parameter values

## Signing the URL

Bannerbear expects the signature to appear in the parameter `s`

`&s=` should be the *last parameter* in your URLs

The signature is calculated as an MD5 hash of your api key + url base + query

### Signing Example

This example is in Ruby but you can find other language examples in the repository.

```ruby
#api_key: your project API key - keep this safe and non-public
api_key = "YOURKEY"

#base: this signed url base
base = "https://cdn.bannerbear.com/signedurl/YOURID/image.jpg"

#query: the query string of modifications you want to generate
query = "?m[][name]=message&m[][text]=Hello+World"

#calculate the signature
signature = Digest::MD5.hexdigest(api_key + base + query)

#append the signature
return base + query + "&s=" + signature
```

## Pull Requests Welcome

We welcome any additional code examples showing how to build Bannerbear Signed URLs in languages not covered here, or updates to our existing examples to make them read better.
