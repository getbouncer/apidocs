---
description: >-
  This document outlines the APIs available to integrate with CardScan.
---

# API Guide

## CardScanActivity

### Overview

`CardScanActivity` contains a companion object (can be used as a static class) which provides methods for launching the
CardScan flow and interpreting the results form the flow.

### Launching CardScan

#### CardScanActivity

<table>
  <tbody>
    <tr>
      <td rowspan="2"><pre lang="kotlin">fun warmUp(context: Context, apiKey: String, initializeNameAndExpiryExtraction: Boolean): Unit</pre></td>
    </tr>
    <tr>
      <td></td>
      <td>This </td>
    </tr>
  </tbody>
</table>


The default user interface for CardScan is available through the `CardScanActivity` class. This class provides static
methods for launching the scan activity and interpreting the results.

# Launching the Flow
```kotlin
class LaunchActivity : AppCompatActivity, CardScanActivityResultHandler {

    private const val API_KEY = "<your_api_key_here>"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_launch)

        findViewById(R.id.scanCardButton).setOnClickListener { _ ->
            CardScanActivity.start(
                activity = LaunchActivity.this,
                apiKey = API_KEY,
                enableEnterCardManually = true,
                enableExpiryExtraction = true,  // expiry extraction is in beta. See the comment below.
                enableNameExtraction = true  // name extraction is in beta. See the comment below.
            )
        }

        /*
         * To test name and/or expiry extraction, please first provision an API key, then reach out to
         * support@getbouncer.com with details about your use case and estimated volumes.
         *
         * If you are not planning to use name or expiry extraction, you can omit the line below.
         */
        CardScanActivity.warmUp(this, API_KEY, /* enableNameAndExpiryExtraction */ true)
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)

        if (CardScanActivity.isScanResult(requestCode)) {
            CardScanActivity.parseScanResult(resultCode, data, this)
        }
    }

    override fun cardScanned(scanId: String?, scanResult: ScanResult) {
        // a payment card was scanned successfully
    }

    override fun enterManually(scanId: String?) {
        // the user wants to enter a card manually
    }

    override fun userCanceled(scanId: String?) {
        // the user canceled the scan
    }

    override fun cameraError(scanId: String?) {
        // scan was canceled due to a camera error
    }

    override fun analyzerFailure(scanId: String?) {
        // scan was canceled due to a failure to analyze camera images
    }

    override fun canceledUnknown(scanId: String?) {
        // scan was canceled for an unknown reason
    }
}
```