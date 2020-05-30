---
title: Local CosmosDB Emulator When Port 8081 is Unavailable
date: 2020-05-30 12:00:03
---

The fact that Microsoft provides a [local CosmosDB emulator](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator) is amazing. We don't have to pay to run an instance for development and we don't need to expose the cloud database to the general internet or set up some VPN for access. Win!

However, the company I'm currently working for installs McAfee on all workstations. McAfee uses port 8081 for something called SuperAgent. Unfortunately, that is the exact port that the CosmosDB emulator wants to use.

Lucky for us, the emulator provides a way to specify the port. I tried 8082, but that caused the system to report that it was started, but not be available at all. No errors were reported, it was just completely unresponsive.

That's fine. . . I don't think any of my colleagues are [running Runescape](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers#Well-known_ports) on their workstations, so let's use port 43594.

PowerShell modules are provided, so let's use those:

```powershell
Import-Module "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules\Microsoft.Azure.CosmosDB.Emulator"
Start-CosmosDbEmulator -Port 43954
```

Great! Now to open the data explorer in Firefox... nope. I received the following errors:

Error while refreshing databases: {"code":401,"body":{"code":"Unauthorized","message":"Authorization header doesn't confirm to the required format. Please verify and try again.\r\nActivityId: 99709b58-6bcb-4c2e-8a14-bef5ea828f48, Microsoft.Azure.Documents.Common/2.11.0"},"headers":{"access-control-allow-credentials":"true","access-control-allow-origin":"","content-location":"https://localhost:43594/offers","content-type":"application/json","date":"Tue, 26 May 2020 20:23:59 GMT","server":"Microsoft-HTTPAPI/2.0","x-firefox-spdy":"h2","x-ms-activity-id":"99709b58-6bcb-4c2e-8a14-bef5ea828f48","x-ms-gatewayversion":"version=2.11.0","x-ms-throttle-retry-count":0,"x-ms-throttle-retry-wait-time-ms":0},"activityId":"99709b58-6bcb-4c2e-8a14-bef5ea828f48"}

Error while querying offers: {"code":401,"body":{"code":"Unauthorized","message":"Authorization header doesn't confirm to the required format. Please verify and try again.\r\nActivityId: 99709b58-6bcb-4c2e-8a14-bef5ea828f48, Microsoft.Azure.Documents.Common/2.11.0"},"headers":{"access-control-allow-credentials":"true","access-control-allow-origin":"","content-location":"https://localhost:43594/offers","content-type":"application/json","date":"Tue, 26 May 2020 20:23:59 GMT","server":"Microsoft-HTTPAPI/2.0","x-firefox-spdy":"h2","x-ms-activity-id":"99709b58-6bcb-4c2e-8a14-bef5ea828f48","x-ms-gatewayversion":"version=2.11.0","x-ms-throttle-retry-count":0,"x-ms-throttle-retry-wait-time-ms":0},"activityId":"99709b58-6bcb-4c2e-8a14-bef5ea828f48"}

So I gave Edge a try and that worked like a charm. Lesson learned, Microsoft loves some Edge.
