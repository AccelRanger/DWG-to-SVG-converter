# DESIGN

## Goal

The simplest possible DWG -> SVG converter, driven from Command Prompt with one question: *"what is the file name?"*

## Pipeline

```
name.dwg --[ODA File Converter]--> name.dxf (temp) --[ezdxf]--> name.svg
```

DWG is proprietary, so there is no simple pure-Python reader. ODA File Converter (free) translates DWG to the open DXF format; `ezdxf` then renders DXF to SVG. The temporary DXF is deleted afterwards.

## Program flow

1. Ask: `Enter the .dwg file name:`
2. Add `.dwg` if missing; look for the file next to the script.
3. Locate `ODAFileConverter.exe` (PATH or `C:\Program Files\ODA\...`).
4. Convert DWG -> DXF into a temp folder.
5. Render DXF model space -> SVG with `ezdxf`.
6. Save `<name>.svg` beside the input; print the result or a clear error.

## Source: `dwg2svg.py`

```python
import glob
import shutil
import subprocess
import tempfile
from pathlib import Path

import ezdxf
from ezdxf.addons.drawing import Frontend, RenderContext, layout, svg

FOLDER = Path(__file__).resolve().parent


def find_oda():
    exe = shutil.which("ODAFileConverter")
    if exe:
        return exe
    hits = sorted(glob.glob(r"C:\Program Files\ODA\ODAFileConverter*\ODAFileConverter.exe"))
    return hits[-1] if hits else None


def dwg_to_dxf(dwg: Path, out_dir: Path, oda: str) -> Path:
    in_dir = out_dir / "in"
    in_dir.mkdir()
    shutil.copy(dwg, in_dir / dwg.name)
    subprocess.run(
        [oda, str(in_dir), str(out_dir), "ACAD2018", "DXF", "0", "1", dwg.name],
        check=True,
    )
    dxf = out_dir / (dwg.stem + ".dxf")
    if not dxf.exists():
        raise RuntimeError("ODA File Converter did not produce a DXF file.")
    return dxf


def dxf_to_svg(dxf: Path, svg_path: Path):
    doc = ezdxf.readfile(dxf)
    backend = svg.SVGBackend()
    Frontend(RenderContext(doc), backend).draw_layout(doc.modelspace())
    svg_path.write_text(backend.get_string(layout.Page(0, 0)), encoding="utf-8")


def main():
    name = input("Enter the .dwg file name: ").strip().strip('"')
    if not name.lower().endswith(".dwg"):
        name += ".dwg"

    dwg = FOLDER / name
    if not dwg.exists():
        print(f"File not found: {dwg}")
        return

    oda = find_oda()
    if not oda:
        print("ODA File Converter not found. Install it first (see README.md).")
        return

    out = dwg.with_suffix(".svg")
    try:
        with tempfile.TemporaryDirectory() as tmp:
            dxf = dwg_to_dxf(dwg, Path(tmp), oda)
            dxf_to_svg(dxf, out)
        print(f"Done: {out.name}")
    except Exception as e:
        print(f"Conversion failed: {e}")


if __name__ == "__main__":
    main()
```

## Possible later improvements

- Accept several names or `*.dwg` to batch-convert.
- Package as a single `.exe` with PyInstaller (`pyinstaller --onefile dwg2svg.py`).
- Export paper-space layouts.
- Replace ODA with LibreDWG (`dwg2dxf`) if a fully open-source chain is preferred.
