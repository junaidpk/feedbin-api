Entry States
============

Entry states provide the users state information include read state and starred state. If an entry does not have an entry state, all attributes should be assumed false or null. The `updated_at` attribute should be used with the `since` parameter.

Get Entry States
----------------

 - `GET /v1/entry_states.json` will return all entry states.

```json
[
  {
    "entry_id": 56,
    "read": true,
    "starred": true,
    "updated_at": "2013-03-09T04:04:29.656288Z",
    "starred_updated_at": "2013-03-09T04:04:29.276038Z",
    "read_updated_at": "2013-03-09T04:04:29.656288Z"
  },
  {
    "entry_id": 23,
    "read": false,
    "starred": false,
    "updated_at": null,
    "starred_updated_at": null,
    "read_updated_at": null
  }
]
```

| Parameter      | Required | Example                                                                                                              |
| -------------- | -------- | -------------------------------------------------------------------------------------------------------------------- |
| `page: number` | false    | `GET /v1/entry_states.json?page=2` will get page two of the entry states                                             |
| `since: date`  | false    | `GET /v1/entry_states.json?since=2013-02-02T14:07:33Z` will get all entry states *updated* after ISO 8601 timestamp. |

Get Entry State
---------------

 - `GET /v1/entry_states/56.json` will return the entry state for entry_id `56`

```json
{
  "entry_id": 56,
  "read": true,
  "starred": true,
  "updated_at": "2013-03-09T04:04:29.656288Z",
  "starred_updated_at": "2013-03-09T04:04:29.276038Z",
  "read_updated_at": "2013-03-09T04:04:29.656288Z"
}
```

**Status Codes**

- `200 OK` will be returned if found
- `404 Not Found` will be returned if there is no saved entry state

Update/Create Entry States
--------------------------

- `POST v1/entry_states.json`

Entry states can be modified and created in bulk. Since multiple clients might be updating the entry state, it is important to send *when* the state was changed. This prevents the case where an entry state is changed offline, and before it has a chance to sync, it is changed again. In this case the entry state update will be ignored. Clients should always save the time an action was performed, when it is performed rather than when it is sent.

If the timestamp that is sent is older than the timestamp that is saved, **the update will be ignored and does not trigger an error**. Instead it shows up in the succeeded array with the correct state.

**Request**

```json
{
  "entry_states": [
    {
      "entry_id": 1,
      "read": true,
      "starred": true,
      "starred_updated_at": "2013-03-09T04:04:29.656289Z",
      "read_updated_at": "2013-03-09T04:04:30.656288Z"
    },
    {
      "entry_id": 3,
      "read": true,
      "starred": true,
    }
  ]
}
```

| Parameter                        | Required    | Example                                      |
| -------------------------------- | --------    | -------------------------------------------- |
| `entry_id: number`               | true        |                                              |
| `starred_updated_at: date`       | conditional | Required when attempting to change `starred` |
| `read_updated_at: date`          | conditional | Required when attempting to change `read`    |
| `read: boolean`                  | false       |                                              |
| `starred: boolean`               | false       |                                              |


**Response**

```json
{
  "succeeded": [
    {
      "entry_id": 1,
      "read": true,
      "starred": true,
      "starred_updated_at": "2013-03-09T04:04:29.656289Z",
      "read_updated_at": "2013-03-09T04:04:30.656288Z"
    }
  ],
  "failed": [
    {
      "entry_id": 3,
      "errors": [
        {
          "field": "read_updated_at",
          "message": "must be present when updating read"
        },
        {
          "field": "starred_updated_at",
          "message": "must be present when updating starred"
        }
      ]
    }
  ]
}
```