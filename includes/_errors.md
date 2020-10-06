# Errors

The Disputes API uses the following error codes:


Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is invalid.
401 | Unauthorized -- Your Access Token is invalid.
405 | Method Not Allowed -- You have tried to submit a dispute using the wrong HTTP method.
406 | Not Acceptable -- You requested a response format that isn't JSON.
409 | Conflict -- You have submitted a duplicate dispute, tenancy, dispute_line or evidence object.
415 | Unsupported Media Type -- You have submited a request in a format that is not JSON.
500 | Internal Server Error -- We had a problem with our server. Contact the admin.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.

