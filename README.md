# Dinner, Already

One page for Ben, Ryan, Jared and Parker to pick a night for dinner.

## How it works

1. One person opens the site and taps **Start the dinner**.
2. Each person opens their **personal link** once (Ben has them). The page remembers them on that device, so the plain link works after that. Then they tap nights: once for yes, twice for if I must, three times for can't. Without a personal link the page is view only.
3. Any "can't" knocks a night out. The rest are ranked by yeses, then by date. The best three show up with who's in.
4. Anyone taps **Lock it in**. The ticket shows the date, time and place with buttons for Google Calendar, Apple Calendar (.ics), Outlook, OpenTable (table for four) and Google Maps.

Shortcuts: tap a weekday letter to set every one of those nights at once, tap **I'm wide open** to say yes to everything you haven't answered, and **Import my calendar** reads an .ics export and marks booked evenings (5 to 10 PM) as can't and all-day events as if I must. Weekly and daily repeating events are expanded.

**Nudge** and **Share** use the phone's share sheet, so the link goes straight into iMessage, WhatsApp, GroupMe or Slack.

## Sync

Votes sync through a Firebase Realtime Database in Ben's Google account (project `dinner-already`). There is one dinner at a time, stored at `/dinner/main`, so the plain site link always opens it. Each person's votes carry a timestamp and the newest wins per person, so two people voting at once never overwrite each other. The page checks for changes every 15 seconds. After a dinner, **Plan the next dinner** on the ticket clears the votes.

If Firebase can't be reached when someone starts a dinner, the page switches to link mode: the whole dinner is packed into the link itself, and each person shares the updated link after voting.

## Personal links

Each link carries `?me=<key>`. The page only stores SHA-256 hashes of the keys, so the keys aren't in the public source, and the key is removed from the address bar as soon as the page opens. This keeps people from voting as each other in the app. It is not account-level security: the database rules don't check the keys.

## Suggestion box

The suggestion box at the bottom writes one line per suggestion to `/suggestions` in the same database. The rules only allow adding new suggestions (500 characters max), so nobody can edit or delete them from the site. Claude reads the inbox when Ben asks for an update.

## Database rules

```json
{
  "rules": {
    ".read": false, ".write": false,
    "dinner": { "$room": { ".read": true, ".write": true, ".validate": "newData.hasChild('v')" } },
    "suggestions": {
      ".read": true,
      "$id": {
        ".write": "!data.exists()",
        ".validate": "newData.hasChildren(['text', 'ts']) && newData.child('text').isString() && newData.child('text').val().length > 0 && newData.child('text').val().length <= 500"
      }
    }
  }
}
```

## Files

- `index.html`: the whole app
- `meme.mp4`, `poster.jpg`: the TV clip

## Publish

Settings, Pages, Deploy from a branch, `main`, root folder.
