# 3D Wood Pattern - AI Agent Instructions

## Project Overview

Post-processing scripts for 3D printer G-code files that add visual texture effects:
- **wood.py**: Varies print temperature to create wood-grain patterns (darker = hotter)
- **colormix.py**: Randomizes material mixing ratios or tool changes along Z-axis

Both scripts use Perlin noise algorithms for natural-looking pseudo-random texture generation.

## Architecture & Dual-Mode Design

**Critical**: Each script has TWO execution modes with shared code:

1. **Cura Plugin Mode**: Imported by Ultimaker Cura as a post-processing plugin
   - [Woodgrain_Cura.py](wood/Woodgrain_Cura.py): Wrapper with PyQt6 GUI integration
   - Uses `from ..Script import Script` and Cura's plugin system

2. **Standalone CLI Mode**: Direct command-line execution via Python
   - [wood.py](wood/wood.py) and [colormix.py](colormix/colormix.py) detect standalone mode via:
   ```python
   try:
       filename  # Only defined in Cura context
   except NameError:
       # Parse command-line args with getopt
   ```

Both modes process G-code files in-place (modifying the original file).

## Python Version

**Python 3 Required**: All scripts have been converted to Python 3. UTF-8 encoding is used for all file operations.

**Known Issue**: [wood.py#L56](wood/wood.py#L56) and [colormix.py#L46](colormix/colormix.py#L46) show `filename` undefined warnings. This is intentional - the code checks if `filename` exists to detect whether running as Cura plugin (where it's defined) or standalone mode (where it raises `NameError`).

## Key G-code Processing Concepts

### Temperature Modulation (wood.py)
- Inserts `M104 S<temp>` commands at layer changes (Z-height transitions)
- Uses normalized Perlin noise mapped to `[minTemp, maxTemp]` range
- **Z-Hop Detection**: Scans ahead to avoid temp changes during retractions (see `z_hop_scan_ahead()`)
- Generates ASCII-art temperature graph as G-code comments: `;WoodGraph: Z 3.200000 @239C | ##################.`

### Critical Parameters
- `grainSize`: Controls Perlin noise frequency (wood grain density)
- `spikinessPower`: Exponent for noise value (>1 makes dark bands sparser)
- `skipStartZ`: Ignore first N mm (for rafts/brims)
- `maxUpward`/`maxDownward`: Rate-limit temp changes for firmware compatibility

## Testing & Development Workflow

Test scripts live in `*/testing/` directories:
```bash
# wood.py test (from wood/testing/)
bash wood.sh [source_file.gcode]
# Copies source, runs wood.py with test params, outputs modified file

# colormix.py test (from colormix/testing/)
bash colormix.sh [randomSeed] [speed]
# Processes test cylinder with different mix parameters
```

**No unit tests exist** - validation is visual inspection of printed objects and ASCII graphs.

## File Processing Conventions

1. **In-place modification**: Both scripts overwrite input files (warn users to backup!)
2. **EOL detection**: Auto-detects Windows (`\r\n`) vs Unix (`\n`) line endings
3. **G-code parsing**: Use `get_value(line, 'G')` and `get_z(line)` helpers for robust parsing
4. **Comment filtering**: Skip lines starting with `;` in temperature logic

## Common Maintenance Tasks

### Adding New Parameters
Update THREE locations:
1. Script header `#Param:` comments (Cura reads these)
2. Standalone mode `getopt.getopt()` argument parsing
3. `plugin_standalone_usage()` help text

### Debugging Temperature Issues
Check generated `;WoodGraph:` comments at end of output G-code - they visualize the temperature curve without printing.

## External Dependencies

- **Cura Plugin Mode**: Requires Ultimaker Cura 5.0+ with PyQt6
- **Standalone Mode**: Pure Python with stdlib (no external deps)
- **3D Printing Context**: Assumes familiarity with G-code commands (M104, M109, G0/G1, etc.)

## Important Notes

- G-code files can be VERY large (multi-MB) - processing is memory-intensive
- Wood filament (LayWoo-D3) burns at high temps - script aims for natural wood darkening without clogging
- Not compatible with Cura's "One at a Time" print sequence
- Test parameter changes with graph visualization before printing
