# Remind volunteers when and where to be with Twilio, Airtable, and Zapier 
_Helping COVID-19 community response teams coordinate neighbors helping neighbors. A Twilio x DEV Hackathon submission for Engaging Engagements and/or Interesting Integrations._ 

## Automating volunteer reminder texts for COVID-19 community response teams 

If we follow Mr. Rogers’ advice to [look for the helpers](https://www.youtube.com/watch?v=-LGHtc_D328) right now, we will find them. Team Rubicon has [distributed hundreds of thousands of masks and gloves](https://twitter.com/TeamRubicon/status/1255534812212817920) to healthcare workers in Colorado. In New York City, Backpacks for the Street has [shared 1,200 COVID-19 prevention kits](https://www.goodmorningamerica.com/living/story/group-handing-backpacks-full-supplies-nycs-homeless-combat-70330087) with residents experiencing homelessness. Yesterday, TogetherSF volunteers delivered [35,000 steaks](https://www.sfchronicle.com/bayarea/article/Bay-Area-food-banks-have-a-surprise-for-15230171.php) to Bay Area food banks.

Organizing these efforts takes a lot of communication, and it can be challenging to stay on top of it all, especially as a resource-constrained nonprofit. It would be impossible to manually text every volunteer who signs up for a shift to remind them when and where to be, but [Airtable](https://airtable.com), [Twilio](https://twilio.com/), and [Zapier](https://zapier.com/) can automate that for us. 

### Setting up an Airtable Base 

![David Rose asks Alexis Rose if it is a spreadsheet](https://media.giphy.com/media/l0ExoJBGYUelaOiME/giphy.gif)

[Airtable](https://airtable.com) is a database spreadsheet hybrid that helps with all things organization. The nonprofit I worked with picked Airtable from the start for keeping track of volunteers and volunteering opportunities, so it made sense to me to dive into their existing system and see how I could work with it. 

After looking at their setup and reading up on Airtable’s [docs](https://airtable.com/api) and [community forums](https://community.airtable.com/), I learned I would need to change a few things to get automated text reminders working. 

Have a look at the [Volunteer Coordination Database template](https://airtable.com/invite/l?inviteId=inv9JYUH1AMIDWkt1&inviteToken=142155a70ea25d910a65c301ad1288769800d97a8920f44d54d1bf63d0e051ba), and I’ll walk you through what’s special. 

The **Events** tab (or table) lists out the volunteering opportunities that come our way, including who asked for help (typically somebody in the **Needs** table), the kind of help they're asking for, and when and where we've scheduled a time to support.

The first change I made was to make the _When_ field in **Events** more granular, adding in a timestamp. It took me longer than I’d like to admit to spot the “Include a time field” toggle in Airtable, but can I really be blamed -- what is time these days? 

![Time Toggle within Airtable](https://dev-to-uploads.s3.amazonaws.com/i/mbtnunnuymb81vfs8mbx.png)

While there is a date type for columns in Airtable, there is no separate time type, so I needed to come up with a workaround. I knew I needed to send a text reminder _n_ hours before an event, so a column that calculated the number of hours until an event cold fit my use case. I added in an _Hours Until Event_ formula column to sort it out with `DATETIME_DIFF({When}, NOW(), 'hours')`. 

![Hours Until Event column in Airtable setup](https://dev-to-uploads.s3.amazonaws.com/i/d07nohbwqnraux6hj4dk.png)

The nonprofit I worked with already included a _Matched Volunteers_ column to list out the volunteers that would be at an event. Next up I needed to get all of those volunteers’ phone numbers into *Events* too. I used a Lookup column type to grab the phone numbers from the _Volunteers Matched_ linked record. 

![Volunteer Phone Numbers column in Airtable](https://dev-to-uploads.s3.amazonaws.com/i/u871lee2d0471f2g3vcu.png)

This next part might spare you the time I spend debugging later (Again, what is time, anyway?). We need to convert our numbers into [E.164 format](https://www.twilio.com/docs/glossary/what-e164) so that we can text them. I created a new _Formatted Phone Numbers_ column with a formula that added the necessary +1 for this US-based nonprofit (it’s really +{CountryCode}), replaced all of the nonnumeric characters with empty strings using SUBSTITUTE, and made sure all of the volunteers’ numbers would be in a comma separated list using ARRAYJOIN: `SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(ARRAYJOIN({Volunteer Phone Numbers}, ","),"(","+1"),")",""), "-","")`

![Formatted phone number column setup](https://dev-to-uploads.s3.amazonaws.com/i/jhxqf5ast3vmvhd5k9pq.png)

With that all in place, I set up an [Airtable View](https://support.airtable.com/hc/en-us/articles/202624989-Guide-to-views) to filter for records where _Hours Until Event_ is equal to when I wanted to send the text, so four hours before. I also added a filter to make sure we’re only looking at events that haven’t happened yet, where their _Hours Until Event_ hasn’t dropped below zero. I called this View _Volunteer Reminder Texts_.

![Airtable View setup](https://dev-to-uploads.s3.amazonaws.com/i/13jdjwus3bxmgjxwh35z.png)

Airtable should be good to go from here! Time to leave spreadsheets behind for a minute and head over to Twilio. 

### Setting up Twilio

![Phone with blinking heart nose](https://media.giphy.com/media/ff6IT8IzC5hEQ/giphy.gif) 

Developers use [Twilio](www.twilio.com) to programmatically send and receive calls and texts, but the limit really does not exist. [Chloe Condon](https://twitter.com/ChloeCondon) and I once used it to build a [Mean Girls’ day bot](https://dev.to/twilio/trying-to-make-fetch-errr-a-post-request-happen-12ad), and [Twilio Champions](https://www.twilio.com/champions) get up to all kinds of projects.  

From my experience with Twilio, I knew it could handle texting this nonprofit’s volunteers. Here’s where to [sign up for an account](www.twilio.com/referral/avaKmb) if you don’t have one already. You will also need a Twilio phone number. 

### Putting it all together with Zapier 

[Zapier](https://zapier.com/) automates connecting different apps together. You can sign up [here](https://zapier.com/sign-up/). 

Zapier can [sync up Airtable and Twilio](https://airtable.com/integrations/twilio). I clicked “Connect these Apps” and had my [Airtable API key](https://support.airtable.com/hc/en-us/articles/219046777-How-do-I-get-my-API-key-) and [Twilio API key](https://www.twilio.com/docs/iam/keys/api-key) both handy to step through the process.

Once the accounts were linked, I specified that when a new record popped up in an Airtable View, specifically  _Volunteer Reminder Texts_, Zapier should tell Twilio to text those volunteers. Since my _Hours Until Event_ column automatically calculates, new records are added to the View automatically, meaning the texts will be automatic too. 

And, since Zapier pulls in all the data from the view, it also automatically pulls in the specific _When_ and _Where_ for each volunteer opportunity, so your texts will be personalized each time once the Zap is turned on. 

![Zapier setup](https://dev-to-uploads.s3.amazonaws.com/i/qwyruoibqy3htww6wfaa.png)

You should see whatever you specify in Zapier show up in your test, and they’ll show up again for hours before the event too: 

![Screenshot of text](https://dev-to-uploads.s3.amazonaws.com/i/5rlg74b5a14g10k3lrdg.PNG)

## After the event

And that was it! There are lots of things I could do to improve, and I know there are loads more things I can do with Twilio and Airtable. We’re considering automated feedback surveys once an event is done, for example. 

I’m excited to explore all the options I can. I thought these automated reminders would involve a lot more time and work, and I was surprised to find a practically no code solution. This was another lesson for me that it’s always good to start looking at systems already in place, meeting your users, in my case volunteer organizers, where they’re at, and understanding their needs before jumping into building something from scratch. I love when existing tools free up time to be a helper in other ways. 

### Resources 
** [Volunteer Coordination Database template](https://airtable.com/invite/l?inviteId=inv9JYUH1AMIDWkt1&inviteToken=142155a70ea25d910a65c301ad1288769800d97a8920f44d54d1bf63d0e051ba)
** [Dev.to post]
** If you need help with this, or have questions about anything else, say hi any time at `hello@kimberlee.dev` 

