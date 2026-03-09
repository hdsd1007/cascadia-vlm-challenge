# Cascadia Board Game Scoring — VLM Challenge

Automated scoring of a 3-player [Cascadia](https://www.alderac.com/cascadia/) board game using **Google Gemini 2.5 Flash** as a Vision-Language Model (VLM). The system extracts wildlife scoring rules, counts animal tokens, identifies habitat tiles, and computes the final scores — all from board images.

## Approach

**Multi-step VLM pipeline with image preprocessing:**

| Step | What it does | API Calls |
|------|-------------|-----------|
| Preprocessing | Crop per-player boards, 2x upscale, sharpen + contrast boost | 0 |
| Rule extraction | Read wildlife scoring card image | 1 |
| Animal counting | Count tokens per player board (one call each) | 3 |
| Verification | Re-examine each board with prior counts to catch miscounts | 3 |
| Habitat identification | Identify habitat types and largest contiguous groups per player | 3 |
| Habitat scoring | Compute majority bonuses (+2 per habitat type leader) | 0 (Python) |
| Wildlife scoring | Apply scoring rules to verified counts | 1 |
| Final summary | Generate combined Wildlife + Habitat + Nature Token tables | 1 |

**Total: 12 API calls**

### Key Design Decisions

- **Per-player crops** instead of full board — VLM focuses on one board at a time for better accuracy
- **Count-then-verify** two-pass approach — catches token miscounts before scoring
- **Habitat scoring separate from wildlife** — different visual task, avoids diluting prompts
- **Majority bonuses in Python** — deterministic arithmetic, not left to the VLM
- **Text context reuse** — scoring and summary steps use text from prior responses, no re-sending images

## Scoring Categories

1. **Wildlife (W):** 5 animal types (Bear, Elk, Hawk, Salmon, Fox) scored by spatial arrangement rules on the scoring cards
2. **Habitat (H):** Largest contiguous group of each of 5 habitat types (Mountain, Forest, Prairie, Wetland, River) + majority bonuses
3. **Nature Tokens:** 1 point per unspent token (manual entry — tokens are off-board, not visible in images)

## How to Run

1. Open `cascadia_challenge_phronetics.ipynb` in [Google Colab](https://colab.research.google.com/)
2. Upload the 3 image files to the Colab runtime (or mount Google Drive)
3. Add your Gemini API key in Colab **Secrets** (key icon, left sidebar) with the name `GOOGLE_API_KEY`
4. Edit the `nature_tokens` dict if you know the token counts
5. Run all cells

## Files

| File | Description |
|------|-------------|
| `cascadia_challenge_phronetics.ipynb` | Main notebook (with outputs from a complete run) |
| `wildlife scoring card.jpg` | Scoring rules card image |
| `player tiles.png` | 3-player board image |
| `Cascadia Scoring sheet.jpg` | Reference scoring sheet |

## Assumptions

- Player boards are arranged left-to-right (Player 1, 2, 3) in `player tiles.png`
- Each hex tile has one primary habitat type (dual-habitat tiles scored by dominant background)
- Nature tokens are off-board and cannot be detected from images — entered manually
- No habitat verification pass needed since tile backgrounds are large and visually distinct
