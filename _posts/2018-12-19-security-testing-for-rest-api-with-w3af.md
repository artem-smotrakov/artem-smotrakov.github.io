---
layout: post
title: Security testing for REST API with w3af
date: 2018-12-19 17:31:40.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- API
- REST
- Security
- w3af
- Web security
permalink: "/en/security/security-testing-for-rest-api-with-w3af.html"
---
Nowadays more and more companies provide web APIs to access their services. They usually follow [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) style. Such a RESTful web service looks like a regular web application. It accepts an HTTP request, does some magic, and then replies with an HTTP response. One of the main differences is that the reply doesn't normally contain HTML to be rendered in a web browser. Instead, the reply usually contains data in a format (for example, JSON or XML) which is easier to process by another application.

Unfortunately, since a RESTful web service is still a web application, it may contain typical security vulnerabilities for web applications such as SQL injections, XXE, etc. One of the ways to identify security issues in web applications is to use web security scanners. Fortunately, since a RESTful web service is still a web application, we can use web security scanners to look for security issues in web APIs.

There are several well-known web security scanners. One of them is [w3af](https://github.com/andresriancho/w3af)&nbsp;created by&nbsp;[Andres Riancho](https://github.com/andresriancho). I'll focus on this scanner in the post.



## Discovering API endpoints to scan

A web scanner usually tries to browse a web site to find all available pages and parameters which then can be tested for vulnerabilities. In case of a typical web site, a scanner often starts from a home page, and looks for links to other pages in the HTML code. But this approach doesn't work with web APIs because usually API endpoints don't serve HTML data, and APIs usually don't refer to each other in their replies.

How do we get a list of API endpoints and parameters to scan? There are two main ways:

1. A web service may have an [OpenAPI specification](https://github.com/OAI/OpenAPI-Specification) which describes all endpoints, parameters, responses, authentication schemes, etc. Such a specification is normally provided by developers.
2. Using a proxy, we can capture HTTP requests which were sent to&nbsp;the API endpoints by a client. Then, we can parse the captured requests and extract information about parameters.

The way #1 may look better than #2. In a perfect world, each web service has an OpenAPI specification which is always available and up-to-date. But in the real world, it doesn't seem to happen too often. A developer may change the APIs but forget to update the spec, or for some reason they don't make the spec publicly available. In most cases, publicly available REST API have human-readable docs which is nice but it's usually hard to use in an automated way. So, in the end of the day, the option #2 may not sound too bad.

w3af scanner supports both options of identifying API endpoints to scan. For the option #1, the tool provides `crawl.open_api`&nbsp;[plugin](https://github.com/andresriancho/w3af/issues/15087) which was added in the beginning of 2018 by Andres. [The option #2 is described in the docs](https://github.com/andresriancho/w3af/blob/master/doc/sphinx/scan-rest-apis.rst#feeding-http-requests-into-w3af).

Let's talk about `crawl.open_api` plugin. The first version of the plugin tried to find an OpenAPI specification on one of the well-known locations such as:

- /api/openapi.json
- /api/v2/swagger.json
- /api/v1/swagger.yaml
- and so on

It's a very nice feature. If you have multiple web services to test, and their API specs are available in well-known locations, then you just need to feed the scanner with the host names, and that's all. The scanner is going to find all API endpoints by itself, and then test them. Unfortunately, sometimes web services don't provide API specifications, and I added `custom_spec_file` [parameter which allows to set a local path to OpenAPI specification](https://github.com/andresriancho/w3af/pull/17194). A user can use API docs and build the spec by himself, or sometimes specifications are publicly available but not at well-known location. By the way, check out [APIs.guru](https://github.com/APIs-guru/openapi-directory) project which is building a directory of REST API definitions in OpenAPI format.

## Scanning APIs which require authentication

It happens quite often that web APIs require authentication. Usually they use API keys, JWT or OAuth2 for authentication and access control. In this case, you'll need to provide authentication information for w3af, otherwise all request are going to result to 401 error, so that nothing will be actually tested. Fortunately, the `crawl.open_api` plugin has a couple of parameters which you can use to provide credentials to access APIs:

- `query_string_auth` option allows to set an API key in a query string
- `header_auth` allows to set a header with an API key or token

## Disabling validation of OpenAPI specification

The `crawl.open_api` plugin validates an API spec when it loads it. While testing several web APIs, I noticed that sometimes API specs may not be fully valid. It happens because a developer might make a mistake, or didn't strictly follow the OpenAPI standard. The best way is to figure out what is wrong in the spec, and fix it, but sometimes just disabling validation may be an acceptable short-term solution. The validation may be disabled by turning on `no_spec_validation` option of the `crawl.open_api` plugin. Note that disabling the validation may result to skipping testing of some API endpoints, so it's better to fix the spec.

## Example of a config for API scan with w3af

Here is a simple configuration for w3af to test a RESTful application. This configuration tells w3af to look just for SQL injections and 500 errors but the scanner can do more - check out the documentation or type `help`&nbsp;in w3af's console.

Store the configuration to `config`&nbsp;file, and then just run a scan with `./w3af_console -s config`&nbsp;command.

```
# configure plugins
plugins

# print verbose logs to output-w3af.txt file
# store all request and responses to output-http.txt
# and also save an html report to report.html
output html_file text_file
output
output config text_file
set output_file output-w3af.txt
set http_output_file output-http.txt
set verbose True
back
output config html_file
set output_file report.html
set verbose True
back

# enable parsing openapi specification
# which should be loaded from openapi.yaml file
# disable openapi validation
# and use authorization header to access the endpoints
crawl
crawl open_api
crawl config open_api
set header_auth "Authorization: Bearer xxx"
set no_spec_validation True
set custom_spec_location openapi.yaml
save
back

# just look for sql injections
audit
audit sqli

# but also report 500 errors
grep
grep error_500
back

# set an url where the application runs
target
set target https://foobar.com
back

# headers file contains a list of headers
# which should be sent in all requests
# in our case, headers file should contain the same authorization header 
# as we specified for the open_api plugin
http-settings
set headers_file headers
save
back

# go!
start
exit
```

Now let’s discuss potential further improvements.

## Providing context-specific parameters to scan

When the `crawl.open_api` plugin finds a new API endpoint, it tries to find all parameters which the endpoint accepts. The parameters may be in URL path, query string, headers or request body.

In order to send a test request to the endpoint, w3af has to provide values for all parameters. The endpoint may have a check for one&nbsp; parameter but miss a check for another one which may result to a vulnerability. To detect the missing check for the second parameter, w3af needs to provide a good value for the first parameter and a malicious value for the second one. Otherwise, the application will just complain that the first parameter is not correct. That's why it's quite important to use the values which are correct in the context of the application.

The `crawl.open_api` plugin tries to find good parameter values which can be used for testing later on. First, it tries to find default values which may be defined in the API specification. If the API spec provides no default value for a parameter, the plugin tries its best to guess the value based on the parameter type.

But nevertheless, it may be extremely difficult for the plugin to guess an acceptable value since it's usually highly context-specific. For example, if the API specification says a endpoint has&nbsp;`user_id`&nbsp;parameter which is an integer, the plugin will use an integer like 42 but there may not be a user with this ID. As a result, the endpoint may reject all requests complaining that a wrong user ID was provided.

It might be useful if we could set correct context-specific values for parameters which should be used during testing. Such a feature might help to test API endpoints a bit deeper. [Hope we can add such a feature soon.](https://github.com/andresriancho/w3af/pull/17339)

## Analysis of scan results

As you might have noticed, w3af contains multiple plugins which are splitted to several main categories:

- `crawl` plugins browse the application and try to discover entry points to test
- `audit` plugins test the discovered entry points for vulnerabilities
- `grep`&nbsp;plugins analyze test results, and look for suspicious cases
- There are a couple of more categories but they are out of scope here

Most of existing plugins use static checks like searching for pre-defined patterns. For example, in order to detect an SQL injection, they can look for typical error messages from database servers. This is a good and well-known approach but unfortunately it allows to catch only issues which the plugins are aware of. However, a scan may cause an application to behave in an unusual and unexpected way. For example, there may be a logical bug which has security implications, or the application may disclose application-specific sensitive information. If such a behavior doesn't match to the patterns which used by the plugins, then most probably no one is going to notice the problem. The chances to miss such a problem increase when we test more API endpoints and use more plugins which, as a result, produce huge amount of logs.

To make our life easier, [it might be good to add a new plugin which analyses HTTP request and responses](https://github.com/andresriancho/w3af/issues/17345) produced during a scan, and put similar ones to the same bucket (cluster). In particular, such a plugin may apply some machine learning algorithms for clustering data, for example, K-means or DBSCAN. Then, it may be a bit easier to review a couple of cases from each cluster to make sure that the application behaves correctly.

It's important to mention that such a plugin wouldn't guarantee that all issues are found during testing. In fact, it should be even fine if the plugin doesn't report issues at all. This plugin should be considered as a tool which may help with analysis of huge amount of scan results.

## Conclusion

w3af is a great tool although I am still not sure how to pronounce it :) The scanner can be useful in manual security assessments, or it can be integrated in a CI/CD process to scan web services regularly. I would definitely recommend to give it a try. You can also consider contributing to the [project](https://github.com/andresriancho/w3af)&nbsp;- I think&nbsp;it's worth it. The code is well-structured and contains lots of comments which make it easier to start hacking the code. Good luck!

