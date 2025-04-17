import time
import json
import os
import random
from datetime import datetime, timedelta

REGISTRATION_FILE = "registered_users.json"
WINNER_FILE = "winner.txt"

REGISTRATION_DURATION = timedelta(hours=1)
EXTENSION_DURATION = timedelta(minutes=30)
DISPLAY_INTERVAL = 10 * 60  # 10 minutes
BACKUP_INTERVAL = 5 * 60  # 5 minutes
MIN_USERS = 5


def load_registered_users():
    if os.path.exists(REGISTRATION_FILE):
        with open(REGISTRATION_FILE, 'r') as f:
            return json.load(f)
    return []


def save_registered_users(users):
    with open(REGISTRATION_FILE, 'w') as f:
        json.dump(users, f)


def select_winner(users):
    winner = random.choice(users)
    with open(WINNER_FILE, 'w') as f:
        f.write(f"🏆 Winner: {winner}\nTotal Participants: {len(users)}")
    print(f"\n🏆 Winner: {winner}\nTotal Participants: {len(users)}")


def display_remaining_time(end_time):
    remaining = end_time - datetime.now()
    minutes = int(remaining.total_seconds() // 60)
    seconds = int(remaining.total_seconds() % 60)
    print(f"⏳ Time Remaining: {minutes}m {seconds}s")


def main():
    print("🎉 Welcome to the Terminal Lottery System!")
    users = load_registered_users()
    start_time = datetime.now()
    end_time = start_time + REGISTRATION_DURATION
    extended = False

    last_display_time = time.time()
    last_backup_time = time.time()

    while datetime.now() < end_time:
        try:
            if time.time() - last_display_time >= DISPLAY_INTERVAL:
                display_remaining_time(end_time)
                last_display_time = time.time()

            if time.time() - last_backup_time >= BACKUP_INTERVAL:
                save_registered_users(users)
                last_backup_time = time.time()

            username = input("Enter a unique username to register: ").strip()

            if not username or not username.isalnum():
                print("❌ Invalid username. Only alphanumeric characters allowed.")
                continue

            if username in users:
                print("⚠️ Username already registered.")
                continue

            users.append(username)
            print(f"✅ {username} registered successfully! Total: {len(users)} users")
        except KeyboardInterrupt:
            print("\n🔁 Program interrupted. Saving progress...")
            save_registered_users(users)
            return

        if datetime.now() >= end_time and not extended and len(users) < MIN_USERS:
            print("📢 Less than 5 users. Extending registration by 30 minutes...")
            end_time += EXTENSION_DURATION
            extended = True

    if not users:
        print("🚫 No users registered. Exiting.")
        return

    save_registered_users(users)
    print("\n🎰 Registration Closed. Drawing a Winner...")
    time.sleep(2)
    select_winner(users)


if __name__ == "__main__":
    main()
# Lottery-system-1
