# XPath Filtering tips

Filename is indicating to where these xpath filterings are intended to be used
- file starting with
  - Customview - Event Viewer custom views
  - Subscription - Windows Event Forwarding subscription filter
  - DCR - Azure monitoring DCR XPath rule

Other file prefixes are just tests, you can try those but might be it is not working

Note, you can potentially take some "custom view" xpath and copy/paste it to DCR and it might work just fine

Reminder:
Good amount of the **events** and **audit policies** can be found from here:
- [Securing AD - Events to monitor](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor)
- [Spreadsheet details the security audit events](https://www.microsoft.com/en-us/download/details.aspx?id=50034)