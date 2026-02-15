# Telegram Bot API 7.0+ Button Styles - February 2026 Update

## Major Update: Native Button Color Support!

As of **February 9, 2026**, Telegram Bot API 7.0+ introduced the `style` parameter for `InlineKeyboardButton`, enabling native colored button backgrounds without relying on emoji tricks.

## Available Button Styles

### 1. **Positive** (Green Background)
Used for affirmative actions like:
- Start/Play buttons
- Confirm actions
- Withdraw/Cashout
- Place Bet
- Selected items

```python
InlineKeyboardButton("Start Game", callback_data="start", style="positive")
```

### 2. **Destructive** (Red Background)
Used for negative or cancelling actions:
- Cancel/Back buttons
- Delete actions
- Exit/Close

```python
InlineKeyboardButton("Cancel", callback_data="cancel", style="destructive")
```

### 3. **Default** (Standard Gray/Dark)
Used for neutral actions:
- Navigation buttons
- Information buttons
- Standard selections

```python
InlineKeyboardButton("Info", callback_data="info")  # No style parameter = default
```

## Implementation in Our Bot

### Start Menu
```python
keyboard = [
    [
        InlineKeyboardButton("💎 Deposit", callback_data="main_deposit"),  # Default
        InlineKeyboardButton("💸 Withdraw", callback_data="main_withdraw", style="positive")  # Green
    ],
    [
        InlineKeyboardButton("🎮 Games", callback_data="main_games"),  # Default
        InlineKeyboardButton("➕ More", callback_data="main_more")  # Default
    ],
]
```

### Keno Game
```python
# Selected numbers: Green background
InlineKeyboardButton(f"✓ {i}", callback_data=f"keno_pick_{game_id}_{i}", style="positive")

# Unselected numbers: Default background
InlineKeyboardButton(str(i), callback_data=f"keno_pick_{game_id}_{i}")

# Place Bet: Green background
InlineKeyboardButton(f"✅ Place Bet", callback_data=f"keno_place_{game_id}", style="positive")

# Cancel: Red background
InlineKeyboardButton("❌ Cancel", callback_data=f"keno_cancel_{game_id}", style="destructive")
```

### Mines Game
```python
# Cashout: Green background
InlineKeyboardButton(f"💰 Cashout", callback_data=f"mines_cashout_{game_id}", style="positive")

# Grid tiles: Default background (clean)
InlineKeyboardButton("◻️", callback_data=f"mines_pick_{game_id}_{i}")
```

### Tower Game
```python
# Start: Green background
InlineKeyboardButton("▶️ Start Game", callback_data="tower_start_game", style="positive")

# Back: Red background
InlineKeyboardButton("🔙 Back", callback_data="cancel_game", style="destructive")
```

### Roulette Game
```python
# Start: Green background
InlineKeyboardButton("▶️ Start", callback_data=f"roul_start_{user_id}", style="positive")

# Selected numbers: Green background
InlineKeyboardButton(f"✅{emoji} {num}", callback_data=f"roul_num_{num}_{user_id}", style="positive")

# Cancel/Back: Red background
InlineKeyboardButton("❌ Cancel Bet", callback_data=f"roul_cancel_{user_id}", style="destructive")
InlineKeyboardButton("🔙 Back", callback_data=f"roul_back_{user_id}", style="destructive")
```

### Leaderboard
```python
# Back button: Red background
InlineKeyboardButton("🔙 Back to More", callback_data="main_more", style="destructive")
```

## Visual Impact

### Before (Emoji-based Colors)
```
🔵 Deposit    🟢 Withdraw
🔵 Games      🔴 More
```
- Visual distinction through emoji colors only
- Button backgrounds remained gray

### After (Native Button Styles)
```
💎 Deposit    �� Withdraw
[Gray BG]     [GREEN BG]

🎮 Games      ➕ More
[Gray BG]     [Gray BG]
```
- Actual colored button backgrounds
- Green for positive actions
- Red for destructive actions
- Professional appearance

## Benefits

1. **Native Support**: No workarounds needed
2. **Better UX**: Clear visual hierarchy
3. **Professional**: Matches modern app design
4. **Accessibility**: Color + text provides multiple cues
5. **Telegram Standard**: Consistent with other modern bots

## Migration Notes

- Old emoji prefixes (🔵🟢🔴) replaced with cleaner emojis
- `style` parameter added to appropriate buttons
- Backward compatible: Bots without `style` still work
- Requires python-telegram-bot library supporting API 7.0+

## Technical Requirements

### Python-telegram-bot Library
Minimum version required: **v20.8+** (supporting Bot API 7.0)

```bash
pip install --upgrade python-telegram-bot>=20.8
```

### API Version Check
```python
from telegram import __version__ as ptb_version
print(f"python-telegram-bot version: {ptb_version}")
```

## Conclusion

The new `style` parameter is a game-changer for Telegram bot UX. Our implementation leverages:
- ✅ Green buttons for positive actions (start, confirm, withdraw)
- ✅ Red buttons for destructive actions (cancel, back, delete)
- ✅ Default buttons for neutral actions (info, navigation)

This provides a professional, intuitive interface that users will instantly understand.
