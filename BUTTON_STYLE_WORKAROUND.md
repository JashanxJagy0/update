# Telegram Button Style Implementation - Workaround Guide

## The Situation

**Telegram Bot API 7.0+** (released February 9, 2026) added support for the `style` parameter in `InlineKeyboardButton`, enabling colored button backgrounds:
- `style="positive"` → Green background
- `style="destructive"` → Red background  
- `style="primary"` → Blue background

**However**, the **python-telegram-bot** library doesn't have native support for this parameter yet (as of implementation date).

## The Workaround

We use a manual workaround to inject the `style` parameter into button dictionaries before sending them to Telegram's API.

### Helper Functions

```python
def apply_button_style(button, style):
    """
    Apply style to a button by converting to dict and injecting style parameter.
    
    Args:
        button: InlineKeyboardButton object
        style: One of 'positive' (green), 'destructive' (red), 'primary' (blue)
    
    Returns:
        Dict with style parameter injected
    """
    if button is None:
        return None
    btn_dict = button.to_dict()
    btn_dict['style'] = style
    return btn_dict

def create_styled_keyboard(keyboard_array):
    """
    Convert a keyboard array with styled buttons to the proper format.
    
    Args:
        keyboard_array: 2D array of buttons (mix of dicts and InlineKeyboardButton objects)
    
    Returns:
        Dict in format expected by Telegram API: {'inline_keyboard': [[...]]}
    """
    styled_rows = []
    for row in keyboard_array:
        styled_row = []
        for item in row:
            if isinstance(item, dict):
                # Already a dict (styled button)
                styled_row.append(item)
            elif hasattr(item, 'to_dict'):
                # InlineKeyboardButton object, convert to dict
                styled_row.append(item.to_dict())
            else:
                # Shouldn't happen, but handle gracefully
                styled_row.append(item)
        styled_rows.append(styled_row)
    return {'inline_keyboard': styled_rows}
```

### Usage Example

#### Before (Without Styles)
```python
keyboard = [
    [
        InlineKeyboardButton("Deposit", callback_data="deposit"),
        InlineKeyboardButton("Withdraw", callback_data="withdraw")
    ]
]

await message.reply_text("Menu", reply_markup=InlineKeyboardMarkup(keyboard))
```

#### After (With Styled Buttons)
```python
keyboard = [
    [
        apply_button_style(InlineKeyboardButton("Deposit", callback_data="deposit"), "primary"),
        apply_button_style(InlineKeyboardButton("Withdraw", callback_data="withdraw"), "positive")
    ]
]

styled_keyboard = create_styled_keyboard(keyboard)
await message.reply_text("Menu", reply_markup=styled_keyboard)
```

### What Gets Sent to Telegram

```json
{
    "inline_keyboard": [
        [
            {
                "text": "Deposit",
                "callback_data": "deposit",
                "style": "primary"
            },
            {
                "text": "Withdraw",
                "callback_data": "withdraw",
                "style": "positive"
            }
        ]
    ]
}
```

## Implementation in Our Bot

### 1. Start Menu (`/start`)

```python
keyboard = [
    [
        apply_button_style(InlineKeyboardButton("💎 Deposit", callback_data="main_deposit"), "primary"),
        apply_button_style(InlineKeyboardButton("💸 Withdraw", callback_data="main_withdraw"), "positive")
    ],
    [
        apply_button_style(InlineKeyboardButton("🎮 Games", callback_data="main_games"), "primary"),
        InlineKeyboardButton("➕ More", callback_data="main_more")
    ]
]

styled_keyboard = create_styled_keyboard(keyboard)
await message.reply_photo(photo=image, caption=text, reply_markup=styled_keyboard)
```

**Result**:
- Deposit: Blue background
- Withdraw: Green background
- Games: Blue background
- More: Default gray

### 2. Keno Game

```python
def create_keno_keyboard(game_id, selected_numbers):
    buttons = []
    for i in range(1, 41):
        btn = InlineKeyboardButton(str(i), callback_data=f"keno_pick_{game_id}_{i}")
        if i in selected_numbers:
            buttons.append(apply_button_style(btn, "positive"))  # Green
        else:
            buttons.append(apply_button_style(btn, "primary"))   # Blue
    
    keyboard = [buttons[i:i+5] for i in range(0, 40, 5)]
    
    # Action buttons
    action_row2 = [
        InlineKeyboardButton("📊 Payout Table", ...),
        apply_button_style(InlineKeyboardButton("❌ Cancel", ...), "destructive")  # Red
    ]
    
    if selected_numbers:
        place_bet = apply_button_style(
            InlineKeyboardButton(f"✅ Place Bet ({len(selected_numbers)} numbers)", ...),
            "positive"  # Green
        )
        keyboard.append([place_bet])
    
    return create_styled_keyboard(keyboard)
```

**Result**:
- Unselected numbers: Blue background
- Selected numbers: Green background
- Place Bet: Green background
- Cancel: Red background

### 3. Mines Game

```python
def mines_keyboard(game_id, reveal=False):
    # ... tile generation ...
    
    for i in range(1, total_cells + 1):
        btn = InlineKeyboardButton(emoji, callback_data=f"mines_pick_{game_id}_{i}_{user_id}")
        if emoji == "🟦":  # Unselected tile
            buttons.append(apply_button_style(btn, "primary"))  # Blue
        else:
            buttons.append(btn)
    
    # Action buttons
    cashout_btn = apply_button_style(
        InlineKeyboardButton(f"💰 Cashout (${winnings:.2f})", ...),
        "positive"  # Green
    )
    random_btn = apply_button_style(
        InlineKeyboardButton("🎲 Random", ...),
        "primary"  # Blue
    )
    
    return create_styled_keyboard(keyboard)
```

**Result**:
- Unselected tiles (🟦): Blue background
- Selected tiles (✅): Default
- Cashout: Green background
- Random: Blue background

### 4. Tower Game

```python
keyboard = [
    [apply_button_style(InlineKeyboardButton("▶️ Start Game", ...), "positive")],  # Green
    [
        InlineKeyboardButton("◀️", ...),
        apply_button_style(InlineKeyboardButton(f"⚙️ {diff_name}", ...), "primary"),  # Blue
        InlineKeyboardButton("▶️", ...)
    ],
    [apply_button_style(InlineKeyboardButton("🔙 Back", ...), "destructive")]  # Red
]

styled_keyboard = create_styled_keyboard(keyboard)
```

**Result**:
- Start Game: Green background
- Difficulty: Blue background
- Back: Red background

### 5. Roulette Game

```python
def create_roulette_menu_keyboard(user_id, bet_amount):
    keyboard = [
        [apply_button_style(InlineKeyboardButton("▶️ Start", ...), "positive")],  # Green
        # ... betting options ...
        [apply_button_style(InlineKeyboardButton("❌ Cancel Bet", ...), "destructive")]  # Red
    ]
    return create_styled_keyboard(keyboard)

def create_roulette_number_selection_keyboard(user_id, selected_numbers):
    keyboard = [
        [apply_button_style(InlineKeyboardButton("▶️ Start", ...), "positive")]  # Green
    ]
    
    for num in range(0, 37):
        btn = InlineKeyboardButton(f"{emoji}  {num}  ", ...)
        if num in selected_numbers:
            row.append(apply_button_style(btn, "positive"))  # Green when selected
        else:
            row.append(btn)  # Default when unselected
    
    keyboard.append([apply_button_style(InlineKeyboardButton("🔙 Back", ...), "destructive")])  # Red
    return create_styled_keyboard(keyboard)
```

**Result**:
- Start: Green background
- Selected numbers: Green background
- Unselected numbers: Default gray
- Cancel/Back: Red background

### 6. Leaderboard

```python
keyboard = [
    # ... navigation buttons ...
    [apply_button_style(InlineKeyboardButton("🔙 Back to More", ...), "destructive")]  # Red
]

reply_markup = create_styled_keyboard(keyboard)
```

**Result**:
- Back button: Red background

## Button Style Guidelines

### Positive (Green) - `style="positive"`
Use for affirmative, success, or positive actions:
- ✅ Start/Play buttons
- ✅ Confirm actions
- ✅ Withdraw/Cashout
- ✅ Place Bet
- ✅ Selected items

### Destructive (Red) - `style="destructive"`
Use for negative, cancelling, or attention-requiring actions:
- ❌ Cancel/Exit buttons
- ❌ Back buttons
- ❌ Delete actions
- ❌ Stop/End

### Primary (Blue) - `style="primary"`
Use for informational or navigational actions:
- ℹ️ Info/Details buttons
- 🔵 Navigation options
- 🔵 Unselected items
- 🔵 General actions

### Default (Gray) - No style parameter
Use for neutral actions that don't need emphasis:
- ⚙️ Settings
- 📊 Stats
- 🔗 Links

## Important Notes

### Mixing Styled and Non-Styled Buttons

You can mix styled and non-styled buttons in the same keyboard:

```python
keyboard = [
    [
        apply_button_style(InlineKeyboardButton("Styled", ...), "primary"),
        InlineKeyboardButton("Not Styled", ...)  # Will be default gray
    ]
]
```

### With `safe_edit_message`

The `safe_edit_message` function accepts styled keyboards directly:

```python
styled_keyboard = create_styled_keyboard(keyboard)
await safe_edit_message(query, text, reply_markup=styled_keyboard)
```

### Compatibility

- ✅ Works with all Telegram clients supporting Bot API 7.0+
- ✅ Backward compatible: Older clients ignore the `style` parameter
- ✅ Works in both DMs and groups
- ✅ Compatible with all button types (callback_data, url, etc.)

## Testing

To test if buttons are working:

1. Send a message with styled buttons
2. Check if buttons display with colored backgrounds
3. Verify button functionality (callbacks still work)
4. Test on different devices (mobile, desktop)

## Migration from Old Code

### Old emoji-based approach:
```python
InlineKeyboardButton("🟢 Start", callback_data="start")
InlineKeyboardButton("🔵 Info", callback_data="info")
InlineKeyboardButton("🔴 Cancel", callback_data="cancel")
```

### New styled approach:
```python
apply_button_style(InlineKeyboardButton("▶️ Start", callback_data="start"), "positive")
apply_button_style(InlineKeyboardButton("ℹ️ Info", callback_data="info"), "primary")
apply_button_style(InlineKeyboardButton("❌ Cancel", callback_data="cancel"), "destructive")
```

Benefits:
- Real colored backgrounds instead of just emoji colors
- Professional appearance
- Better visual hierarchy
- Consistent with modern Telegram UI

## Future: When Library Updates

When python-telegram-bot adds native support for the `style` parameter, you can remove the workaround and use:

```python
InlineKeyboardButton("Start", callback_data="start", style="positive")
```

Until then, the workaround provides full functionality! 🎨
