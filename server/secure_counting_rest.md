# Secure Counting REST API

After you've integrated [setup secure counting in your iOS app](secure_counting_ios.md), you can check the counts for devices server side. If you detect that a device has added too many cards for this device or logged in with too many different accounts, you can act on it appropriately (e.g., deny the transaction or challenge them with Card Verify). You can also increment counts for your iOS clients server-side for a more secure integration.

## POST /v1/secure_counting/:vendor_id

Get the current counts for a device. Note: we use POST because the DeviceCheck tokens can be large (4KB), but this request is idempotent.

### Request
Authentication required, (Vendor ID)[https://developer.apple.com/documentation/uikit/uidevice/1620059-identifierforvendor] from the iOS device for the path of the object.

### Fields

**devicecheck_token** (_string_)
The [DeviceCheck token](https://developer.apple.com/documentation/devicecheck/dcdevice/2902276-generatetoken) obtained from the iOS device.

### Sample Request

```bash
curl -X POST "https://api.getbouncer.com/v1/secure_counting/test_vendorid"
  -H "Content-Type: application/json"
  -H "Authorization: Bearer API_KEY"
  -d '{ "devicecheck_token": "test_devicecheck_token"}'
```

### Response
A **SecureCount** response JSON object

#### SecureCount
**counts** (_Map[event: string, Map["count": int, "maximum": int]]_)
A map of events, each event has a current _count_ and a _maximum_ for the maxiumum value that this count can reach.

**last_reset_at** (string)
ISO8601 format of the timestamp when we detected the last device factory reset or app uninstall.

### Response Examples
**Success**
```javascript
{
  "counts": {
    "cards_tokenized": {"count": 4, "maximum": 7},
    "successful_logins": {"count": 2, "maximum": 11}
  },
  "last_reset_at": null
}
```

**Invalid device check token** If the DeviceCheck token failed Apple's basic validation we return an error
```javascript
{
  "failure_reasons": ["invalid_devicecheck_token"]
}
```

## POST /v1/secure_counting/:vendor_id/increment
### Request
Authentication required, (Vendor ID)[https://developer.apple.com/documentation/uikit/uidevice/1620059-identifierforvendor] from the iOS device for the path of the object.

### Fields

**devicecheck_token** (_string_)
The [DeviceCheck token](https://developer.apple.com/documentation/devicecheck/dcdevice/2902276-generatetoken) obtained from the iOS device.

**event** (_string_)
The event that you want to increment a count for

**user_id** (_string_)
The userId for the current user

### Sample Request

```bash
curl -X POST "https://api.getbouncer.com/v1/secure_counting/test_vendorid/increment"
  -H "Content-Type: application/json"
  -H "Authorization: Bearer API_KEY"
  -d '{ "devicecheck_token": "test_devicecheck_token", "event": "card_tokenized", "user_id": "kingst"}'
```

### Response
A **SecureCount** response JSON object

#### SecureCount
**counts** (_Map[event: string, Map["count": int, "maximum": int]]_)
A map of events, each event has a current _count_ and a _maximum_ for the maxiumum value that this count can reach.

**last_reset_at** (string)
ISO8601 format of the timestamp when we detected the last device factory reset or app uninstall.

### Response Examples
**Success**
```javascript
{
  "counts": {
    "cards_tokenized": {"count": 5, "maximum": 7},
    "successful_logins": {"count": 2, "maximum": 11}
  },
  "last_reset_at": "2019-10-23T00:48:07+0000"
}
```

**Invalid device check token** If the DeviceCheck token failed Apple's basic validation we return an error
```javascript
{
  "failure_reasons": ["invalid_devicecheck_token"]
}
```
