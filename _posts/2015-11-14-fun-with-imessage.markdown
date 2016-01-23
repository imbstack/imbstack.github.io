---
layout: post
title: Fun with iMessage
---
A friend of mine in a group chat was wondering how long people would go between talking or who talked the most often. I became curious and after some quick [stackoverflowing](http://apple.stackexchange.com/a/80025) I found that you already have all of the data you need locally if you use Messages on <span class="nowrap">OS X</span>. As of today on <span class="nowrap">El Capitan</span>, you can easily hop into the database with <span class="code">sqlite3 ~/Library/Messages/chat.db</span>.

<div class="note">
Note: This only goes back as far as you used Messages on OS X. To get all of your messages from your phone, back it up to your computer and find the sqlite database in <span class="code">~/Library/Application\ Support/MobileSync/Backup/</span> <a href="http://www.wired.com/2013/11/backup-sms-iphone/">(Thanks, Wired!)</a>. You'll find that there are just a mess of files with no names to help you; picking a string from your OS X copy of the Messages db and grepping for it in the backup seems to work.
</div>

The only tables I ended up caring about to answer this question were <span class="code">message</span> and <span class="code">handle</span>. I'll talk a bit more about how to generate this in a moment, but for now, we can look at [Figure 1](#imessage-fig1) and see what the final product is.

<div id="imessage-fig1" class="figure">
    <div></div>
    <span class="caption pure-hidden-xs">Figure 1. Messages per day by participant.</span>
    <span class="caption pure-visible-xs">Figure 1</span>
</div>

From [Figure 1](#imessage-fig1) we can extrapolate into the future and see that Pat will need to spend every single waking second sending texts to this group and I'm probably dead, can someone check on me?

<div class="note">
    Note: It's <em>always</em> ok to extrapolate from two data points.
</div>

There aren't that many steps to getting that plot put together. The first step will be to find which "room" your chats have been taking place in. As far as I can tell, each message is connected to a room via the <span class="code">cache\_roomnames</span> column. Your chat's roomname will be something along the lines of <span class="code">chatNNNNNNNNNNNNNNNNNN</span>, where the <span class="code">N</span>'s are obviously numbers. I found the room I wanted by getting the last messages I received and then matching up a message that was from the room to its roomname.

{% highlight sql linenos %}
SELECT cache_roomnames, text
FROM message
WHERE cache_roomnames != ''
ORDER BY date DESC LIMIT 10;
{% endhighlight %}

Once you have your desired room, you can get every message in it that your heart desires. I found that pulling out just a few key columns got me what I wanted, and joining the <span class="code">handle\_id</span> column to the <span class="code">handle</span> table gets you a phone number for each participant.

{% highlight sql linenos %}
SELECT M.handle_id,
       H.id,
       strftime('%s', M.date + strftime('%s', '2001-01-01 00:00:00'), 'unixepoch', 'localtime'),
       M.text
FROM message AS M
LEFT OUTER JOIN handle AS H ON M.handle_id = H.rowid
WHERE M.cache_roomnames = 'chatNNNNNNNNNNNNNNNNNN'
ORDER BY date;
{% endhighlight %}

That '2001-01-01 00:00:00' is necessary because iMessage starts counting time from 2001 for some reason I'll never understand. What you want to do at this point is up to you, but I output it as a csv and go to work from there in Python. One note to keep in mind, messages will occasionally have newlines in them, so you may want to define a -newline of your own in your csv output. This will make it harder to read normally, but fixes that issue. I'm almost certain there's a better way of solving it, and if you know one, let me know!

From there it's just some simple scripting to make a histogram of posts per user and plot it however you like. I have a manual phone-number <i class="fa fa-long-arrow-right"></i> human-name dictionary in my code. I'm not sure if there's a way to automate this with wherever Contacts stores its data, but perhaps that's a post for another day.

This got me curious, what does my history look like across _everyone_ I've talked to? Well, here you go:

<div id="imessage-fig2" class="figure">
    <div></div>
    <span class="caption pure-hidden-xs">Figure 2. Messages per week per person.</span>
    <span class="caption pure-visible-xs">Figure 2</span>
</div>

In [Figure 2](#imessage-fig2) I plot the messages per week from anyone who sent me over 150 messages in a week at least once since I've had the phone (arbitrary threshold to keep plot clean). With the exception of Person E and Person F, the top texter during any period of time was the person I was dating at that time. I've not revealed identities here to protect the innocent.

Over the course of a normal week, when are people texting me? We can pretty much steal Github's punchcard visualization and reproduce it [Figure 3](#imessage-fig3) with texts instead of commits.

<div id="imessage-fig3" class="figure">
    <div></div>
    <span class="caption pure-hidden-xs">Figure 3. Times when texts are happening.</span>
    <span class="caption pure-visible-xs">Figure 3</span>
</div>

There's still more to find with all of this, check back soon for more!

<script type="text/javascript" src="{{ "/js/imessage.js" | prepend: site.baseurl }}"></script>
<link rel="stylesheet" href="{{ "/css/imessage.css" | prepend: site.baseurl }}">