# Secure Counting REST API

After you've integrated [setup secure counting in your iOS app](secure_counting_ios.md), you can check the counts for devices server side. If you detect that a device has added too many cards for this device or logged in with too many different accounts, you can act on it appropriately (e.g., deny the transaction or challenge them with Card Verify). You can also increment counts for your iOS clients server-side for a more secure integration.

## POST /v1/secure_counting/:vendor_id

Get the current counts for a device. Note: we use POST because the DeviceCheck tokens can be large (4KB), but this request is idempotent.

### Request
Authentication required

### Fields

**devicecheck_token** (string)
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
**counts** (_Map[string, int]_)
A map 

