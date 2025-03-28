# HTTP Status Codes

## 1. Informational Responses (100–199)

- **100 Continue**: The initial part of a request has been received and the client can continue with the request. It is used when client needs to check if server can serve the request or not.Client sends 100 http request with content-type,filesize and if server can serve the request it sends green signal/100 back to client.
## 2. Successful Responses (200–299)

- **200 OK**: The request has succeeded. The meaning of the success depends on the HTTP method used.Generally used for sucessful GET/PUT request.
- **201 Created**: The request has succeeded and a new resource has been created as a result.Generally used for POST request.
- **202 Accepted**: The request has been accepted for processing, but the processing is not complete.
- **204 No Content**: The server successfully processed the request, but is not returning any content.Generally used for DELETE request.After sucessful delete,there is no data to send to client.


## 3. Redirection Messages (300–399)

- **300 Multiple Choices**: The request has more than one possible response, and the user or user agent should choose one.
- **301 Moved Permanently**: The requested resource has been permanently moved to a new URL.Suppose /api/suman is now changed to /api/sumandevkota.So in this case 301 or 302 is used depending upon situation.
- **302 Found**: The requested resource is temporarily located at a different URL, as specified in the Location header.


## 4. Client Error Responses (400–499)

- **400 Bad Request**: The server cannot process the request due to a client error (e.g., malformed request syntax).
- **401 Unauthorized**: Authentication is required and has failed or has not yet been provided.
- **402 Payment Required**: Reserved for future use, but it is not widely used.
- **403 Forbidden**: The server understands the request but refuses to authorize it.
- **404 Not Found**: The requested resource could not be found on the server. Eg:/api/123 returns 404 if user with id 123 is not present.
- **408 Request Timeout**: The server timed out waiting for the request.


## 5. Server Error Responses (500–599)

- **500 Internal Server Error**: A generic error message indicating that the server encountered an unexpected condition.
- **503 Service Unavailable**: The server is currently unable to handle the request due to temporary overload or maintenance of the server.
