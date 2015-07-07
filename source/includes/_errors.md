# Errors

The ProxyPay API uses the following error codes:

Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is malformed
401 | Unauthorized -- Your API key is wrong
403 | Forbidden -- You are not authorized to access this resource
404 | Not Found -- The specified resource could not be found
405 | Method Not Allowed -- You tried to access a resource with an invalid HTTP method
406 | Not Acceptable -- You requested a format that isn't json
422 | Unprocessable Entity -- Your request includes invalid fields. Check the response body for details.
429 | Too Many Requests -- You're exceding the API rate limit! Reduce the number of requests / minute.
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.