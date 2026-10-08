# LayoutCAD

**An AI assistant for SolidWorks.** Describe what you want in plain English: a bracket, a welded frame, a sheet-metal enclosure, a whole assembly, or a part traced from a drawing. Your AI builds it in SolidWorks while you watch, and can make the drawings.

LayoutCAD is an [MCP](https://modelcontextprotocol.io) tool server. It works with any AI app on your PC that can use MCP tools: Claude Desktop, Claude Code, Cursor, Windsurf, VS Code (Copilot agent mode), LM Studio and others.

- **A spreadsheet holds the dimensions.** Every model has an Excel workbook, `<Name>_Layout.xlsx`, holding all its dimensions. Change a number there, or ask the AI to, and the parts update.
- **Each part is a normal SolidWorks part.** It is built standing where it belongs in the assembly, so you can open and edit it on its own.
- **Your hand edits are kept.** If you change a dimension in SolidWorks, the AI can copy it back into the spreadsheet.
- **It runs on your PC.** LayoutCAD drives your own copy of SolidWorks; only your conversation goes to the AI.

---

## What you need

| | |
|---|---|
| **Windows** | Windows 10 or 11, 64-bit |
| **SolidWorks** | Developed on SolidWorks 2026. Earlier versions work: where an API call is missing, the AI is told and finds another way. |
| **An AI app with MCP support** | Installed on the PC. AI websites used in a browser can't reach tools on your PC. |
| **Excel** | Optional. Handy for looking at the workbook; the AI doesn't need it. |
| **AsmFEA** | Optional. Needed only for checking parts against loads (see [Strength checks](#strength-checks-optional)). |

---

## Install

The installer brings its own Python, so there's nothing else to install.

1. **Get the installer:** **[⬇ Download layoutcad-0.18.1.zip](https://github.com/SimonRafferty/LayoutCAD/raw/main/releases/layoutcad-0.18.1.zip)** (about 100 MB; the download starts straight away). It's also in this repository's `releases` folder.
2. **Unzip it:** right-click the zip, then **Extract All**. The installer won't run from inside the zip.
3. **Run it:** double-click **`Install layoutcad.bat`**.
   - If Windows shows *"Windows protected your PC"*, click **More info**, then **Run anyway**.
   - It installs LayoutCAD to `%LOCALAPPDATA%\layoutcad\app`.
   - It then connects LayoutCAD to every AI app it finds: Claude Desktop (including the Microsoft Store version), Claude Code, Cursor, Windsurf, VS Code and LM Studio. Each app's settings are backed up first, and nothing else in them is changed.
4. **Restart your AI app** so it picks LayoutCAD up.
   - **Claude Desktop:** closing the window isn't enough. Right-click its icon by the clock, choose **Quit**, then start it again.

To check it worked, look for **layoutcad** in your AI app's tools or MCP list, and make sure it's switched on.

**Updating:** close your AI apps and run the new version's `Install layoutcad.bat`. It replaces the old version in the same place, so any app you set up by hand keeps working.

**Removing:** run **`Uninstall layoutcad.bat`**. It removes LayoutCAD from every AI app it was added to. Your projects are left alone.

### Using other AI apps

LayoutCAD is a standard local ("stdio") MCP server. If your app isn't one the installer recognises, use the file the installer writes next to `Install layoutcad.bat`: **`layoutcad settings for other AI apps.txt`**. It holds the exact command for your PC.

The command is:

```
Command:    %LOCALAPPDATA%\layoutcad\app\python\python.exe
Arguments:  -S "%LOCALAPPDATA%\layoutcad\app\server\main.py"
```

Most apps take it as JSON in their MCP settings file:

```json
{
  "mcpServers": {
    "layoutcad": {
      "command": "C:\\Users\\<you>\\AppData\\Local\\layoutcad\\app\\python\\python.exe",
      "args": ["-S", "C:\\Users\\<you>\\AppData\\Local\\layoutcad\\app\\server\\main.py"]
    }
  }
}
```

VS Code is slightly different: it uses `"servers"` instead of `"mcpServers"`, and needs `"type": "stdio"` added to the entry.

Restart the app after adding it. The AI app must run on the same PC as SolidWorks.

---

## Using it

### Your first model

1. **Start SolidWorks** and leave it open. LayoutCAD drives the SolidWorks that's running.
2. In your AI app, start a new chat and say something like:

   > Use layoutcad to start a new project: a 200 x 100 x 6 mm mounting plate with four M8 holes 20 mm in from the corners.

3. **Choose a folder.** The AI opens a folder picker (it may pop up behind other windows). Choose or create an empty folder for the project.
4. **Allow the tools.** When your AI asks to use the layoutcad tools, click **Allow**, or **Always allow** so it stops asking.
5. **Watch it build.** The AI takes pictures of the model to check its own work, and shows them to you.

### Things you can say

- *Make the plate 8 mm thick and rebuild.*
- *Add a 90° flange 40 mm deep along the long front edge.*
- *Mirror the left bracket to make the right-hand one.*
- *I changed the hole spacing in SolidWorks. Pull my changes from the Plate part into the spreadsheet.*
- *Here's a drawing (attach a picture). Trace the side profile to scale.*
- *Build it as a welded frame from 40 x 40 x 3 box section.*
- *Show me the assembly from underneath.*
- *Check which parts don't touch anything.*
- *Make drawings for all the parts.*
- *Check the bracket holds 2 kN on the end with a factor of safety of 2.*
- *Put the axle and the chassis together as a mechanism, pivoting on the radius-arm bushes.*
- *Animate the flap opening over 2 seconds.*
- *Run a motion study of the tipper and check the frame with AsmFEA at the worst moments.*

### Mechanisms

Several projects (an axle, a chassis) can be put together as a mechanism: each stays rigid, and they are joined only by pivots and limits.
Draw every project in the same coordinates, say where the pivots and limits are (the AI marks them on each project), and ask for the mechanism.
The AI checks the joints line up as drawn and builds the assembly in SolidWorks, telling you of anything that came out wrong way round.

### Motion studies and animations

A mechanism can be driven through its travel: a flap opening, a tipper body lifting, a crank turning.

1. **Say how it moves:** which pivot or slide is driven, from where to where (an angle or a distance from the position it's drawn in), and how long it takes, or a steady speed. The AI asks rather than guessing.
2. **The motion study** runs in SolidWorks Motion Analysis (if you asked for it, the AI just runs it; otherwise it asks once, as it takes seconds to a minute and uses SolidWorks) and reports the peak force on each joint and the peak torque or force the drive needs, with when each happens.
3. **The video:** if you ask for one (or say yes when the AI offers), the AI makes an AVI of the motion, in the project folder's `<Name>_MOTION` folder. Ask for a particular style if you like, for example *"start wide, swing round to the side, then close in on the hinge"*. It checks pictures from the video before handing it over.
4. **Strength at the worst moments (optional, with AsmFEA):** the peak moments become strength checks of the whole mechanism, frozen at that instant with its inertia and gravity as loads. You give the criteria (for example a maximum stress) and where the fixed part is mounted. **The AI always asks before it starts them:** they run one after another and can keep your PC busy for minutes or hours. When the motion study is for this check, it asks about both in one question.

Things to know:
- **Motion Analysis needs SolidWorks Premium** (or Simulation Professional or Premium). Without it, you still get the video, but no forces or strength checks.
- **Limit mates don't stop a driven part** in a motion study. Keep each move within its limits; the AI warns you if a move may run past one.
- **The strength checks treat each moment as static.** That suits most mechanisms, but not impacts, springy parts or vibration.

### What you get

Each project folder holds:

- the **workbook**, `<Name>_Layout.xlsx`;
- a **layout part** with the reference sketches;
- **one SolidWorks part per part**;
- an **assembly** that includes the layout as a reference;
- on request, **drawings**.

**Drawings:**
- The AI asks once per project whether you want **one drawing file per part** or **everything in as few files as possible**.
- Every drawing build also saves a **PDF** of each drawing file beside it.
- Parts are dimensioned from their own edges, with each hole located and called out, and the AI checks each sheet.

LayoutCAD never deletes your files. Parts it retires are moved to `_layoutcad\archive` in the project folder.

### Tips

- **Be specific about sizes, and say what's an estimate.** Guessed values are marked `MEASURE` in the workbook so they're easy to find and correct.
- **Work a step at a time, and correct as you go.** For example: *"That bracket should be on the outside of the rail."*
- **Rebuild freely.** Only the parts whose dimensions changed are opened again.
- **Close the workbook in Excel before asking for changes.** LayoutCAD can't write to it while Excel has it open.
- **Use one AI at a time.** Only one AI chat should be driving SolidWorks at once.
- **Long jobs carry on in the background.** If the AI says a build is still running, just wait or ask how it's going.
- **Try it on copies or a new folder first,** not on important existing work.

### Strength checks (optional)

With AsmFEA (a separate FEA tool for SolidWorks models) installed, the AI can check a design against load cases and change it until it passes.

1. Give it the loads, how the part is held, and the criteria: maximum stress, minimum factor of safety, maximum deflection or buckling.
2. It runs the analysis and reports **PASS** or **FAIL** for each criterion, with the margin.
3. If something fails, it resizes the part and runs again.

When the design passes, AsmFEA can produce the report.

**The AI always asks before it starts an analysis:** a run can keep your PC busy for minutes or hours.

Without AsmFEA, everything else works as normal, and the AI tells you AsmFEA isn't installed if you ask for a strength check.

---

## Troubleshooting

| Problem | What to do |
|---|---|
| LayoutCAD isn't in your AI's tools list | Restart the AI app properly. For Claude Desktop: right-click its icon by the clock, **Quit**, then start it again. If the installer didn't list your app, set it up from `layoutcad settings for other AI apps.txt`. |
| *"SolidWorks is not running"* | Start SolidWorks, wait until it has fully opened, and ask the AI to try again. |
| The AI says the workbook is open | Close it in Excel and say *"try again"*. |
| The folder picker doesn't appear | It's probably behind another window. Check the taskbar. |
| Parts come out in inches | SolidWorks' default part template isn't metric. Go to **Tools > Options > System Options > Default Templates**, pick a millimetre part and assembly template, then ask the AI to rebuild. |
| SolidWorks gets slow after a long session | Save, close and restart SolidWorks. LayoutCAD also restarts it between big builds if nothing of yours is open. |
| The installer says a file is in use | Close your AI apps and run it again. |
| Anything else | Tell the AI what happened: it can read its own build logs. Claude Desktop's server log is `%APPDATA%\Claude\logs\mcp-server-layoutcad.log`. |
