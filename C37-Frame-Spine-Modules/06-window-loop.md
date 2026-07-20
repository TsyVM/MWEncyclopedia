# C37.6 — The Window & Message Loop

> **The one-sentence version:** `WndProc` (`0x6DB6C0`) handles the OS window messages via a jump table —
> `WM_PAINT` paints, `WM_ACTIVATE` emits a `"LostFocus"` event so the game can pause/mute when backgrounded —
> the boundary between Windows and the game loop.

[← C37.5 — The module update order](05-module-order.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md) ·
[Next: C37.7 — Reading the frame spine in RE →](07-reading-spine.md)

---

## The OS boundary

A Windows game has two loops: the **game loop** ([C37.2](02-winmain-loop.md)) that ticks the simulation, and the
**message loop** that handles OS window events (paint, focus, close, input). `WndProc` (`0x6DB6C0`) is the
window procedure — the callback Windows invokes with each message. Its signature is verified: `ret 0x10`
(stdcall, 4 arguments = `hwnd, msg, wParam, lParam`), the standard `WndProc` prototype.

So `WndProc` is the game's ear to the OS: Windows sends it messages, and it translates the relevant ones into
game actions.

## The message switch

`WndProc` dispatches on the message id via a **jump table `[0x6DB97C]`** — a switch over the message types it
cares about:

- **`WM_PAINT`** → `BeginPaint` / `EndPaint` — handle repaint requests (the game renders in its loop, so this is
  minimal).
- **`WM_ACTIVATE`** → emits the **`"LostFocus"` event** when the window is deactivated (backgrounded).
- **`IsIconic`** check — whether the window is minimised.
- Other messages (close, size, input) dispatched as needed; unhandled ones fall through to `DefWindowProc`.

So the switch picks out the messages that affect the game (focus, paint, close) and ignores the rest — the
standard `WndProc` pattern.

> ✅ *Verified:* `WndProc (0x6DB6C0)` is `ret 0x10` (stdcall, 4 args), dispatches via jump table `[0x6DB97C]`,
> handles `WM_PAINT` (BeginPaint/EndPaint), and on `WM_ACTIVATE` emits `"LostFocus"`; checks `IsIconic`.

## Focus: pause and mute when backgrounded

The most gameplay-relevant message is **`WM_ACTIVATE`**, which fires when the window gains or loses focus. On
losing focus, `WndProc` emits the **`"LostFocus"` event**, which lets the game react to being backgrounded:

- **Pause** the simulation (or continue, depending on mode) — so an alt-tabbed pursuit doesn't run unattended.
- **Mute** the audio ([Chapter 19](../C19-Audio-Banks/C19-Audio-Banks.md)) — so a backgrounded game is silent.
- **Release** exclusive resources (input device, display) as needed.

So the `"LostFocus"` event is the hook that makes the game a well-behaved window — pausing and quieting when the
player switches away, resuming when they return (the matching focus-gained path). This is why the game doesn't
blare audio from the taskbar.

## Two loops, one program

The window loop and the game loop coexist ([C37.2](02-winmain-loop.md)):

```
main loop (WinMain):                     message loop (pumped each iteration):
  while(!quit) {                           while (PeekMessage) {
    process OS messages ──────────────────▶  TranslateMessage; DispatchMessage → WndProc
    dt = QPC delta                         }
    FrameTick(dt)                        WndProc handles WM_ACTIVATE→"LostFocus", WM_PAINT, close→set quit
  }
```

The game loop **pumps** the message queue each iteration (so window events are handled promptly), then ticks the
frame. `WndProc` runs during the pump, handling each message. So the two loops interleave: pump messages
(`WndProc`), then tick the game (`FrameTick`), every iteration. A **close** message ultimately sets the quit
flag ([C37.2](02-winmain-loop.md)), ending the game loop.

## RE implications

- **`WndProc (0x6DB6C0)` is the OS message handler** — `ret 0x10` (stdcall, 4 args), jump table `[0x6DB97C]`.
- **`WM_ACTIVATE` → `"LostFocus"`** — the hook for pausing/muting when backgrounded
  ([Chapter 19](../C19-Audio-Banks/C19-Audio-Banks.md)).
- **The message loop is pumped each game-loop iteration** — window events handled promptly, then `FrameTick`.
- **A close message sets the quit flag** — the OS path to game shutdown ([C37.2](02-winmain-loop.md)).

---

### Key takeaways

- `WndProc (0x6DB6C0)` handles OS window messages (`ret 0x10`, stdcall, 4 args) via jump table `[0x6DB97C]`.
- It handles `WM_PAINT` (BeginPaint/EndPaint) and, on `WM_ACTIVATE`, emits the **`"LostFocus"`** event.
- `"LostFocus"` lets the game **pause and mute** when backgrounded — a well-behaved window.
- The **message loop** (pumping `WndProc`) and the **game loop** (`FrameTick`) interleave each iteration.
- A window **close** sets the quit flag, driving shutdown ([C37.2](02-winmain-loop.md)).

**Continue:** [C37.7 — Reading the frame spine in RE](07-reading-spine.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md)
