# Integration overview

### Role of the **Bouncer Mobile SDKs:**

* Scan the number side of a card
* Enable users to enter CVV and ZIP codes to finish a scanned card entry
* Enable users to enter all card details
* Track and log signals about the device and user, and send these signals back to Bouncer

**Note:** The Bouncer SDK will not directly expose whether this was a legitimate scan of a real card to the client App. It’ll be up to the client App to make a subsequent server-to-server call to get the authentication results.

### **Role of the Bouncer Server:**

* The Mobile SDKs and JavaScript beacons communicate with Bouncer's servers directly to keep integrations quick and easy. As long as you use the Bouncer SDKs, appropriate ViewControllers and Activities, and instrument the payments forms, Bouncer will communicate results back automatically.
* Your server needs to make two API calls to Bouncers server \(1\) when users tokenize a card and \(2\) at transaction time to get a fraud score:
  * `/insight/v1/tokenize`\(server-to-server\): Called when the App Server tokenizes a card successfully
  * `/insight/v1/score` \(server-to-server\): Risk assessment at transaction time, called for all transactions

