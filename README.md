# Speaking-Bot-Coding
For discord
import discord
from discord.ext import commands
import json
import os

# Replace 'YOUR_BOT_TOKEN' with your actual bot token
TOKEN = 'YOUR_BOT_TOKEN'

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix='!', intents=intents)

# Simple JSON database for demonstration
DATABASE_FILE = 'database.json'

def load_database():
    try:
        with open(DATABASE_FILE, 'r') as f:
            return json.load(f)
    except FileNotFoundError:
        return {}

def save_database(data):
    with open(DATABASE_FILE, 'w') as f:
        json.dump(data, f, indent=4)

@bot.event
async def on_ready():
    print(f'Logged in as {bot.user.name}')
    try:
        synced = await bot.tree.sync()
        print(f"Synced {len(synced)} commands")
    except Exception as e:
        print(e)

@bot.command(name='ping')
async def ping(ctx):
    await ctx.send(f'Pong! Latency: {round(bot.latency * 1000)}ms')

@bot.slash_command(name="hello", description="Says hello")
async def hello(interaction: discord.Interaction):
    await interaction.response.send_message(f"Hey {interaction.user.mention}!")

@bot.command(name='count')
async def count(ctx):
    data = load_database()
    user_id = str(ctx.author.id)
    if user_id in data:
        data[user_id] += 1
    else:
        data[user_id] = 1
    save_database(data)
    await ctx.send(f'You have used this command {data[user_id]} times.')

class MyView(discord.ui.View):
    @discord.ui.button(label="Click me!", style=discord.ButtonStyle.green)
    async def my_button(self, interaction: discord.Interaction, button: discord.ui.Button):
        await interaction.response.send_message("Button clicked!", ephemeral=True)

@bot.command(name="button")
async def button(ctx):
    await ctx.send("Click the button!", view=MyView())

@bot.event
async def on_message(message):
    if message.author == bot.user:
        return

    if "hello bot" in message.content.lower():
        await message.channel.send("Hello there!")

    await bot.process_commands(message)

#Error Handling Example
@bot.event
async def on_command_error(ctx, error):
    if isinstance(error, commands.CommandNotFound):
        await ctx.send("That command does not exist.")
    else:
        print(f"An error occurred: {error}")
        await ctx.send("An unexpected error occurred.")

bot.run(TOKEN)
