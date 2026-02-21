---
title: Navibot Plugin Development Guide
description: A complete guide to authoring plugins using Navibot.
category: navibot
order: 0
lastModified: 2026-02-21
version: "0.6"
image: ""
imageAlt: ""
hideCoverImage: false
hideTOC: false
draft: false
noIndex: false
featured: true
---
# 🤖 Navi Engine Comprehensive API Reference



Welcome to the complete API reference for the **Navi Engine**. This document covers every function, event, and data structure exposed from the Rust core to the Lua plugin system.

## 🌍 Global Engine API
The Rust core exposes a global `navi` object containing all engine functions.

### System & Logging
* `navi.log(msg: String)`: Sends a log message directly to the Terminal UI.
* `print(...)`: The standard Lua print function is overridden to automatically call `navi.log` and prepend the origin file name.

### Database (`navi.db`)
Navi uses a thread-safe SQLite key-value store (`kv_store` table).
* `navi.db.set(key: String, value: String)`: Inserts or replaces a string in the database.
* `navi.db.get(key: String) -> String|nil`: Retrieves a string from the database.

### Discord Presence
* `navi.set_status(activity_type: String, text: String)`: Updates the bot's presence.
* Supported activity types: `"playing"`, `"listening"`, `"watching"`, `"competing"`, `"custom"`, or `"none"` (to clear the status).

---

## 💬 Channels & Messaging

### Channel Management
* `navi.create_channel(guild_id: String, name: String, options: Table)`: Creates a new text channel dynamically.
* `category_id` (String): Spawns the channel inside a specific category.
* `user_id` (String): Grants private `VIEW_CHANNEL` and `SEND_MESSAGES` permissions to a specific user, while denying `@everyone`.
* `role_id` (String): Grants private permissions to a specific role.
* `welcome_message` (String): Automatically sends this text upon creation.
* `close_button` (Boolean): If true, attaches a red "🔒 Close Ticket" button to the welcome message.
* `navi.delete_channel(channel_id: String)`: Instantly deletes the specified channel.

### Sending Messages
* `navi.say(channel_id: number, text: String)`: A lightweight function to send plain text.
* `navi.react(channel_id: String, message_id: String, emoji: String)`: Adds a unicode or custom emoji reaction to a message.
* `navi.send_message(channel_id: String, data: Table)`: Sends text, rich embeds, and UI components.
* `title`, `description`, `color` (number): Standard embed properties.
* `fields`: A list of tables containing `name`, `value`, and `inline` (Boolean).
* `components`: A list of UI components (detailed below).



---

## 🖱️ UI Components
Components are added to messages via the `components` array in `navi.send_message`.

### Buttons
A component table with `type = "button"`.
* `id` (String): The custom ID triggered when clicked.
* `label` (String): The button text.
* `style` (String): `"primary"`, `"secondary"`/`"gray"`, `"success"`/`"green"`, `"danger"`/`"red"`, or `"link"`/`"url"`.
* `url` (String): Required only if style is link/url.

### Select Menus (Dropdowns)
A component table with `type = "select"`.
* `id` (String): The custom ID.
* `placeholder` (String): Placeholder text.
* `options`: An array of tables containing `label`, `value`, `description` (optional), and `emoji` (optional).

---

## ⚙️ Configuration & Commands

### TUI Configuration (`navi.register_config`)
Registers settings to appear in the Terminal UI dashboard.
* `navi.register_config(plugin_name: String, schema: Table)`: Initializes configuration fields.
* Schema fields include: `key`, `name`, `description`, `type`, and `default`.
* Supported TUI Types: `"string"`, `"number"`, `"boolean"`, `"channel"`, `"role"`, `"category"`.
* Settings are saved and retrieved natively via SQLite as `config:plugin_name:key`.

### Slash Commands (`navi.create_slash`)
* `navi.create_slash(name: String, desc: String, options: Table, callback: Function)`: Registers a Discord slash command.
* Command options support types: `"string"`, `"integer"`, `"boolean"`, `"user"`, `"channel"`, `"role"`, and `"number"`.

---

## 👤 Role Management
* `navi.add_role(guild_id: String, user_id: String, role_id: String)`: Assigns a role to a user.
* `navi.remove_role(guild_id: String, user_id: String, role_id: String)`: Removes a role from a user.

---

## 📡 Event Listeners (Triggers)
These functions must be defined globally in your Lua plugins to catch Discord events sent by the Rust engine.



### `on_message(msg_table)`
Fires when a user sends a message (bot messages are ignored).
* `content` (String), `channel_id` (number), `message_id` (number), `author` (String), `author_id` (number), `author_avatar` (String).
* `mentions`: Array of tables containing `name`, `id`, and `avatar`.
* `attachments`: Array of attachment URLs.

### `on_component(ctx)`
Fires when a user interacts with a button or select menu.
* `custom_id` (String), `user_id` (String), `username` (String), `channel_id` (String), `guild_id` (String).
* `values`: Array of selected values (only populated for select menus).
* `ctx.reply(msg: String, ephemeral: Boolean)`: An async function to respond directly to the interaction.

### Slash Command Context
Passed directly to the callback function of `navi.create_slash`.
* `user_id` (String), `username` (String), `channel_id` (String), `guild_id` (String).
* `args`: A table of parsed arguments mapping directly to their respective types (String, Integer, Boolean, Number).
* `ctx.reply(msg: String)`: An async function to send an interaction response.

### `on_member_join(user_id, username)`
Fires when a user joins the server.
* `user_id` (String): The Discord ID of the new member.
* `username` (String): The display name of the new member.
* The engine logs any crashes during this event directly to the TUI.

### `on_reaction_add(ctx)` & `on_reaction_remove(ctx)`
Fires when reactions are added or removed.
* `user_id` (String), `channel_id` (String), `message_id` (String), `guild_id` (String), `emoji` (String).