# vatnode for Make

Source of the vatnode custom app for [Make](https://www.make.com): validate EU VAT numbers against VIES, monitor them for changes, and read EU VAT rates.

The API it talks to is documented at [vatnode.dev/docs](https://vatnode.dev/docs).

## Modules

| Module | Type | What it does |
| --- | --- | --- |
| Watch VAT Events | Instant trigger | Fires when a monitored VAT number stops being valid, becomes valid again, or the trader name or address changes in VIES. |
| Validate a VAT Number | Action | Checks a VAT number against VIES; returns validity, trader name and address, country rates and the VIES consultation number. |
| Monitor a VAT Number | Action | Adds a VAT number to daily monitoring. |
| Get VAT Rates | Action | Standard, reduced, super-reduced and parking rates for a European country. |
| Make an API Call | Universal | Any authorized call to the vatnode API. |

## Connection

Type `basic`, one field: a vatnode API key from the [dashboard](https://vatnode.dev/dashboard/api-keys), sent as `Authorization: Bearer <key>`. The header is sanitized out of the logs in both the base and the connection.

The connection is verified against `GET /v1/key`, which spends no quota, and the connection label shows the key label and environment.

Live keys (`vat_live_`) work everywhere. Test keys (`vat_test_`) validate the `XX` fixture numbers only and are rejected by monitoring and webhooks, so **Watch VAT Events** and **Monitor a VAT Number** need a live key.

## How the instant trigger works

1. `Attach` registers the Make webhook URL with `POST /v1/webhooks`, storing the returned id as `vatnodeWebhookId`.
2. A vatnode webhook is created unverified, so `Attach` immediately fires `POST /v1/webhooks/{id}/test`. The successful test delivery is what activates it.
3. The webhook `Api` drops that verification delivery (`event != 'webhook.test'`) so it never becomes a bundle.
4. `Detach` deletes the webhook by the stored id.

## Repository layout

The files mirror the sections of a Make custom app, so each one is pasted into (or synced with) the matching tab in the Make app editor:

```
app.json                            app metadata + module list with typeIds
general/Base.imljson                base URL, auth header, error mapping, log sanitization
connections/vatnode/Api.imljson     connection verification + label
connections/vatnode/Parameters.imljson
modules/<module>/Api.imljson        request definition
modules/<module>/Parameters.imljson input fields
modules/<module>/Interface.imljson  output fields
modules/<module>/Samples.imljson    sample output
webhooks/vat-events/Api.imljson     payload handling, condition, respond
webhooks/vat-events/Attach.imljson  register + verify
webhooks/vat-events/Detach.imljson  unregister
```

Editing is done either in the Make app editor UI or with the **Make Apps Editor** VS Code extension pointed at this folder.

## Review checklist

Make's app review requires, among other things: sanitized credentials (done in `general/Base.imljson` and the connection), error handling in base and connection, a universal module (`make-api-call`), test scenarios exercising every module including one that produces an error, and no personal data in test data.

## License

MIT © Iurii Rogulia
