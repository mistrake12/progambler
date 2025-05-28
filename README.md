# progambler
yes
import random
import time
import json
import os

# Define aura table
auras = [
    {"name": "Misty", "rarity": "Common", "chance": 1/2},
    {"name": "Frost", "rarity": "Uncommon", "chance": 1/10},
    {"name": "Glow", "rarity": "Rare", "chance": 1/50},
    {"name": "Voidlight", "rarity": "Epic", "chance": 1/250},
    {"name": "Radiant Soul", "rarity": "Legendary", "chance": 1/1000},
    {"name": "Eclipse", "rarity": "Mythical", "chance": 1/10000},
    {"name": "Omnisoul", "rarity": "Divine", "chance": 1/100000},
]

auras.sort(key=lambda x: x["chance"])

# Save file paths
stats_file = "stats.json"
leaderboard_file = "leaderboard.json"

# Game state
inventory = {}
roll_count = 0
rarest = None

# Variant chances
SHINY_CHANCE = 1/500
CORRUPTED_CHANCE = 1/1000

def load_stats():
    global inventory, roll_count, rarest
    if os.path.exists(stats_file):
        with open(stats_file, "r") as f:
            data = json.load(f)
            inventory = data.get("inventory", {})
            roll_count = data.get("roll_count", 0)
            rarest = data.get("rarest", None)

def save_stats():
    with open(stats_file, "w") as f:
        json.dump({
            "inventory": inventory,
            "roll_count": roll_count,
            "rarest": rarest
        }, f, indent=2)

def save_leaderboard(name):
    data = {
        "name": name,
        "rolls": roll_count,
        "rarest": rarest or "None"
    }

    leaderboard = []
    if os.path.exists(leaderboard_file):
        with open(leaderboard_file, "r") as f:
            leaderboard = json.load(f)

    leaderboard.append(data)
    leaderboard.sort(key=lambda x: next((i for i, a in enumerate(auras) if a['name'] == x['rarest']), 999))

    with open(leaderboard_file, "w") as f:
        json.dump(leaderboard[:10], f, indent=2)

def roll():
    global roll_count, rarest
    roll_count += 1
    rng = random.random()
    for aura in auras:
        if rng <= aura["chance"]:
            name = aura["name"]
            variant = ""
            if random.random() <= SHINY_CHANCE:
                name = "✨ Shiny " + name
                variant = "Shiny"
            elif random.random() <= CORRUPTED_CHANCE:
                name = "💀 Corrupted " + name
                variant = "Corrupted"

            inventory[name] = inventory.get(name, 0) + 1
            print(f"[{roll_count}] You rolled: {name} ({aura['rarity']}{' - ' + variant if variant else ''})")

            if not rarest or aura["chance"] < next((a['chance'] for a in auras if a['name'] == rarest), 1):
                rarest = aura["name"]
            return
    print(f"[{roll_count}] You rolled... nothing.")

def display_inventory():
    print("\nYour Auras:")
    for name, count in inventory.items():
        print(f"{name}: {count}")
    print(f"\nTotal Rolls: {roll_count}")
    print(f"Rarest Aura: {rarest or 'None'}")

def auto_roll(amount):
    for _ in range(amount):
        roll()
        time.sleep(0.01)

def main():
    load_stats()
    print("🌌 Welcome to Soul’s RNG: Python Edition 🌌\n")

    while True:
        choice = input("\n(R)oll  (A)uto-roll  (I)nventory  (Q)uit\n> ").lower()
        if choice == 'r':
            roll()
        elif choice == 'a':
            try:
                amt = int(input("How many rolls? "))
                auto_roll(amt)
            except:
                print("Invalid input.")
        elif choice == 'i':
            display_inventory()
        elif choice == 'q':
            save_stats()
            display_inventory()
            name = input("Enter your name for the leaderboard: ")
            save_leaderboard(name)
            print("🌟 Your journey has ended. See you again!")
            break
        else:
            print("Unknown command.")

if __name__ == "__main__":
    main()
