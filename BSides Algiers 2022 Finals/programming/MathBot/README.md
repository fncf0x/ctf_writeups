# MathBot (programming 500)

### overview
We were given a Discord bot that sends you a math challenge to solve 50 times with a time delay.

### solution

```python
import discord
import asyncio
import time


client = discord.Client()

@client.event
async def on_ready():
    print(f'Logged in as {client.user.name} {client.user.id}')
    ch_id = 133333333333333333337 # a channel ID where both the bot and you are joined
    channel = client.get_channel(ch_id)
    await channel.send('$start')
    time.sleep(3)
    while True:
        last_msg = (await client.get_channel(ch_id).history(limit=1).flatten())[0]
        print(last_msg)
        solution = eval(last_msg.content)
        await channel.send(solution)
        time.sleep(0.5)

client.run('DISCORD_USER_TOKEN', bot=False)

```


~ **J4ck** alias **fncf0x**

