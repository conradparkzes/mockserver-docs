---
sidebar_position: 1
slug: /
---

# Mock Server - Intro

This guide explains how to **simulate an API call** from the mobile operators available in Madagascar on our server.

### What you'll need

Ensure **Postman** is downloaded.

### What does the mock server do?
For each provider (MVola, Orange Money, Airtel Money), the mock server ...
* Generates tokens
* Initiates a payment
* Returns the status of the transaction

... all by using mock data only to fully simulate server functionality.

## Run Mock Server

Click below button to run Mock Server via the Postman collection :
<a href="https://conradcoyanda-parkzes.postman.co/collection/45611146-6b65e19d-ce08-4942-bda3-60482f49feda?source=rip_markdown" target="_blank">
<img src="https://run.pstmn.io/button.svg" alt="Run in Postman" width="128" height="32" />
</a>
&nbsp; <!-- space in between lines -->

Your Postman window should look something like this :
![Postman display when "Run in Postman" button is clicked](\img\run-in-postman-opening-window.png)
Then, navigate to the "Run" button, circled in red. This will take you to the "Runner" tab, where you can control 
how you would like to run the test suite. Default functional and performance settings are fine, and you can select 
and deselect members of the run sequence.
&nbsp; <!-- space in between lines -->

![Postman display when "Runner" button is clicked](\img\run-in-postman-runner.png)
