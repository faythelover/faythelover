import os
import sys
import time
import pickle
from pathlib import Path

import colorful as cf
from pynput import keyboard as kb
from pynput import mouse as ms

if hasattr(sys, "frozen"):
    PARENT = Path(sys.executable).parent
else:
    PARENT = Path(__file__).parent

os.system("cls")

class Mode:
    first = 1
    third = 3

class State:
    toggle = "t"
    hold   = "h"

cf.use_true_colors()
cf.update_palette({'brightblue': '#3b8eea'})
cf.update_palette({'darkred': '#990000'})

BTS = {
    "middle": ms.Button.middle,
    "x1": ms.Button.x1,
    "x2": ms.Button.x2
}

class Macro:

    def __init__(self):
        self.key = None
        self.mode = None
        self.state = None
        self.mouse = False
        self.held = False
        self.on = False
        self.last = None
        self.kb_controller = kb.Controller()
        self.ms_controller = ms.Controller()
        self.settings = {
            "state": None,
            "mode": None,
            "key": None,
            "mouse": None
        }

    def use_settings(self):
        if not any(list(self.settings.values())):
            return
        pov = "1st" if self.settings["mode"] == Mode.first else "3rd"
        mode = "Hold" if self.settings["state"] == State.hold else "Toggle"
        key =  self.settings["key"]
        mouse = self.settings["mouse"]
        msg = (f"{cf.green}Last Used Settings:\n{cf.reset}"
               f"Game Mode: {cf.red}{pov} Person{cf.reset}; "
               f"{cf.blue}{mode}{cf.reset}\n"
               f"Key: {cf.yellow}{key.title()}{cf.reset}; "
               f"Mouse: {cf.yellow}{mouse}{cf.reset}\n"
               f"{cf.cyan}Use settings again? {cf.reset}")
        return input(msg).lower().startswith("y")

    def save_settings(self):
        self.settings['mode'] = self.mode
        self.settings['state'] = self.state
        self.settings['key'] = self.key
        self.settings['mouse'] = self.mouse
        pickle.dump(self.settings, open(PARENT/".nuko", "wb"))

    def load_settings(self):
        path = PARENT / ".nuko"
        if path.exists():
            settings = pickle.load(open(path, "rb"))
            self.settings = settings

    def get_mode(self):
        while self.mode is None:
            mode = input("1st or 3rd person?: " )
            if "1" in mode:
                self.mode = Mode.first
            elif "3" in mode:
                self.mode = Mode.third
        while self.state is None:
            state = input("Toggle or Hold? ").lower()
            if state.startswith("t"):
                self.state = State.toggle
            elif state.startswith("h"):
                self.state = State.hold

    def get_start_key(self):
        start_key = input(f"Enter key for macro or {cf.yellow('mouse-middle')}, {cf.yellow('mouse-x1')}, {cf.yellow('mouse-x2')}?: ").lower()
        if start_key == "mouse-middle" or start_key == "middle":
            self.mouse = True
            self.key = "middle"
        elif start_key == "mouse-x1" or start_key == "x1":
            self.mouse = True
            self.key = "x1"
        elif start_key == "mouse-x2" or start_key == "x2":
            self.mouse = True
            self.key = "x2"
        else:
            self.key = start_key

    def show_status(self):
        if self.state == State.hold and self.held:
            state = cf.green("nuko Started")
        elif self.state == State.toggle and self.on:
            state = cf.green("nuko Started")
        else:
            state = cf.red("nuko Stopped")
        t1 = cf.magenta('|')
        t2 = cf.magenta('|\n')
        t3 = cf.magenta('|')
        t4 = cf.magenta('|\n')
        t5 = cf.magenta('|')
        t1 += state + "\n"
        t3 += cf.magenta("Need or want to support? \n")
        t5 += cf.orange("Join the discord!: https://discord.gg/uqyv8NVWMX\n")
        t6 = cf.yellow("made by shelovesaokiYT\n")
        status = t1 + t2 + t3 + t4 + t5 + t6
        if self.last != status:
            self.last = status
            os.system("cls")
            print(status)

    def pressed(self, key):
        if key == kb.KeyCode.from_char(self.key):
            self.held = True
            self.on = not self.on
            self.show_status()

    def released(self, key):
        if key == kb.KeyCode.from_char(self.key):
            self.held = False
            self.show_status()

    def clicked(self, x, y, button, pressed):
        if button == BTS[self.key]:
            if pressed:
                self.held = True
                self.on = not self.on
            else:
                self.held = False
            self.show_status()

    def perform_action(self):
        if self.mode == 3:
            self.kb_controller.press('i')
            time.sleep(0.01)
            self.kb_controller.release('i')
            self.kb_controller.press('o')
            time.sleep(0.01)
            self.kb_controller.release('o')
        elif self.mode == 1:
            self.ms_controller.scroll(0, 1)
            time.sleep(0.01)
            self.ms_controller.scroll(0, -1)
            time.sleep(0.01)

    def start(self):
        with kb.Listener(on_press=self.pressed, on_release=self.released) as listener, ms.Listener(on_click=self.clicked) as mouse_listener:
            while True:
                try:
                    if self.state == State.toggle:
                        if self.on:
                            self.perform_action()
                    else:
                        if self.held:
                            self.perform_action()
                    time.sleep(0.01)
                except:
                    break

    def change_settings(self):
        var = input("Do you want to exit? ")
        if var.lower().startswith("y"):
            return False
        return True

def main():
    macro = Macro()
    macro.load_settings()
    while True:
        if macro.use_settings():
            macro.mode = macro.settings['mode']
            macro.state = macro.settings['state']
            macro.mouse = macro.settings['mouse']
            macro.key = macro.settings['key']
        else:
            macro.get_start_key()
            macro.get_mode()
            macro.save_settings()
        macro.show_status()
        macro.start()
        if not macro.change_settings():
            break

if __name__ == "__main__":
    main()
