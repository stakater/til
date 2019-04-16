# IMC Too many Status Pages Issue

During the addition of new monitors for IMC, a bug was causing the uptimerobot unit tests to fail. 

### ISSUE
After debugging two issues were found:

- Status pages created for monitors were not being deleted properly causing the number of status pages to raise to `65`.

- The way status pages are being pulled in `GetStatusPagesForMonitor` is not correct because the `uptimerobot` api return `50` results(status pages information) max per request in reverse chronological order. It means that in tests those status pages are being pulled that have not been assigned to any monitor, an example is given below to pull the first `50` records:

`CURL REQUEST`
```bash
curl -X POST -H "Cache-Control: no-cache" -H "Content-Type: application/x-www-form-urlencoded" -d 'api_key=XXXXXXX&format=json' "https://api.uptimerobot.com/v2/getPSPs"
```

`RESPONSE`

```json
{
  "stat": "ok",
  "pagination": {
    "offset": 0,
    "limit": 50,
    "total": 68
  },
  "psps": [
    {
      "id": 70705,
      "friendly_name": "status-page-test",
      "monitors": 0,
      "sort": 1,
      "status": 1,
      "standard_url": "https://stats.uptimerobot.com/j8xYoIrVW",
      "custom_url": ""
    },
    {
      "id": 70710,
      "friendly_name": "status-page-test",
      "monitors": 0,
      "sort": 1,
      "status": 1,
      "standard_url": "https://stats.uptimerobot.com/oyDZwcnwX",
      "custom_url": ""
    },
    ...
    psp no 50]
}
```
In the above example it can be seen that status pages are not assigned to any monitors.

`CURL REQUEST TO PULL NEXT 15 records`

```bash
curl -X POST -H "Cache-Control: no-cache" -H "Content-Type: application/x-www-form-urlencoded" -d 'api_key=XXXXXXXX&format=json&offset=50'  "https://api.uptimerobot.com/v2/getPSPs"
```

`RESPONSE`

```json
{
  "stat": "ok",
  "pagination": {
    "offset": 50,
    "limit": 50,
    "total": 65
  },
  "psps": [
    {
      "id": 71582,
      "friendly_name": "status-page-test-1",
      "monitors": [
        782434861
      ],
      "sort": 1,
      "status": 1,
      "standard_url": "https://stats.uptimerobot.com/PQPjBFwjk",
      "custom_url": ""
    },
    {
      "id": 71583,
      "friendly_name": "status-page-test-2",
      "monitors": [
        782434861
      ],
      "sort": 1,
      "status": 1,
      "standard_url": "https://stats.uptimerobot.com/Q7QkDcQ2W",
      "custom_url": ""
    },
    ...
    RECORD # 65]
}
```

As we can see in above example the in last `15` records status pages are assigned to a monitor.



### SOLUTION

These suggestions will resolve the above issues:

- Add a few tests in the uptimerobot unit tests file to remove everything (status pages and monitors) before and after any other tests execute.

- Before performing any tests. First pull all the entries available and then perform the test logic so that the test can scope all the entries.




