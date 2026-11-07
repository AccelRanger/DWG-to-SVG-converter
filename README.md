# dwg2svg

A tiny command-line tool that converts a `.dwg` file into an `.svg` file.

## How it works (for the user)

1. Put `dwg2svg.py` and your `.dwg` file in the same folder.
2. Open Command Prompt in that folder.
3. Run:

   ```
   python dwg2svg.py
   ```
4. When asked, type the file name:

   ```
   Enter the .dwg file name: floorplan.dwg
   ```
5. The tool creates `floorplan.svg` in the same folder.

That's it. Typing the name without `.dwg` (`floorplan`) also works.

## One-time setup

| Requirement | Why | How |
|---|---|---|
| Python 3.9+ | runs the tool | https://www.python.org/downloads/ |
| `ezdxf` library | draws the SVG | `pip install ezdxf matplotlib` |
| ODA File Converter (free) | reads the proprietary DWG format | https://www.opendesign.com/guestfiles/oda_file_converter |

Install ODA File Converter with its default settings. The tool finds it automatically.

## Folder layout

```
dwg2svg/
├── dwg2svg.py     <- the program
├── floorplan.dwg  <- your input file
└── floorplan.svg  <- created output
```

## Limitations

- DWG is a closed format, so the tool converts DWG -> DXF (via ODA) -> SVG (via ezdxf).
- Very complex drawings (3D solids, proxy objects, some custom fonts) may render incompletely.
- Only the Model space is exported, not paper-space layouts.

See [DESIGN.md](DESIGN.md) for how it is built and the full source code.
