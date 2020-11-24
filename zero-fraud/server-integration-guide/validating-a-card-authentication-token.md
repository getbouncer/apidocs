# Validating a card authentication token

## **POST `/v1/token/validate`**

The `/v1/token/validate` endpoint accepts a CardVerifyToken generated from `/v1/card/verify` endpoint and returns whether the Card Verify attempt corresponding to that token is valid or not. On invalid requests, the response will contain a list of failure\_reasons around why Bouncer believes this Card Verify scan to be suspicious.

### HTTP Request

**Authentication Required**

{% tabs %}
{% tab title="Request" %}
### **Body Parameters**

`token: string` CardVerifyToken from the `/v1/card/verify` endpoint

### **Sample Request**

```bash
curl -X POST "https://api.getbouncer.com/v1/token/validate"
  -H "Content-Type: application/json"
  -H "Authorization: Bearer API_KEY"
  -d '{
    "token"             : "CARD_VERIFY_TOKEN"
  }'
```
{% endtab %}
{% endtabs %}

