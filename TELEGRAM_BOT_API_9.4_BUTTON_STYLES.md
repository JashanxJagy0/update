# Telegram Bot API 9.4 - Colored Button Styles Implementation

## Overview

**Telegram Bot API 9.4** (released February 2026) introduced native support for colored inline keyboard buttons via the `style` parameter. This document explains the correct implementation.

## Correct Style Values

| Style Value | Color | Use Case |
|-------------|-------|----------|
| `'success'` | 🟢 Green | Positive actions (Start, Confirm, Withdraw, Cashout, Selected items) |
| `'danger'` | 🔴 Red | Negative/Cancel actions (Back, Cancel, Delete) |
| `'primary'` | 🔵 Blue | Information/Navigation (Deposit, Games, Unselected items) |
| (no style) | ⚪ Gray | Neutral actions (Settings, Info) |

## ❌ Common Mistakes

**WRONG values** (these will cause "invalid button style specified" error):
- ❌ `style='positive'` (should be `success`)
- ❌ `style='destructive'` (should be `danger`)
- ❌ `style='secondary'` (not supported)

## Implementation Workaround

Since python-telegram-bot library doesn't have native support yet, use this workaround:

### Helper Functions

```python
def apply_button_style(button, style):
    """
    Apply style to InlineKeyboardButton using dict injection.
    
    Args:
        button: InlineKeyboardButton object
        style: One of 'success' (green), 'danger' (red), or 'primary' (blue)
    
    Returns:
        dict: Button dictionary with style parameter injected
    """
    btn_dict = button.to_dict()
    btn_dict['style'] = style
    return btn_dict

def create_styled_keyboard(keyboard_array):
    """
    Create a styled keyboard from a 2D array of buttons/dicts.
    
    Args:
        keyboard_array: 2D list of InlineKeyboardButton objects or dicts
    
    Returns:
        dict: Formatted keyboard in {'inline_keyboard': [[...]]} format
    """
    styled_rows = []
    for row in keyboard_array:
        styled_row = []
        for item in row:
            if isinstance(item, dict):
                styled_row.append(item)
            else:
                styled_row.append(item.to_dict() if hasattr(item, 'to_dict') else item)
        styled_rows.append(styled_row)
    return {'inline_keyboard': styled_rows}
```

### Usage Examples

#### Single Button with Style

```python
# GREEN button (success)
btn_withdraw = InlineKeyboardButton("Withdraw", callback_data="withdraw")
styled_btn = apply_button_style(btn_withdraw, 'success')

# RED button (danger)
btn_cancel = InlineKeyboardButton("Cancel", callback_data="cancel")
styled_btn = apply_button_style(btn_cancel, 'danger')

# BLUE button (primary)
btn_info = InlineKeyboardButton("Info", callback_data="info")
styled_btn = apply_button_style(btn_info, 'primary')
```

#### Complete Keyboard

```python
keyboard = [
    [
        apply_button_style(InlineKeyboardButton("Deposit", callback_data="deposit"), 'primary'),  # BLUE
        apply_button_style(InlineKeyboardButton("Withdraw", callback_data="withdraw"), 'success')  # GREEN
    ],
    [
        InlineKeyboardButton("Settings", callback_data="settings").to_dict(),  # No style (gray)
        apply_button_style(InlineKeyboardButton("Cancel", callback_data="cancel"), 'danger')  # RED
    ]
]

reply_markup = create_styled_keyboard(keyboard)

# Send message with styled keyboard
await update.message.reply_text(
    "Choose an option:",
    reply_markup=reply_markup  # Pass dict directly, NOT InlineKeyboardMarkup
)
```

## Implementation in Casino Bot

### 1. Start Menu
```python
keyboard = [
    [
        apply_button_style(InlineKeyboardButton("💎 Deposit", callback_data="main_deposit"), 'primary'),  # BLUE
        apply_button_style(InlineKeyboardButton("💸 Withdraw", callback_data="main_withdraw"), 'success')  # GREEN
    ],
    [
        apply_button_style(InlineKeyboardButton("🎮 Games", callback_data="main_games"), 'primary'),  # BLUE
        apply_button_style(InlineKeyboardButton("📊 More", callback_data="main_more"), 'danger')  # RED
    ]
]
reply_markup = create_styled_keyboard(keyboard)
```

### 2. Keno Game
```python
# Number selection
for i in range(1, 41):
    btn = InlineKeyboardButton(str(i), callback_data=f"keno_pick_{game_id}_{i}")
    if i in selected_numbers:
        buttons.append(apply_button_style(btn, 'success'))  # GREEN for selected
    else:
        buttons.append(apply_button_style(btn, 'primary'))  # BLUE for unselected

# Action buttons
place_bet = apply_button_style(
    InlineKeyboardButton("✅ Place Bet", callback_data=f"keno_place_{game_id}"),
    'success'  # GREEN
)
cancel = apply_button_style(
    InlineKeyboardButton("❌ Cancel", callback_data=f"keno_cancel_{game_id}"),
    'danger'  # RED
)
```

### 3. Mines Game
```python
# Tiles
for i in range(1, 26):
    btn = InlineKeyboardButton("🟦", callback_data=f"mines_pick_{game_id}_{i}_{user_id}")
    buttons.append(apply_button_style(btn, 'primary'))  # BLUE tiles

# Action buttons
cashout = apply_button_style(
    InlineKeyboardButton(f"💰 Cashout (${winnings:.2f})", callback_data=f"mines_cashout_{game_id}_{user_id}"),
    'success'  # GREEN
)
random = apply_button_style(
    InlineKeyboardButton("🎲 Random", callback_data=f"mines_random_{game_id}_{user_id}"),
    'primary'  # BLUE
)
```

### 4. Tower Game
```python
keyboard = [
    [apply_button_style(InlineKeyboardButton("▶️ Start Game", callback_data="tower_start_game"), 'success')],  # GREEN
    [
        InlineKeyboardButton("◀️", callback_data="tower_diff_prev").to_dict(),
        apply_button_style(InlineKeyboardButton(f"⚙️ {difficulty}", callback_data="tower_diff_info"), 'primary'),  # BLUE
        InlineKeyboardButton("▶️", callback_data="tower_diff_next").to_dict()
    ],
    [apply_button_style(InlineKeyboardButton("🔙 Back", callback_data="cancel_game"), 'danger')]  # RED
]
```

### 5. Roulette Game
```python
# Main menu
keyboard = [
    [apply_button_style(InlineKeyboardButton("▶️ Start", callback_data=f"roul_start_{user_id}"), 'success')],  # GREEN
    [InlineKeyboardButton("🎯 Bet on Number", callback_data=f"roul_bet_number_{user_id}").to_dict()],
    # ... other options ...
    [apply_button_style(InlineKeyboardButton("❌ Cancel Bet", callback_data=f"roul_cancel_{user_id}"), 'danger')]  # RED
]

# Number selection
for num in range(0, 37):
    btn = InlineKeyboardButton(f"{emoji} {num}", callback_data=f"roul_num_{num}_{user_id}")
    if num in selected_numbers:
        row.append(apply_button_style(btn, 'success'))  # GREEN for selected
    else:
        row.append(btn.to_dict())  # No style for unselected
```

### 6. Leaderboard
```python
keyboard = [
    [
        InlineKeyboardButton("📅 Weekly", callback_data=f"leaderboard_weekly_{user_id}").to_dict(),
        InlineKeyboardButton("📆 Monthly", callback_data=f"leaderboard_monthly_{user_id}").to_dict()
    ],
    [apply_button_style(InlineKeyboardButton("🔙 Back to More", callback_data="main_more"), 'danger')]  # RED
]
```

## Technical Details

### Button Format Sent to Telegram

```json
{
    "inline_keyboard": [
        [
            {
                "text": "💸 Withdraw",
                "callback_data": "main_withdraw",
                "style": "success"
            },
            {
                "text": "💎 Deposit",
                "callback_data": "main_deposit",
                "style": "primary"
            }
        ],
        [
            {
                "text": "❌ Cancel",
                "callback_data": "cancel",
                "style": "danger"
            }
        ]
    ]
}
```

### Why Use dict Instead of InlineKeyboardMarkup?

When using `InlineKeyboardMarkup(keyboard)`, the library validates the structure and may strip unknown parameters like `style`. By passing a dict directly to `reply_markup`, we bypass validation and send the `style` parameter directly to Telegram's API.

### Backward Compatibility

Older Telegram clients that don't support Bot API 9.4 will simply ignore the `style` parameter and display buttons with default styling. No errors will occur.

## Testing

### Test Button Colors

1. Send message with styled buttons
2. Check button colors on different clients:
   - Telegram Desktop
   - Telegram Mobile (iOS/Android)
   - Telegram Web

### Expected Results

- 🟢 Green buttons: Start, Withdraw, Cashout, Place Bet, Selected items
- 🔴 Red buttons: Cancel, Back, Delete
- 🔵 Blue buttons: Deposit, Games, Info, Unselected items
- ⚪ Gray buttons: Settings, neutral actions

## Troubleshooting

### Error: "invalid button style specified"

**Cause**: Using wrong style values

**Solution**: Use only `'success'`, `'danger'`, or `'primary'`

### Buttons Not Colored

**Possible causes**:
1. Using `InlineKeyboardMarkup()` instead of dict - use `create_styled_keyboard()` instead
2. Old Telegram client - update to latest version
3. Style parameter not injected - check button creation code

### Library Update

When python-telegram-bot adds native support for `style` parameter, you can replace:

```python
# Current workaround
apply_button_style(InlineKeyboardButton("Text", callback_data="data"), 'success')

# Future native support
InlineKeyboardButton("Text", callback_data="data", style='success')
```

## References

- Telegram Bot API 9.4 Changelog (February 2026)
- Official Documentation: https://core.telegram.org/bots/api#inlinekeyboardbutton
- python-telegram-bot library: https://python-telegram-bot.org/

## Summary

✅ Use `'success'` for green buttons (NOT 'positive')
✅ Use `'danger'` for red buttons (NOT 'destructive')
✅ Use `'primary'` for blue buttons
✅ Use `apply_button_style()` helper function
✅ Use `create_styled_keyboard()` to format keyboards
✅ Pass dict to `reply_markup` parameter, NOT InlineKeyboardMarkup object

**All buttons in the casino bot now have beautiful colored backgrounds using native Telegram Bot API 9.4 features!** 🎨
