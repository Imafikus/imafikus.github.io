---
layout: post
title: 'Backfilling our prod database with a discord channel'
date: 2025-01-27 22:48:00 +0200
categories: startup
---

## The Problem

I was checking something in our prod database on [Notify Me](https://notify-me.rs){:target="\_blank"} and I realized that the recently created user didn't have a `created_at` field set, which was super weird, since we are doing this automatically for every new database insert. After checking few more users I've realized that all of our recent sign ups that were done via Google OAuth (almost everyone signs up this way) had an empty `created_at` field.

While this wasn't a problem for the user, since we didn't used the field for any user facing functionalities, it created a problem for us, since `created_at` field is very important for our statistics and it was very useful in the past. Especially when we introduced paid plans and we wanted to enable free access to some paid features to our early users.

## Our Database Setup

We are using a self-hosted version of [MongoDB](https://www.mongodb.com/){:target="\_blank"}{:rel="noopener noreferrer"}, and we are using [Beanie ODM](https://beanie-odm.dev/){:target="\_blank"}{:rel="noopener noreferrer"} to interact with the database. We have various models that respond to a specific table in the database, (obviously) including the user model as well. Our user model has previously mentioned `created_at` field which we were automatically setting on every insert event.

```python
# code that runs automatically on every insertion
    @before_event(Insert)
    def set_default_fields(self):
        if self.telegram_token != "":
            self.telegram_token = str(uuid4())
        if self.created_at == "":
            self.created_at = get_current_time()
```

## The Solution

Since I realized that the bug was present only on Google OAuth routes, we quickly pin-pointed the problem to a file that handles the oauth route in our webserver, and, almost instantly, we spotted the problem.

Whenever user tries to sign up using oauth, we firstly check whether the user already exists, since in that case it can be automatically logged in. If the user doesn't exist, we are creating the new user object and we are inserting it in the database, only problem was that instead of using `user.insert()` we were using `user.save()`. Insert command will always create a new entry in the database, while save will update the element if it exists, otherwise it will create a new one.

From a logical standpoint, both methods were fine in our case, since we were always creating a new users, but **we didn't have automatic triggers** on `.save()` events, which caused our used model to be saved with the default `created_at` instead of the current time.

Fix was a simple change from `user.save()` to `user.insert()`.

## Backfilling Data

While the fix was pretty easy, we were still left with around 700 users for whom we had no information about their creation date, and that was a problem.

That's were the headline of this blog post comes in.

We have an internal discord server which is our main way of communication for all things related to [Notify Me](https://notify-me.rs){:target="\_blank"}. Among many channels we have, there is a channel called `user-alerts` where we automatically receive a message whenever a new user has signed up, when user leaves us a review and when user deletes the account. We realized that that channel contained all the necessary info we needed to backfill our database, since we had that channel for years, and it contained info about (almost) every single user that ever signed up to [Notify Me](https://notify-me.rs){:target="\_blank"}.

Messages we needed where in the following format:

`User **foo@bar.com** successfully signed up via **google_oauth**!`

The only thing left for us to do was to somehow extract and filter all of these messages so we could backfill the database.

Awesome thing about discord is that it has a very active community focussed around creating discord bots, so API support for discord is excellent.

Couple of searches later, we found the following repo: <https://github.com/Tyrrrz/DiscordChatExporter> which was just what we needed. Command we used to extract the complete chat history was:

```bash
docker run --rm -it -v ./exported_messages:/out tyrrrz/discordchatexporter:stable export -t AUTH_TOKEN -c CHANNEL_ID -f Json --utc
```

And after extraction we were left with a json formatted file which contained all messages from the channel along with their UTC timestamp. Last step that was left was simply extracting all messages which had the mentioned format and correlating those emails with the emails from the database which had empty `created_at` field.

## The Conclusion

I'll be completely honest, this was one of the coolest hacks I've ever done, and I'm most likely going to tell this story a lot in the following days. Apologies in advance to all my friends.

In hindsight, I'm not really sure what we could've done better in order to avoid this bug, since we thoroughly tested our db logic and the error was not that obvious if you didn't know what you were looking for, and it was easy to miss since the bug was pretty subtle. Sometimes things like this just slip by.

Possible indirect prevention of this bug could've happened if I finally got around to writing real-time tracking of user signups by date, since we would've spotted something weird going on right away.
