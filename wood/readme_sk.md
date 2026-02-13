# Python 3 Conversion Notes

I converted this wood script with Claude Sonnet AI agent. 

The script is working now on Python 3 Win11 machine.

## Working Command

```bash
python wood.py --min 190 --max 235 --grain 3.0 --firstTemp 215 --file "C:\Users\blub\file.gcode"
```

## Conversion Details

The conversion from Python 2 to Python 3 required several important changes:

1. **Removed `xrange` compatibility** - All `xrange()` calls were replaced with `range()` directly, as Python 3 only has `range()`

2. **Fixed file encoding** - Changed from UTF-8 to `latin-1` encoding for G-code file operations. This was critical because G-code files can contain binary data or special characters that aren't valid UTF-8. The initial UTF-8 approach caused crashes like:
   ```
   UnicodeDecodeError: 'utf-8' codec can't decode byte 0xb0 in position 86
   ```

3. **Added division by zero protection** - When processing G-code files with uniform noise values, the normalization step could divide by zero. Added a check to handle this edge case gracefully by setting all noise values to 0.5 (middle of temperature range).

4. **Fixed bare exception clauses** - Changed `except:` to `except (ValueError, AttributeError):` for proper Python 3 exception handling practices.

## Files Modified
- `wood.py` - Main standalone/CLI script
- `colormix.py` - Color mixing script
- `Woodgrain_Cura.py` - Cura plugin wrapper
- `.github/copilot-instructions.md` - Updated documentation

All scripts now use `encoding="latin-1"` for robust G-code file handling across different slicer outputs.
