# System Brief: KAM Supply Intelligence Agent

## What the system does

This system helps a Key Account Manager ask a question in Slack and get back a structured answer about a car-rental client. It combines two internal data sources:

- Salesforce, which holds the client profile
- Supabase, which holds supplier and product information

In plain language, the system answers questions such as:

- How many suppliers does a client have?
- Which suppliers work with a client?
- What products, routes, or rate details exist for that client?

It is designed for day-to-day KAM work, particularly for preparing for client meetings and checking commercial or operational details, eliminating the need to raise tickets and wait for delayed responses.

## Inputs

The main input is a free-text question written by a KAM in Slack.

Examples of input data:

- Client name, such as `Check24`, `Autoslash`, or `HappyCar`
- Question text written in normal language
- Optional export request words like "Excel", "export", or "xlsx"

The system also reads data from internal business systems:

- Salesforce account data: client name, account number, account tier, business model, contract status, and assigned KAM
- Supabase supplier and product data: supplier names, supplier codes, rate codes, rate types, product type, route information, and active/inactive status

Personal data is limited, but not zero. The Salesforce record includes the assigned KAM name, which is a person's name. The Slack question itself may also contain names or other details if the user types them. The system is not built to collect sensitive personal data.

## Outputs

The system returns a written answer in Slack. The answer usually contains:

- A client profile section from Salesforce
- An operational data section from Supabase
- The exact SQL query that was used
- A cost and token summary

If the KAM asks for an export and the question type supports it, the system also creates an Excel file and posts it to the same Slack thread.

So the output can be:

- A written answer
- A file attachment
- A warning or error message if the client is not recognized or if the data query fails

## Who is affected by the output

The direct users are the KAM team of a car-rental company. They use the answer to support client management and meeting preparation.

The output also affects any internal person who depends on KAM decisions or client reporting, because the KAM may use the result to guide follow-up actions, explain account status, or prepare commercial discussions.

## Human review

There is no separate human approval step built into the system before the output is delivered. The answer is generated automatically and posted to Slack.

In practice, the KAM reviews the answer in Slack after it is returned. If a file is generated, the KAM can open and check it before using it. But the system itself does not wait for a manager or analyst to approve the result before sending it.

## Who built it

The system was built by Dilia Navarro, the project author.

## Who would use it in production

In production, the intended users would be the client's KAM team. The system is meant to support their daily work by giving them fast answers about client profiles, suppliers, and product details from Slack.
