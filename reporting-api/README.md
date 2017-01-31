Reporting API Documentation
===========================

Intro
-----

The Talkdesk Reporting API allows you to export raw data from your Talkdesk account in more complex ways than the integrations we currently provide.

All methods must be called using HTTPS, using a mix of `GET`, `POST` and `DELETE` requests.

The reponse format is `application/json`. Additional hypermedia metadata may be provided in the responses in a format compatible with `JSON+HAL`, which can be used to extract links to related resources (including pagination) without the need to build URLs.

The API described in this guide is specified using Swagger's Open API format and reference documentation for it can be found at: <http://api-docs.talkdesk.org/redoc>

Authentication
--------------

### Requesting client credentials

Please contact platform-team@talkdesk.com request a new set of client credentials so that your application can access the Talkdesk APIs.

### Obtaining an access token

Armed with your client credentials, your application will now be able to access Talkdesk API resources on a user's behalf by requesting an access token.

There are several ways to make this request, and they vary based on the type of application you are building. Currently, the OAuth 2.0 Resource Owner Password Credentials grant is supported.

Issue the following `POST` request to `/auth/token`:

Headers:
``` http
Content-Type: application/json
```

Request body:
``` http
{
  "client_id": "INSERT_CLIENT_ID_HERE",
  "client_secret": "INSERT_CLIENT_SECRET_HERE",
  "client_version": "INSERT_CLIENT_VERSION_HERE",
  "account": "INSERT_ACCOUNT_NAME_HERE",
  "username": "INSERT_USERNAME_HERE",
  "password": "INSERT_PASSWORD_HERE"
}
```

A successful response containing an access token is returned. All timestamps will be returned in UTC.

``` json
{
  "user_id": "55a3cc81041268c5b4000001",
  "id_token_expires": "2016-08-12T14:58:28Z",
  "id_token": "23edd8605209e7061f3e106454ddfe62e40b756d67b5a78e74745695416cfd302811085dba45b8323ee88a1a0b3c71da",
  "account_id": "559d52fcd22fe6fd24000001",
  "access_token_expires": "2016-08-13T13:53:28Z",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c3IiOiJ0aWFnby5zb3VzYStzZmRldkB0YWxrZGVzay5jb20iLCJzdWIiOiI1NWEzY2M4MTA0MTI2OGM1YjQwMDAwMDEiLCJuYmYiOjE0NzEwMTAwMDgsImp0aSI6IjU3YWRkNGQ4ZjBjNGNmMDAwNDI4ZTM4NiIsImlzcyI6IkVsaXhpci5BdXRoU3lzdGVtIiwiaWF0IjoxNDcxMDEwMDA4LCJleHAiOjE0NzEwOTY0MDgsImF1ZCI6bnVsbCwiYWlkIjoiNTU5ZDUyZmNkMjJmZTZmZDI0MDAwMDAxIiwiYWNjIjoic3VwNCJ9.KIsF-Rw3BjD7y4zGtXqKmZdhSPOAQXHNVUZeZl-mbHA",
  "_links": {
    "self": {
      "href": "https://api.talkdeskdev.com/auth/token"
    }
  }
}
```

### Refreshing an access token

If the application use case needs to perform background requests to Talkdesk resources without asking for the user credentials, it is possible to generate a new `access_token` by using the `id_token` from the previous response.

Please note that the recommended approach to refresh tokens is to reactively handle failures when using expired access tokens identified with `403 Forbidden` status code. If you really need to pro-actively generate new access tokens before older ones expire, you can check the `access_token_expires` timestamp from the previous response.

Issue the following `POST` request to `/auth/access`:


Headers:
```http
Content-Type: application/json
```

Request body
``` http
{
  "client_id": "INSERT_CLIENT_ID_HERE",
  "client_secret": "INSERT_CLIENT_SECRET_HERE",
  "client_version": "INSERT_CLIENT_VERSION_HERE",
  "id_token": "INSERT_ID_TOKEN_HERE"
}
```

A successful response containing a new `access_token` is returned. The `id_token` may or may not be refreshed as well based on the client application permissions.

``` json
{
  "user_id": "55a3cc81041268c5b4000001",
  "id_token_expires": "2016-08-12T15:33:59Z",
  "id_token": "9952e826e562436802eee37910647a2222b8ef2b77fe7e3e1140d621790c22102ede0cbc48b6f63a9db47ea213b264ca",
  "account_id": "559d52fcd22fe6fd24000001",
  "access_token_expires": "2016-08-13T14:28:59Z",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c3IiOiJ0aWFnby5zb3VzYStzZmRldkB0YWxrZGVzay5jb20iLCJzdWIiOiI1NWEzY2M4MTA0MTI2OGM1YjQwMDAwMDEiLCJuYmYiOjE0NzEwMTIxMzksImp0aSI6IjU3YWRkZDJiZjBjNGNmMDAwNDhmNzAxYyIsImlzcyI6IkVsaXhpci5BdXRoU3lzdGVtIiwiaWF0IjoxNDcxMDEyMTM5LCJleHAiOjE0NzEwOTg1MzksImF1ZCI6bnVsbCwiYWlkIjoiNTU5ZDUyZmNkMjJmZTZmZDI0MDAwMDAxIiwiYWNjIjoic3VwNCJ9.ZGljyGQNAmGBiYTmswQAkFHiIjUjk68m_yvJ0ig0XGU",
  "_links": {
    "self": {
      "href": "https://api.talkdeskdev.com/auth/access"
    }
  }
}
```

### Calling Talkdesk APIs with an access token

After obtaining an access token, your application can use it to make calls to Talkdesk API resources on behalf of a given user by including it as an authorization bearer header.

``` http
Authorization: Bearer "INSERT_ACCESS_TOKEN_HERE"
```

Asynchronous API
----------------

The Talkdesk Reporting API allows developers to trigger reporting jobs and then download the generated bulk report files.

The following steps allow the report job creation and further retrival of results:

1.  `POST /reports/calls/jobs` to trigger a new report generation. A job status resource will be returned.
2.  `GET /reports/calls/jobs/{id}` to check on the report status. When the report is created this request will be automatically redirected to download the file.
3.  `GET /reports/calls/files/{id}` to download the report once it has been generated.

### Creating a report job

The API currently supports generation of calls reports.

A format can be specified (`json` and `csv` are supported) as well as a timespan specifying the date interval that this report will include.

**Important:** Report generation is subject to a data availability policy where only data is made available with a 1 hour delay. Therefore, when specifying the `timespan` field the `to` date should be at least one our in the past (if a request is issued at 12pm, the param should be at most 11am).

Issue an authenticated `POST` request to `/reports/calls/jobs`:

Headers:
``` http
Authorization: Bearer "INSERT_ACCESS_TOKEN_HERE"
Content-Type: application/json
```

Request body:
```http
{
  "format": "${format}",
  "timespan": {
    "from": "${from}",
    "to": "${to}"
  }
}
```

A successful `202 Accepted` response containing the job status will be returned. Links are provided to check on this job status and with the final location of the report (once it gets generated):

``` json
{
  "id": "57add63204c799583700028e",
  "status": "created",
  "timespan": {
    "from": "2016-07-29 00:00:00 UTC",
    "to": "2016-07-29 23:59:59 UTC"
  },
  "_links": {
    "self": {
      "href": "https://api.talkdeskdev.com/reports/calls/jobs/57add63204c799583700028e"
    },
    "files": {
      "href": "https://api.talkdeskdev.com/reports/calls/files/57add63204c799583700028e"
    }
  }
}
```

### Checking a report job status

While a report is being generated, the respective job status can be checked using the endpoint specified in the previous response:

Until a report job is in progress, it is possible to cancel it using a `DELETE` request to this endpoint (see below).

Issue an authenticated `GET` request to `/reports/calls/jobs/{id}`

Headers:
``` http
Authorization: Bearer "INSERT_ACCESS_TOKEN_HERE"
```

A successful `200 Ok` response containing the current report job status will be returned.

``` json
{
  "id": "57add63204c799583700028e",
  "status": "done",
  "timespan": {
    "from": "2016-07-29 00:00:00 UTC",
    "to": "2016-07-29 23:59:59 UTC"
  },
  "_links": {
    "self": {
      "href": "https://api.talkdeskdev.com/reports/calls/jobs/57add63204c799583700028e"
    },
    "files": {
      "href": "https://api.talkdeskdev.com/reports/calls/files/57add63204c799583700028e"
    }
  }
}
```

Once the report is generated, this endpoint will return a `303 See Other` status code, redirecting to the report download location. Please note that its body still returning the updated job status.

Response status code:
``` http
HTTP/1.1 303 See Other
```

Response headers:
``` http
Content-Type: application/json
Location: https://api.talkdeskdev.com/reports/calls/files/57add63204c799583700028e
```

Response body:
``` json
{
	"id": "57add63204c799583700028e",
	"status": "done",
	"timespan": {
		"from": "2016-07-29 00:00:00 UTC",
		"to": "2016-07-29 23:59:59 UTC"
	},
	"_links": {
		"self": {
			"href": "https://api.talkdeskdev.com/reports/calls/jobs/57add63204c799583700028e"
		},
		"files": {
			"href": "https://api.talkdeskdev.com/reports/calls/files/57add63204c799583700028e"
		}
	}
}
```

### Downloading a report file

A generated report can be downloaded several times after generation by using its canonical URL. This generates a signed URL in Amazon S3 which can be used to download the file. This will only be valid for a short period of time so if you need to re-download a report a new request to this endpoint should be made.

Issue an authenticated `GET` request to `/reports/calls/files/(id}`:

Headers:
``` http
Authorization: Bearer "INSERT_ACCESS_TOKEN_HERE"
```

A `302 Redirect` response to an S3 link is returned. If your HTTP client automatically follows redirects, the report will start downloading. Otherwise, issue a `GET` request to the signed S3 URL specified in the `Location` header to download.

``` json
{
  "total": 3,
  "entries": [
    {
      "callsid": "CA8699f82076ebfe864da32e681aefd1ba",
      "type": "inbound",
      "start_at": "2016-01-29 10:22:02",
      "end_at": "2016-01-29 10:22:20",
      "talkdesk_phone_number": "+18443265295",
      "talkdesk_phone_display_name": "support",
      "contact_phone_number": "+17207081229",
      "agent_name":"Alice",
      "agent_email": "alice@talkdesk.com",
      "total_time": 18,
      "talk_time": 10,
      "wait_time": 2,
      "hold_time": 0,
      "abandon_time": 0,
      "agent_speed_to_answer_time": 2,
      "disposition_code": "Not Interested",
      "notes": null,
      "agent_voice_rating": null,
      "ring_groups": "Product Support",
      "ivr_options": "1",
      "hangup": "handled",
      "is_in_business_hours": true,
      "is_callback_from_queue": null,
      "is_transfer": false,
      "handling_agent_name": null,
      "handling_agent_email": null,
      "recording_url": "https://media-files.talkdeskapp.com/calls/CA8699f82076ebfe864da32e681aefd1ba/recordings/0.mp3",
      "is_external_transfer": false,
      "is_if_no_answer": false,
      "is_call_forwarding": false
    },
    {
      "callsid": "CAa2c55bd75a3352f4244933b92942bf70",
      "type": "short_abandoned",
      "start_at": "2016-02-29 11:45:05",
      "end_at": "2016-02-29 11:45:08",
      "talkdesk_phone_number": "+18443265295",
      "talkdesk_phone_display_name": "support",
      "contact_phone_number": "+17207081229",
      "agent_name": null,
      "agent_email": null,
      "total_time": 3,
      "talk_time": 0,
      "wait_time": 3,
      "hold_time": 0,
      "abandon_time": 3,
      "agent_speed_to_answer_time": 0,
      "disposition_code": null,
      "notes": null,
      "agent_voice_rating": 0,
      "ring_groups": "Product Support",
      "ivr_options": "",
      "hangup": "waiting",
      "is_in_business_hours": true,
      "is_callback_from_queue": false,
      "is_transfer": false,
      "handling_agent_name": null,
      "handling_agent_email": null,
      "recording_url": null,
      "is_external_transfer": false,
      "is_if_no_answer": false,
      "is_call_forwarding": false
    },
    {
      "callsid": "CA730a06a24e9ba8c55a4e793ba4f4e751",
      "type": "inbound",
      "start_at": "2016-03-30 10:22:02",
      "end_at": "2016-03-30 10:35:02",
      "talkdesk_phone_number": "+18443265295",
      "talkdesk_phone_display_name": "support",
      "contact_phone_number": "+17207081229",
      "agent_name": null,
      "agent_email": null,
      "total_time": 780,
      "talk_time": 775,
      "wait_time": 5,
      "hold_time": 70,
      "abandon_time": 0,
      "agent_speed_to_answer_time": 5,
      "disposition_code": "Not Interested",
      "notes": null,
      "agent_voice_rating": 0,
      "ring_groups": "Product Support",
      "ivr_options": "",
      "hangup": "handled",
      "is_in_business_hours": true,
      "is_callback_from_queue": false,
      "is_transfer": false,
      "handling_agent_name": "Charlie",
      "handling_agent_email": "charlie@talkdesk.com",
      "recording_url": "https://media-files.talkdeskapp.com/calls/CA730a06a24e9ba8c55a4e793ba4f4e751/recordings/0.mp3",
      "is_external_transfer": true,
      "is_if_no_answer": false,
      "is_call_forwarding": false
    }
  ]
}
```

### Delete a report file

After a report has been generated, its corresponding file can be deleted using its canonical URL.

Issue an authenticated `DELETE` request to `/reports/calls/files/{id}`:

Headers:
``` http
Authorization: Bearer "INSERT_ACCESS_TOKEN_HERE"
```

A successful no content reponse will be returned:

``` json
HTTP/1.1 204 No Content
Server: Cowboy
Date: Fri, 05 Aug 2016 18:31:20 GMT
Content-Length: 0
Connection: keep-alive
Via: 1.1 vegur

```

A request to delete an already deleted report will be rejected:

``` json
HTTP/1.1 403 Forbidden
Server: Cowboy
Date: Thu, 11 Aug 2016 10:48:46 GMT
Connection: keep-alive
Content-Type: application/json
Content-Length: 30
Via: 1.1 vegur

```

### Canceling a report job

While a report is not yet generated, it is possible to cancel its creation by deleting the respective job.

Issue an authenticated `DELETE` request to `/reports/calls/jobs/{id}`

``` http
Authorization: Bearer "INSERT_ACCESS_TOKEN_HERE"
```

If the reports is not yet created, a successful no content response will be returned.

``` json
HTTP/1.1 204 No Content
Server: Cowboy
Date: Thu, 11 Aug 2016 10:50:02 GMT
Content-Length: 0
Connection: keep-alive
Via: 1.1 vegur

```

If the job is already finished and the reports has been created this operation is forbidden. However, its file can be deleted (see previous section).

``` json
HTTP/1.1 403 Forbidden
Server: Cowboy
Date: Fri, 05 Aug 2016 19:07:53 GMT
Connection: keep-alive
Content-Type: application/json
Content-Length: 30
Via: 1.1 vegur

```

### Inspecting previous report jobs

It is also possible to inspect the recent history of report generation jobs by using the list jobs endpoint.

Optionally, pagination and sorting can be specified using `page`, `per_page`, `order` and `order_by` as query parameters.

Issue an authenticated `GET` request to `/reports/calls/jobs?per_page=3`

``` http
Authorization: Bearer "INSERT_ACCESS_TOKEN_HERE"
```

A paginated list of the recent jobs ordered from the most recent ones is returned:

``` json
{
  "_embedded": {
    "jobs": [
      {
        "id": "57a07d18108e4282ae0000d6",
        "status": "canceled",
        "timespan": {
          "from": "2010-01-01 22:13:12 UTC",
          "to": "2016-05-01 22:13:12 UTC"
        },
        "_links": {
          "self": {
            "href": "https://api.talkdeskdev.com/reports/calls/jobs/57a07d18108e4282ae0000d6"
          },
          "files": {
            "href": "https://api.talkdeskdev.com/reports/calls/files/57a07d18108e4282ae0000d6"
          }
        }
      },
      {
        "id": "57a07c5a108e4248c30000bd",
        "status": "canceled",
        "timespan": {
          "from": "2010-01-01 22:13:12 UTC",
          "to": "2016-05-01 22:13:12 UTC"
        },
        "_links": {
          "self": {
            "href": "https://api.talkdeskdev.com/reports/calls/jobs/57a07c5a108e4248c30000bd"
          },
          "files": {
            "href": "https://api.talkdeskdev.com/reports/calls/files/57a07c5a108e4248c30000bd"
          }
        }
      },
      {
        "id": "57a4d60ed3188a8bbc0004a2",
        "status": "canceled",
        "timespan": {
          "from": "2010-04-29 00:00:00 UTC",
          "to": "2016-07-29 00:00:00 UTC"
        },
        "_links": {
          "self": {
            "href": "https://api.talkdeskdev.com/reports/calls/jobs/57a4d60ed3188a8bbc0004a2"
          },
          "files": {
            "href": "https://api.talkdeskdev.com/reports/calls/files/57a4d60ed3188a8bbc0004a2"
          }
        }
      }
    ]
  },
  "total": 41,
  "page": 1,
  "per_page": 3,
  "_links": {
    "self": {
      "href": "https://api.talkdeskdev.com/reports/calls/jobs?page=1&per_page=3"
    },
    "page": {
      "templated": true,
      "href": "https://api.talkdeskdev.com/reports/calls/jobs{?page,per_page,order_by,order}"
    },
    "next": {
      "href": "https://api.talkdeskdev.com/reports/calls/jobs?page=2&per_page=3"
    }
  }
}
```

Error responses and rate limits
-------------------------------

TBD

Help and Support
----------------

TBD
