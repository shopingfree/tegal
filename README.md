#!/usr/bin/env python3
"""
Cross-platform terminal key handler (safe demo).
- Captures keys ONLY while this program's terminal is focused.
- Exits on Esc or 'q'.
"""

import curses
import time

# Map a few common key codes to human-friendly names
KEY_NAMES = {
    curses.KEY_UP: "KEY_UP",
    curses.KEY_DOWN: "KEY_DOWN",
    curses.KEY_LEFT: "KEY_LEFT",
    curses.KEY_RIGHT: "KEY_RIGHT",
    curses.KEY_HOME: "KEY_HOME",
    curses.KEY_END: "KEY_END",
    curses.KEY_PPAGE: "KEY_PAGE_UP",
    curses.KEY_NPAGE: "KEY_PAGE_DOWN",
    curses.KEY_IC: "KEY_INSERT",
    curses.KEY_DC: "KEY_DELETE",
    curses.KEY_BACKSPACE: "KEY_BACKSPACE",
    9: "TAB",
    10: "ENTER",
    27: "ESC",
}

def main(stdscr):
    curses.curs_set(0)              # hide cursor
    stdscr.nodelay(True)            # non-blocking getch
    stdscr.keypad(True)             # enable special keys
    start = time.time()
    history = []                    # last few keys (not written to disk)

    def draw():
        stdscr.erase()
        stdscr.addstr(0, 0, "Keyboard demo (safe, in-app only)")
        stdscr.addstr(1, 0, "Press keys. Exit with Esc or 'q'.")
        stdscr.addstr(2, 0, "-" * 50)

        # Show last N keys
        stdscr.addstr(4, 0, "Recent keys:")
        for i, item in enumerate(history[-10:][::-1], start=1):
            stdscr.addstr(4 + i, 2, f"{i:>2}. {item}")

        # Status footer
        elapsed = time.time() - start
        h, w = stdscr.getmaxyx()
        footer = f"Running {elapsed:.1f}s | Window size: {h}x{w}"
        stdscr.addstr(h - 1, 0, footer[:max(0, w-1)])
        stdscr.refresh()

    draw()
    while True:
        k = stdscr.getch()
        if k == -1:
            # No key pressed—just update the UI occasionally
            time.sleep(0.02)
            draw()
            continue

        # Exit conditions
        if k in (27, ord('q'), ord('Q')):  # ESC or q
            break

        # Decode key
        name = KEY_NAMES.get(k)
        if name is None:
            try:
                if 0 <= k <= 255:
                    ch = chr(k)
                    # Printable?
                    name = f"'{ch}' (code {k})" if ch.isprintable() else f"code {k}"
                else:
                    name = f"code {k}"
            except Exception:
                name = f"code {k}"

        history.append(name)
        draw()

curses.wrapper(main)
