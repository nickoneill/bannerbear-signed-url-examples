# Bannerbear Signed URLs

[Signed URLs](https://www.bannerbear.com/integrations/signed-urls/) are a way to use Bannerbear to generate images on demand using only URL parameters.

The standard Bannerbear product is a [REST API](https://www.bannerbear.com/product/image-generation-api/) accessed using an API key. 

Signed URLs *do not use the API key* in the URL params as by nature these params will be publicly visible. Instead, the Signed URL uses an encrypted signature so that the API key is never publicly exposed.

### Example Image

This image url is an example signed url using the Base64 encoding method described further down this README. Notice that if you try to change any of the parameters, the response becomes invalid.

```
https://cdn.bannerbear.com/signedurl/29oZzmYy7Qe7jxDORa/image.jpg?m[][name]=cGhvdG8&m[][image_url]=aHR0cHM6Ly93d3cuYmFubmVyYmVhci5jb20vaW1hZ2VzL2Jsb2cvcGhvdG8tMTQ5NTYzOTg2NzM4Ny01NDIzZDY4MTE1ODMtMS5qcGVn&m[][name]=dGl0bGU&m[][text]=V2lsbCBBSSBFdmVyIFJlcGxhY2UgRGVzaWduZXJzPw&m[][name]=cmVhZGluZw&m[][text]=OCBtaW51dGUgcmVhZA&m[][name]=YXZhdGFy&m[][image_url]=aHR0cHM6Ly93d3cuYmFubmVyYmVhci5jb20vaW1hZ2VzL2F1dGhvcl95b25nZm9vay5qcGc&m[][name]=bmFtZQ&m[][text]=Sm9uIFlvbmdmb29r&m[][name]=ZGF0ZQ&m[][text]=Tm92ZW1iZXIgMjAxOQ&base64=true&s=52f90de6cf09abbd3b58828046cd726e
```

## Table of Contents

- [How it Works](#how-it-works)
- [Create a Signed URL Base](#create-a-signed-url-base)
- [Building the Query String](#building-the-query-string)
- [Signing the URL](#signing-the-url)
- [Advanced: Using Base64](#advanced-using-base64)
- [Troubleshooting](#troubleshooting)

## How it Works

When a fresh and properly-signed URL is accessed, this chain of events happens:

- URL signature is validated
- Placeholder image is displayed
- Image is generated by Bannerbear in the background

The placeholder image looks like this:

![](https://cdn.bannerbear.com/signature_valid.png)

On subsequent requests, the same URL will display the generated image and is served from a CDN.

**You may hotlink to these CDN-backed images generated by Bannerbear - in fact, this is encouraged.**

Bear in mind that the above workflow means that signed URLs must be hit once to generate the image for the first time. After the first hit, the generated image will be returned on all subsequent hits. You should account for this in your application.

### Why No On-the-fly Generation?

With Bannerbear you can generate images using parameters that point to multiple external source images, plus use your own custom fonts etc. This makes Bannerbear images richer and more flexible than other solutions, but also increases rendering time. Rather than create brittleness through unpredictable network response times, we feel this 2-step approach is safer.

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

The returned URL at the end of this script is a signed url that will generate an image when accessed.

## Advanced: Using Base64

Depending on your use case you may find that using escaped parameters does not produce sufficiently robust URLs for the platform you intend to use them with. As an example, in our testing we found that Pinterest had trouble importing Bannerbear Signed URLs using escaped parameters. When changed to the Base64 method, the URLs imported just fine. 

### Add the &base64=true Parameter

To use Base64 encoding, add the parameter `&base64=true` to your query string *before calculating the signature*.

Then you will need to encode *all parameter values* in Base64, removing any padding or newlines that get added during the encoding process e.g. `==\n`

In Ruby this is achieved via:

```ruby
Base64.urlsafe_encode64(string, :padding => false)
```

Example standard query:

`?m[][name]=message&m[][text]=Hello+World`

Same query using Base64:

`?m[][name]=bWVzc2FnZQ&m[][text]=SGVsbG8gV29ybGQ&base64=true`

## Troubleshooting

Getting your signature to match the one Bannerbear expects can be tricky at first. Here are some common issues if you're seeing an `invalid signature` error:

- Ensure `&s=` is the last parameter in your URL
- Signature should be calculated *before* appending the `&s=` parameter
- Signature should be a MD5 hash of api key + full url
- Ensure that you are not changing the query string after calculating the signature
- If using Base64, ensure you are encoding *all* of your parameter values *including the layer name*

### Examples

:heavy_check_mark: Correct standard query

`?m[][name]=message&m[][text]=Hello+World`

:heavy_check_mark: Correct Base64 query

`?m[][name]=bWVzc2FnZQ&m[][text]=SGVsbG8gV29ybGQ&base64=true`

:x: Incorrect Base64 query - parameter is missing

`?m[][name]=bWVzc2FnZQ&m[][text]=SGVsbG8gV29ybGQ`

:x: Incorrect Base64 query - mix of encoded and unencoded parameter

`?m[][name]=message&m[][text]=SGVsbG8gV29ybGQ&base64=true`

## Pull Requests Welcome

We welcome any additional code examples showing how to build Bannerbear Signed URLs in languages not covered here, or updates to our existing examples to make them read better.
