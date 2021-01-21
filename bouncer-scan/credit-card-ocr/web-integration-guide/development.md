---
description: Developing the cardscan-web library.
---

# Development Guide

## Clone the repo and set up the demo

The [cardscan repository](https://github.com/getbouncer/cardscan-web) contains a demonstration app for the CardScan product. To build and install this library follow the following steps:

1. Clone the repository from github

   ```bash
   git clone https://github.com/getbouncer/cardscan-web && cd cardscan-web
   ```
   
1. Install dependencies using NPM

   ```bash
   npm install
   ```

1. Build the library using NPM and webpack

   ```bash
   npm run build
   ```
   
   This will create a `dist` directory with the compiled javascript.

1. Start a local server to host the scanner and demo app

   ```bash
   python -m SimpleHTTPServer 8000
   ```

1. Navigate to http://localhost:8000 to view a sample app.
