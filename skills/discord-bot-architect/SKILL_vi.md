---
name: discord-bot-architect
description: "Kỹ năng chuyên biệt để xây dựng bot Discord sẵn sàng cho sản xuất. Bao gồm Discord.js (JavaScript) và Pycord (Python), gateway intents, lệnh slash, thành phần tương tác, giới hạn tốc độ và sharding."
source: vibeship-spawner-skills (Apache 2.0)
---

# Kiến Trúc Sư Bot Discord (Discord Bot Architect)

## Các Mẫu (Patterns)

### Discord.js v14 Foundation

Thiết lập bot Discord hiện đại với Discord.js v14 và lệnh slash

**Khi nào sử dụng**: ['Xây dựng bot Discord bằng JavaScript/TypeScript', 'Cần kết nối gateway đầy đủ với các sự kiện', 'Xây dựng bot với các tương tác phức tạp']

```javascript
// src/index.js
const { Client, Collection, GatewayIntentBits, Events } = require("discord.js");
const fs = require("node:fs");
const path = require("node:path");
require("dotenv").config();

// Create client with minimal required intents
const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    // Add only what you need:
    // GatewayIntentBits.GuildMessages,
    // GatewayIntentBits.MessageContent,  // PRIVILEGED - avoid if possible
  ],
});

// Load commands
client.commands = new Collection();
const commandsPath = path.join(__dirname, "commands");
const commandFiles = fs
  .readdirSync(commandsPath)
  .filter((f) => f.endsWith(".js"));

for (const file of commandFiles) {
  const filePath = path.join(commandsPath, file);
  const command = require(filePath);
  if ("data" in command && "execute" in command) {
    client.commands.set(command.data.name, command);
  }
}

// Load events
const eventsPath = path.join(__dirname, "events");
const eventFiles = fs.readdirSync(eventsPath).filter((f) => f.endsWith(".js"));

for (const file of eventFiles) {
  const filePath = path.join(eventsPath, file);
  const event = require(filePath);
  if (event.once) {
    client.once(event.name, (...args) => event.execute(...args));
  } else {
    client.on(event.name, (...args) => event.execute(...args));
  }
}

client.login(process.env.DISCORD_TOKEN);
```

```javascript
// src/commands/ping.js
const { SlashCommandBuilder } = require("discord.js");

module.exports = {
  data: new SlashCommandBuilder()
    .setName("ping")
    .setDescription("Replies with Pong!"),

  async execute(interaction) {
    const sent = await interaction.reply({
      content: "Pinging...",
      fetchReply: true,
    });

    const latency = sent.createdTimestamp - interaction.createdTimestamp;
    await interaction.editReply(`Pong! Latency: ${latency}ms`);
  },
};
```

### Pycord Bot Foundation

Bot Discord với Pycord (Python) và lệnh ứng dụng

**Khi nào sử dụng**: ['Xây dựng bot Discord bằng Python', 'Thích các mẫu async/await', 'Cần hỗ trợ lệnh slash tốt']

```python
# main.py
import os
import discord
from discord.ext import commands
from dotenv import load_dotenv

load_dotenv()

# Configure intents - only enable what you need
intents = discord.Intents.default()
# intents.message_content = True  # PRIVILEGED - avoid if possible
# intents.members = True          # PRIVILEGED

bot = commands.Bot(
    command_prefix="!",  # Legacy, prefer slash commands
    intents=intents
)

@bot.event
async def on_ready():
    print(f"Logged in as {bot.user}")
    # Sync commands (do this carefully - see sharp edges)
    # await bot.sync_commands()

# Slash command
@bot.slash_command(name="ping", description="Check bot latency")
async def ping(ctx: discord.ApplicationContext):
    latency = round(bot.latency * 1000)
    await ctx.respond(f"Pong! Latency: {latency}ms")

# Slash command with options
@bot.slash_command(name="greet", description="Greet a user")
async def greet(
    ctx: discord.ApplicationContext,
    user: discord.Option(discord.Member, "User to greet"),
    message: discord.Option(str, "Custom message", required=False)
):
    msg = message or "Hello!"
    await ctx.respond(f"{user.mention}, {msg}")

# Load cogs
for filename in os.listdir("./cogs"):
    if filename.endswith(".py"):
        bot.load_extension(f"cogs.{filename[:-3]}")

bot.run(os.environ["DISCORD_TOKEN"])
```

```python
# cogs/general.py
import discord
from discord.ext import commands

class General(commands.Cog):
    def __init__(self, bot):
        self.bot = bot

    @commands.slash_command(name="info", description="Bot information")
    async def info(self, ctx: discord.ApplicationContext):
        embed = discord.Embed(
            title="Bot Info",
            description="A helpful Discord bot",
            color=discord.Color.blue()
        )
        embed.add_field(name="Servers", value=len(self.bot.guilds))
        embed.add_field(name="Latency", value=f"{round(self.bot.latency * 1000)}ms")
        await ctx.respond(embed=embed)
```

### Mẫu Thành phần Tương tác

Sử dụng các nút, menu chọn và modal cho UX phong phú

**Khi nào sử dụng**: ['Cần giao diện người dùng tương tác', 'Thu thập đầu vào người dùng ngoài các tùy chọn lệnh slash', 'Xây dựng menu, xác nhận hoặc biểu mẫu']

```javascript
// Discord.js - Buttons and Select Menus
const {
  SlashCommandBuilder,
  ActionRowBuilder,
  ButtonBuilder,
  ButtonStyle,
  StringSelectMenuBuilder,
  ModalBuilder,
  TextInputBuilder,
  TextInputStyle
} = require('discord.js');

module.exports = {
  data: new SlashCommandBuilder()
    .setName('menu')
    .setDescription('Shows an interactive menu'),

  async execute(interaction) {
    // Button row
    const buttonRow = new ActionRowBuilder()
      .addComponents(
        new ButtonBuilder()
          .setCustomId('confirm')
          .setLabel('Confirm')
          .setStyle(ButtonStyle.Primary),
        new ButtonBuilder()
          .setCustomId('cancel')
          .setLabel('Cancel')
          .setStyle(ButtonStyle.Danger),
        new ButtonBuilder()
          .setLabel('Documentation')
          .setURL('https://discord.js.org')
          .setStyle(ButtonStyle.Link)  // Link buttons don't emit events
      );

    // Select menu row (one per row, takes all 5 slots)
    const selectRow = new ActionRowBuilder()
      .addComponents(
        new StringSelectMenuBuilder()
          .setCustomId('select-role')
          .setPlaceholder('Select a role')
          .setMinValues(1)
          .setMaxValues(3)
          .addOptions([
            { label: 'Developer', value: 'dev', emoji: '💻' },
            { label: 'Designer', value: 'design', emoji: '🎨' },
            { label: 'Community', value: 'community', emoji: '🎉' }
          ])
      );

    await interaction.reply({
      content: 'Choose an option:',
      components: [buttonRow, selectRow]
    });

    // Collect responses
    const collector = interaction.channel.createMessageComponentCollector({
      filter: i => i.user.id === interaction.user.id,
      time: 60_000  // 60 seconds timeout
    });

    collector.on('collect', async i => {
      if (i.customId === 'confirm') {
        await i.update({ content: 'Confirmed!', components: [] });
        collector.stop();
      } else if (i.custo
```

## Các Mẫu Chống (Anti-Patterns)

### ❌ Nội dung Tin nhắn cho Lệnh

**Tại sao xấu**: Message Content Intent là quyền hạn đặc biệt và không được khuyến khích cho các lệnh bot.
Lệnh slash là cách tiếp cận được dự định.

### ❌ Đồng bộ Lệnh mỗi lần Khởi động

**Tại sao xấu**: Đăng ký lệnh bị giới hạn tốc độ. Các lệnh toàn cầu mất tới 1 giờ để lan truyền. Đồng bộ mỗi lần khởi động lãng phí các cuộc gọi API và có thể chạm giới hạn.

### ❌ Chặn Vòng lặp Sự kiện

**Tại sao xấu**: Discord gateway yêu cầu nhịp tim (heartbeats) thường xuyên. Các hoạt động chặn gây ra nhịp tim bị bỏ lỡ và ngắt kết nối.

## ⚠️ Các Cạnh Sắc (Sharp Edges)

| Vấn đề | Mức độ nghiêm trọng | Giải pháp                                                       |
| ------ | ------------------- | --------------------------------------------------------------- |
| Vấn đề | nghiêm trọng        | ## Xác nhận ngay lập tức, xử lý sau                             |
| Vấn đề | nghiêm trọng        | ## Bước 1: Bật trong Cổng thông tin Nhà phát triển              |
| Vấn đề | cao                 | ## Sử dụng tập lệnh triển khai riêng (không phải khi khởi động) |
| Vấn đề | nghiêm trọng        | ## Không bao giờ hardcode token                                 |
| Vấn đề | cao                 | ## Tạo URL mời chính xác                                        |
| Vấn đề | trung bình          | ## Phát triển: Sử dụng lệnh guild                               |
| Vấn đề | trung bình          | ## Không bao giờ chặn vòng lặp sự kiện                          |
| Vấn đề | trung bình          | ## Hiển thị modal ngay lập tức                                  |
