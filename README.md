# KT2 Panel

Static browser panel for controlling a Kungfu Turtle KT2 through the device's `/py` endpoint.

The project currently consists of a single file, [`panel.html`](./panel.html), which contains:

- The full HTML structure
- All CSS styling and responsive behavior
- The JavaScript that sends Python code to the robot
- The UI translations

## Installation

Upload the file panel.html to your device, folder my, then access it in your browser

http://192.168.1.37/file?path=/my/panel.html

Change the ip of your robot accordingly

## What This Project Does

The panel is a thin frontend over a Python-execution HTTP endpoint exposed by the KT2 device. It does not implement robot logic locally. Instead, each UI action builds a Python snippet and sends it to:

```text
POST /py
Content-Type: application/x-www-form-urlencoded

code=<python code here>
```

The KT2 executes the submitted Python code on-device, and the panel displays the response in the status bar and log.

This means the panel works as a remote control and quick scripting console, not as a standalone simulator.

## Architecture

## Frontend

There is no build step, bundler, or framework.

- Single static page: `panel.html`
- Styling: embedded CSS
- Behavior: embedded vanilla JavaScript
- Persistence: `localStorage`
- Transport: `fetch()` to relative path `/py`

## Runtime Model

The UI triggers JavaScript handlers.
Those handlers generate Python code strings.
Those strings are POSTed to `/py`.
The KT2 firmware runs the code and returns text.
The panel shows:

- A status message with the HTTP status code and a short response preview
- A reverse chronological log of commands sent and responses received

## Why Relative `/py` Calls Matter

The panel uses relative requests:

```js
fetch('/py', { ... })
```

That means the intended deployment model is:

1. `panel.html` is served by the KT2 itself, or by a reverse proxy in front of it.
2. The browser opens the page from the same origin that exposes `/py`.
3. All control actions use the same host automatically.

This avoids hardcoding an IP address in the current version.

## Project Structure

```text
kt2-panel/
├── panel.html
└── README.md
```

## Panel Features

## 1. Connection Panel

Purpose:

- Check that `/py` is reachable
- Show short request feedback

UI action:

- `Test /py`

Python sent:

```python
print("pong")
```

Expected use:

- Verify the robot is online
- Confirm the browser can reach the KT2 server
- Check that Python execution is working before using movement or LEDs

## 2. Lights Panel

Purpose:

- Set all LEDs to a single selected color
- Run a rainbow animation
- Turn LEDs off
- Pick quick preset colors

### Selected Color

The current implementation sends the selected color to all 10 LEDs:

```python
import car
color=0xff00ff
car.led.on([color] * 10)
```

Important detail:

- This was intentionally changed so `Turn LEDs on` lights all LEDs, not only one.

### Rainbow

The rainbow effect builds a list of 10 colors and rotates them over time:

```python
import car
import time
colors=[0xff0000,0xffff00,0x00ff00,0x00ffff,0x0000ff,0xff00ff]
for s in range(10):
    leds=[]
    for i in range(10):
        leds.append(colors[(i+s)%len(colors)])
    car.led.on(leds)
    time.sleep(0.08)
```

### Turn Off

```python
import car
car.led.off()
```

### Quick Palette

The palette contains these preset colors:

- `#ff0000`
- `#00ff00`
- `#0000ff`
- `#ffff00`
- `#ff00ff`
- `#00ffff`
- `#ffffff`
- `#ffa500`
- `#00ff99`
- `#7c4dff`
- `#ff4da6`
- `#222222`

Clicking a swatch:

1. Updates the color picker
2. Immediately sends the solid-color LED command

### Brightness Field

The numeric brightness field is currently visual only.

It does not affect the Python code sent to the device.

This is important if someone expects hardware brightness control. At the moment:

- The field exists in the UI
- No handler reads it
- No `/py` command uses it

## 3. Sound Panel

Purpose:

- Trigger built-in buzzer sounds
- Play simple notes like a piano
- Send custom music strings

### Built-In Sound Buttons

#### Hello

```python
import car
car.buzzer.hello()
```

#### Fire

```python
import car
car.buzzer.fire()
```

#### Stop Buzzer

```python
import car
car.buzzer.close()
```

### Piano

The piano buttons send frequency-based buzzer tones using `car.buzzer.freq(freq, dur)`.

Notes configured in the panel:

- `C`: `261`
- `D`: `294`
- `E`: `329`
- `F`: `349`
- `G`: `392`
- `A`: `440`
- `B`: `494`
- `C2`: `523`

Example command:

```python
import car
car.buzzer.freq(440, 0.25)
```

### Custom Music

The text area sends a raw music string to `car.buzzer.music(...)`.

Default value:

```text
1 2 3 5 3 2 1
```

Generated Python:

```python
import car
car.buzzer.music('''1 2 3 5 3 2 1''')
```

The code strips `'''` from the entered text before sending it, to avoid breaking the triple-quoted Python string.

### Scale Button

```python
import car
car.buzzer.music('''1 2 3 4 5 6 7 1''' )
```

## 4. Movement Panel

Purpose:

- Basic forward/backward movement
- Left/right turning
- Stop
- Quick posture reset
- Common stunt shortcuts

### Forward

```python
import car
import actions
q.play(actions.walk(q, ofs=actions.ofs_stand), 2, dly=0.1)
```

### Backward

```python
import car
import actions
q.play(actions.walk(q, x=-1, ofs=actions.ofs_stand), 2, dly=0.1)
```

### Turn Left

```python
import car
import actions
actions.c_pivot(q, 25)
```

### Turn Right

```python
import car
import actions
actions.c_pivot(q, -25)
```

### Stop

```python
import car
car.stop()
```

### Reset Pose

```python
import car
import actions
actions.one_key_reset(q)
```

### Bark Shortcut

```python
import car
import actions
q.play(actions.bark(q), dly=0.1)
```

### Back Flip Shortcut

```python
import car
import actions
q.play(actions.back_flip(q), dly=0.1)
```

## 5. Actions Panel

Purpose:

- Expose a dense grid of action shortcuts from the `actions` module

The panel defines these actions:

- `Shake Hand`
- `Push`
- `Pounce`
- `Turn Over`
- `Flip`
- `Back Flip`
- `Double Flip`
- `Left Punch`
- `Right Punch`
- `Slide`
- `Left Split`
- `Right Split`
- `Seesaw`
- `Spring`
- `Bark`
- `Throw`
- `Stand Low`
- `Head Up`
- `Head Down`
- `One Key Reset`

### Action Commands Sent

#### Shake Hand

```python
import car
import actions
q.play(actions.shake_hand(q), dly=0.1)
```

#### Push

```python
import car
import actions
q.play(actions.push(q), dly=0.1)
```

#### Pounce

```python
import car
import actions
q.play(actions.pounce(q), dly=0.1)
```

#### Turn Over

```python
import car
import actions
q.play(actions.turn_over(q), dly=0.1)
```

#### Flip

```python
import car
import actions
q.play(actions.flip(q), dly=0.1)
```

#### Back Flip

```python
import car
import actions
q.play(actions.back_flip(q), dly=0.1)
```

#### Double Flip

```python
import car
import actions
q.play(actions.double_flip(q), dly=0.1)
```

#### Left Punch

```python
import car
import actions
q.play(actions.left_punch(q), dly=0.1)
```

#### Right Punch

```python
import car
import actions
q.play(actions.right_punch(q, y=1), dly=0.1)
```

#### Slide

```python
import car
import actions
q.play(actions.slide(q), dly=0.1)
```

#### Left Split

```python
import car
import actions
q.play(actions.left_split(q), dly=0.1)
```

#### Right Split

```python
import car
import actions
q.play(actions.right_split(q), dly=0.1)
```

#### Seesaw

```python
import car
import actions
q.play(actions.seesaw(q), dly=0.1)
```

#### Spring

```python
import car
import actions
q.play(actions.spring(q), dly=0.1)
```

#### Bark

```python
import car
import actions
q.play(actions.bark(q), dly=0.1)
```

#### Throw

```python
import car
import actions
q.play(actions.throw(q), dly=0.1)
```

#### Stand Low

```python
import car
import actions
q.play(q.f(0, 0, 0, 0, ofs=actions.ofs_stand_low), dly=0.1)
```

#### Head Up

```python
import car
import actions
q.play(q.f(0, 0, 0, 0, ofs=actions.ofs_head_up), dly=0.1)
```

#### Head Down

```python
import car
import actions
q.play(q.f(0, 0, 0, 0, ofs=actions.ofs_head_down), dly=0.1)
```

#### One Key Reset

```python
import car
import actions
actions.one_key_reset(q)
```

## 6. Quick Python Panel

Purpose:

- Send arbitrary Python to `/py`
- Try unsupported commands without editing the UI
- Debug the KT2 runtime
- Access firmware features not exposed by a button

Default code:

```python
import car
car.led.on(0x00ffff)
```

### Run

The `Run` button sends the contents of the text area exactly as written.

This is the most powerful feature in the panel, because it bypasses all UI assumptions.

### Reset

The `machine.reset()` button sends:

```python
import machine
machine.reset()
```

This reboots the KT2 device.

## 7. Log Panel

Purpose:

- Show every outgoing request label
- Show the Python code that was sent
- Show the response status and response text preview

Logging behavior:

- New entries are prepended at the top
- Timestamps use `toLocaleTimeString()`
- Request lines look like `→ label: code`
- Response lines look like `← 200 response text`

This is useful for:

- Debugging failed actions
- Copying generated Python snippets
- Discovering how a UI button maps to the API

## 8. Keyboard Shortcuts

The panel binds these keys:

- `ArrowUp`: forward
- `ArrowDown`: backward
- `ArrowLeft`: turn left
- `ArrowRight`: turn right
- `Space`: stop
- `B`: bark
- `F`: back flip
- `R`: reset pose

The handler calls `preventDefault()` for these keys so the browser does not scroll the page while controlling the robot.

## 9. Multi-Language UI

The panel includes built-in translations for these languages:

- Spanish
- English
- Chinese
- Hindi
- Arabic
- Portuguese
- Bengali
- Russian
- Japanese
- Punjabi
- German
- Javanese
- Korean
- French
- Telugu
- Marathi
- Turkish
- Tamil
- Vietnamese
- Urdu
- Catalan
- Galician
- Basque

Language behavior:

- The selected language is stored in `localStorage` key `kt2_lang`
- On load, the page reads the saved language or defaults to Spanish
- Text is updated through `data-i18n` and `data-i18n-html` attributes
- English is the fallback base language if a translation key is missing

## 10. Responsive Layout

The panel uses CSS Grid and responsive breakpoints.

Desktop layout:

- Left column: connection, lights, sound
- Center column: movement, actions, quick Python
- Right column: log, shortcuts, notes

Responsive behavior:

- The 3-column layout uses `minmax(...)` so columns can shrink without overlapping
- At medium widths, the side columns narrow and the actions grid compresses
- At `1150px` and below, the layout becomes one column
- At `900px` and below, the palette and piano reduce the number of columns, and the d-pad shrinks

## How `/py` Works in This Project

The entire control model depends on one helper:

```js
async function sendPython(code, label = 'python') {
  const url = '/py';
  const body = new URLSearchParams({ code });
  const res = await fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: body.toString()
  });
  return await res.text();
}
```

So the contract assumed by this panel is:

- Endpoint: `/py`
- Method: `POST`
- Encoding: `application/x-www-form-urlencoded`
- Required field: `code`
- Value of `code`: Python source code as plain text
- Response: text, not necessarily JSON

## Confirmed `/py` API Surface Used by the Panel

This section documents the robot-side Python API that is explicitly exercised by the UI. These items are confirmed by the code in `panel.html`.

## Endpoint Contract

### `POST /py`

Form body:

```text
code=<python code>
```

Example with `curl`:

```bash
curl -X POST \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'code=print("pong")' \
  http://KT2_HOST/py
```

## Confirmed Modules

The panel imports and uses these modules:

- `car`
- `actions`
- `machine`
- `time`

## Confirmed Objects and Functions

### Module `car`

Confirmed calls:

- `car.stop()`
- `car.led.on(...)`
- `car.led.off()`
- `car.buzzer.hello()`
- `car.buzzer.fire()`
- `car.buzzer.close()`
- `car.buzzer.freq(freq, dur)`
- `car.buzzer.music(sequence)`

### Module `actions`

Confirmed calls and values:

- `actions.walk(q, ...)`
- `actions.c_pivot(q, angle)`
- `actions.shake_hand(q)`
- `actions.push(q)`
- `actions.pounce(q)`
- `actions.turn_over(q)`
- `actions.flip(q)`
- `actions.back_flip(q)`
- `actions.double_flip(q)`
- `actions.left_punch(q)`
- `actions.right_punch(q, y=1)`
- `actions.slide(q)`
- `actions.left_split(q)`
- `actions.right_split(q)`
- `actions.seesaw(q)`
- `actions.spring(q)`
- `actions.bark(q)`
- `actions.throw(q)`
- `actions.one_key_reset(q)`
- `actions.ofs_stand`
- `actions.ofs_stand_low`
- `actions.ofs_head_up`
- `actions.ofs_head_down`

### Object `q`

The panel also assumes a runtime object named `q` exists on the KT2 side and supports:

- `q.play(...)`
- `q.f(...)`

This object is not created by the panel, so it must already be defined by the KT2 firmware environment.

### Module `machine`

Confirmed call:

- `machine.reset()`

### Module `time`

Confirmed call:

- `time.sleep(seconds)`

## Confirmed LED Usage Patterns

The panel demonstrates two accepted input styles for `car.led.on(...)`:

### Single color value

Example:

```python
car.led.on(0x00ffff)
```

Meaning:

- Likely supported by the firmware
- In practice, on this setup it lit only one LED when used from the panel's solid-color action

### List of colors

Example:

```python
car.led.on([0xff0000] * 10)
```

And:

```python
car.led.on([
  0xff0000, 0xffff00, 0x00ff00, 0x00ffff, 0x0000ff,
  0xff00ff, 0xff0000, 0xffff00, 0x00ff00, 0x00ffff
])
```

Meaning:

- Confirmed working pattern for addressing all LEDs
- Used by the panel for solid color and rainbow

## Confirmed Buzzer Usage Patterns

### Predefined sounds

```python
car.buzzer.hello()
car.buzzer.fire()
car.buzzer.close()
```

### Direct frequency

```python
car.buzzer.freq(440, 0.25)
```

### Music notation string

```python
car.buzzer.music('''1 2 3 4 5 6 7 1''')
```

The exact grammar accepted by `car.buzzer.music(...)` is not documented in this repository, but numeric scale strings are clearly expected.

## Confirmed Motion Usage Patterns

### Immediate stop

```python
import car
car.stop()
```

### Play an animation or gait

```python
import car
import actions
q.play(actions.walk(q, ofs=actions.ofs_stand), 2, dly=0.1)
```

### Pivot turn

```python
import car
import actions
actions.c_pivot(q, 25)
```

### Pose or frame generation

```python
import car
import actions
q.play(q.f(0, 0, 0, 0, ofs=actions.ofs_head_up), dly=0.1)
```

## Inferred `/py` Behavior and API Notes

This section is intentionally separate. These points are reasonable conclusions from the code and observed behavior, but they are not formally guaranteed by documentation in this repository.

## Inferred Endpoint Semantics

- `/py` probably executes Python synchronously and returns output after the code finishes
- `print(...)` output likely appears in the HTTP response body
- Runtime exceptions likely become response text or error output visible in the log
- Long-running code will block the request until completion

## Inferred LED Topology

- The panel assumes there are 10 addressable LEDs
- Rainbow explicitly iterates `range(10)`
- Solid-color mode now sends a list of length 10

If a different firmware exposes a different LED count, the panel would need to be adjusted.

## Inferred Motion Runtime

Because the panel uses `q` without defining it, the KT2 environment likely preloads:

- A motion controller object named `q`
- The `car` module
- The `actions` motion library

If a firmware build does not expose `q`, all motion actions will fail even if `/py` itself works.

## Inferred Safety Caveats

- `machine.reset()` reboots the device immediately
- Arbitrary code in the Quick Python panel can move the robot, reboot it, or lock execution
- Long loops or delays can make the UI feel unresponsive while the request is still running

## Manual API Testing Examples

These examples can be sent from the Quick Python panel or with `curl`.

## Ping

```python
print("pong")
```

## All LEDs Blue

```python
import car
car.led.on([0x0000ff] * 10)
```

## LEDs Off

```python
import car
car.led.off()
```

## Single Tone

```python
import car
car.buzzer.freq(523, 0.3)
```

## Melody

```python
import car
car.buzzer.music('''1 1 5 5 6 6 5''')
```

## Walk Forward

```python
import car
import actions
q.play(actions.walk(q, ofs=actions.ofs_stand), 2, dly=0.1)
```

## Reset Pose

```python
import car
import actions
actions.one_key_reset(q)
```

## Reboot Device

```python
import machine
machine.reset()
```

## Local Development and Usage

## Running the Panel

The simplest intended setup is to serve `panel.html` from the KT2 device itself so that the page and `/py` share the same origin.

If you serve the file elsewhere, you need one of these:

- A reverse proxy that forwards `/py` to the KT2
- A modified frontend that points to the KT2 host explicitly
- Proper CORS support on the KT2 side

## No Build Needed

Open or serve `panel.html` directly. There are no dependencies to install.

## Recommended Workflow

1. Open the panel from the same origin as the KT2 `/py` endpoint.
2. Press `Test /py`.
3. Try LED or buzzer commands.
4. Use movement only after connectivity is confirmed.
5. Use Quick Python for experiments and unsupported firmware features.

## Limitations

- No authentication layer is implemented in this project
- No schema validation exists for Python snippets
- The UI assumes KT2-specific firmware behavior
- The brightness field does not control hardware brightness
- Error handling is limited to HTTP/text feedback
- The API documentation here is based on observed usage, not official firmware docs

## Future Improvements

- Add official API docs if firmware documentation becomes available
- Detect LED count instead of hardcoding `10`
- Make the brightness field actually affect LED output
- Add named presets for poses and animations
- Add response parsing if `/py` ever returns structured data
- Split `panel.html` into separate HTML, CSS, and JS files if the project grows

## Source of Truth

For this repository, the source of truth is:

- The commands constructed in `panel.html`
- The labels and handlers bound in JavaScript

If behavior on the robot differs from this README, trust the firmware behavior first and then update the panel and documentation to match.
