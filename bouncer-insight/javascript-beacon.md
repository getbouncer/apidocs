# JavaScript Beacon

Bouncer Insight utilizes signals gathered from our JS Beacon for detecting anomalous behaviors

```javascript
<script type="text/javascript">
  var _user_id = "kingst"; // the user id, if known.
  var _bip = window._bip || {};
  window._bip = _bip;
  _bip = {
    "userId": _user_id,
    "clientId": "YOUR_BOUNCER_CLIENT_ID",
    "sessionId": "Session ID for this user session (optional)"
  };
  (function () {
    function loadBeacon() {
      var script = document.createElement('script');
      script.type = 'text/javascript';
      script.async = true;
      script.src = "https://d28npvtfvv63k9.cloudfront.net/bip.js";
      var first = document.getElementsByTagName('script')[0];
      first.parentNode.insertBefore(script, first);
    }
    if (window.attachEvent) {
      window.attachEvent('onload', loadBeacon);
    } else {
      window.addEventListener('load', loadBeacon, false);
    }
  })();
</script>
```



