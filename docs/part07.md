# Part 7 - Welcome

Ok, so far we've learned how to make a bot, add commands, use cogs, and change the status. Let's combine some of this and a bit of new stuff to make something that welcomes users with a custom message when they join a server.

To start out, we'll once again use the same code from the previous part and add to that. First, we have a modification we need to make to the bot&#46;py file. Up until this point we've been using the default gateway intents that discord&#46;py sends to the gateway - which is all except the privileged intents (guild members and guild presences).

Before we can proceed to modify the code, we need to enable these privileged intents on the Discord developer portal. Go to your bot's bot page on the portal, and scroll down to the privileged intents section:

![Privileged Intents](https://github.com/vcokltfre/bot-tutorial/raw/master/images/privileged_intents.png "Privileged Intents")

Now, you need to enable both of these, and click save to update the application.

![Privileged Intents](https://github.com/vcokltfre/bot-tutorial/raw/master/images/privileged_intents_enabled.png "Privileged Intents")

With that done, let's get back to the code - this will be the last time we have to visit the developer portal for a while. First, we need to tell discord&#46;py to use these intents when connecting to the gateway, which we'll do by setting intents in the bot's constructor to discord.Intents.all(), after which the bot file will look like this:

```py
import discord
from discord.ext import commands

bot = commands.Bot(command_prefix="!", intents=discord.Intents.all())

bot.load_extension("somecommands")

bot.run("your_token_here")
```

That's all that needed to enable intents, and now we can harness the full power of Discord gateway events! Now, we have to create the code that welcomes people, which we will stick in the somecommands&#46;py for now, but you're free to make another file and another cog, and load it in the same way as before.

Again, this will be within the class, just under the commands we already have - but wait, this isn't a command, its an event. Luckily discord&#46;py has us covered for events too, and provides listeners so that we can use them.

As with previous examples I'll show the code we need to use and then explain what the new bits do afterwards:

```py
    @commands.Cog.listener()
    async def on_member_join(self, member: discord.Member):
        channel = self.bot.get_channel(799309066202775624)
        await channel.send(f"Welcome, {member}!")
```

Ok, quite a lot to cover with this one. First, we have a new decorator, commands.Cog.listener(). This is like commands.command(), only it nows receives different gateway events rather than messages, in this case the member join event (a full list of events can be found [here](https://discordpy.readthedocs.io/en/latest/api.html#event-reference) in the documentation) which gives us details about a member joining a guild.

Next, we get a channel from the bot, this is the channel that we'll send the welcome message in. To do this we use self.bot.get_channel() to get a channel from the bot's cache of channels. Note that I explicitly use the word get here, not fetch. This is because in discord&#46;py, and now throughout this tutorial, get will refer to fetching something from the local cache, and fetch will mean fetching something from the API. It's important to distinguish which is which, because the get methods are synchronous, while fetch methods are async and must be awaited.

Now, you'll likely have noticed the long number in there, that's the channel's unique [snowflake ID.](https://discord.com/developers/docs/reference#snowflakes) All objects in Discord, be it a role, channel, guild, user, etc. have an ID (excluding things like permission overwrites which just map user and role IDs to a set of permissions) which can be found in the client by right clicking the object and clicking copy ID. To have this option you need to have [developer mode enabled.](https://support.discord.com/hc/en-us/articles/206346498-Where-can-I-find-my-User-Server-Message-ID-) Make sure to replace my channel ID with one of your own, or the message won't be sent!

Finally, we send a message to the channel which welcomes the user. In f-string expressions or when converting discord&#46;py objects to strings in general, perhaps using str(), if the object has a name attribute that's what it will convert to. For members and users however it's slightly more, and it will convert to the username and discriminator, which is the 4 numbers after the #, so let's take my user object for example, if we do str(vcokltfre_user) we'll get 'vcokltfre#6868' - which in a welcome message is preferable to pinging people when they join, which can be quite annoying. It's also recommended that you dont DM people welcome messages either.

That's it for this part - your bot should now have a fully functioning welcome message when new people join the server. If it doesn't work be sure to check that you have used the correct channel ID for the channel you want to send it in, and that the ID is a number, not a string. You can now move on to [Part 8 - A Better Ping Command](./part08.md), or you can take a [look at the full code for this part.](https://github.com/vcokltfre/bot-tutorial/tree/master/code/part7)

##### [Back to the main page](../README.md)