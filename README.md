
# Automating text reminders to volunteers for COVID-19 community response teams 

## About 
This Github README walks through how to automate volunteer text reminders using Twilio, Airtable, and Zapier. It was originally built to help COVID-19 community response teams coordinate neighbors helping neighbors, and submitted to a Twilio x DEV Hackathon under Engaging Engagements and/or Interesting Integrations. 

## How it works 
This project integrates Twilio and Airtable through Zapier to send automatic text reminders whenever a volunteer has an event coming up in fewer than four hours. 

## How to use it 
_You can find more details instructions over at Dev.to_
1. Copy this [Airtable Base](https://airtable.com/apppKrJy5Y3wCznHj).
2. Add in your own records. You can send out the Volunteer sign up form, or add volunteers manually. You'll also need to add Needs (lists of asks from the community), and Events, the time when you're matching volunteers to fulfill those Needs.
3. Create a Twilio account if you don't have one already. 
4. Create a Zapier account if you don't have one already. 
5. Connect Zapier with Airtable, and then Zapier with Twilio. 
6. Set up a Zap to send a text whenever a new record pops into the 'Volunteer Reminder Texts' View. 

## Set up 

### Requirements 
* An [Airtable account](https://airtable.com/tblNvv1RVuxVxgI8W/viwbFcn3F34QmJMGX?blocks=hide) and a copy of [this Base]()
* [Twilio account](www.twilio.com/referral/avaKmb)
* [Zapier account](https://zapier.com/)

### Resources 
* Dev.to post 

### Contributing 
* Reach out to `hello@kimberlee.dev` if you're interested in contributing. 

### License 
[MIT](https://opensource.org/licenses/mit-license.html)
