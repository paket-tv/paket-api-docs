# Errors

The Paket API uses the following error codes:


Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is invalid.
401 | Unauthorized -- Your API key is wrong.
403 | Forbidden -- The item requested is hidden for administrators only.
404 | Not Found -- The specified item could not be found.
405 | Method Not Allowed -- You tried to access a item with an invalid method.
406 | Not Acceptable -- You requested a format that isn't json.
410 | Item removed
429 | Too Many Requests
500 | Internal Server Error
503 | Service Unavailable
