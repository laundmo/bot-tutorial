# Cooldowns

You may often find a need when making bots to give commands their own cooldowns, to ensure that they're not called too often. There are a variety of reasons for this, such as mitigating spam, or fairly commonly, making sure people don't send too many requests if the command is an API intensive one.

Fortunately for us, discord&#46;py provides built in cooldown decorators that we can use on our existing commands that do just this. This will be a fairly small part, since there is not that much to cover on the topic, however I hope it will be of great use for ratelimiting your commands.

To start with, let's recap one of the commands we have from earlier, the ping command:

```py
    @commands.command(name="ping")
    async def ping(self, ctx: commands.Context):
        """Get the bot's current websocket latency"""
        await ctx.send(f"Pong! {round(self.bot.latency * 1000)}ms")
```

Now, we don't want someone seeing this in chat and deciding that they want to keep testing it over and over, that would be spammy and unwanted, and besides, the gateway latency only updates every 40 seconds or so. So, we put a cooldown on the command. Cooldowns use decorators, just like how you defined that it was a command, as a few other things do also, which I'll show later in the tutorial. If we add a cooldown the command will end up looking like this:

```py
    @commands.command(name="ping")
    @commands.cooldown(1, 5)
    async def ping(self, ctx: commands.Context):
        """Get the bot's current websocket latency"""
        await ctx.send(f"Pong! {round(self.bot.latency * 1000)}ms")
```

As you can see, we added the line '@commands.cooldown(1, 5)', but on it's own this probably doesn't do quite what we want. By default the cooldown decorator uses commands.BucketType.default as it's bucket type - which is a global bucket. This means if I use !ping in my server, and Dave uses !ping in his server 2 seconds later, the command won't work for him, because the cooldown is global. To fix this we need to use a different bucket type, I'm going to opt for the channel bucket type, which as you'd expect uses a channel based bucket, but you can use [any of these bucket types.](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.discord.ext.commands.BucketType)

```py
    @commands.command(name="ping")
    @commands.cooldown(1, 5, commands.BucketType.channel)
    async def ping(self, ctx: commands.Context):
        """Get the bot's current websocket latency"""
        await ctx.send(f"Pong! {round(self.bot.latency * 1000)}ms")
```

That's better, now we've limited that command to be run once every 5 seconds, per channel, which is much more useful than a global ratelimit here. But now, we have a different issue. Now, if we run the command in rapid succession nothing happens - not even showing an error to the user. This can leave people fairly confused, "It worked for vco, why didn't it work for me?" which is why we now need to add error handling for it. Error handling will also be covered in far greater detail later in the tutorial, but for the purposes of this part, I'll add a simple one for the command. Adding an error handler for a command looks something like the following:

```py
    @ping.error
    async def ping_error(self, ctx: commands.Context, error):
        if isinstance(error, commands.CommandOnCooldown):
            await ctx.send(f"This command is on cooldown, try again after {round(error.retry_after)} seconds.", delete_after=5)
        print(error)
```

I won't go over the error handling in detail here, but there is one new thing here I should mention. If you look in the ctx.send() function, you'll see a delete_after parameter, this tells discord.py to delete the message after the interval provided, to the error message gets deleted after a few seconds and doesn't clog up chat.

That's it for cooldowns, there isn't much more to know (though I suggest you check the [docs page for them](https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#discord.ext.commands.cooldown)), you're now ready to move onto the next part, [Part 12 - Command Error Handling](./part12.md), or you can take a [look at the full code for this part.](../code/part11/somecommands.py)