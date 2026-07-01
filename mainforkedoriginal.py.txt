import json
from pathlib import Path
from datetime import datetime, timedelta

DATA_FILE = Path.home() / "uta_journal_data.json"
QUESTIONS = [
    "Who am I in this circumstance?",
    "What are my circumstances?",
    "What do I want?",
    "Why do I want it?",
    "When is it?",
    "Where is it?",
    "What must I overcome?",
    "How will I accomplish my objective?",
    "What have I discovered?"
]


def load_data():
    if DATA_FILE.exists():
        try:
            return json.loads(DATA_FILE.read_text())
        except json.JSONDecodeError:
            return {}
    return {}


def save_data(data):
    DATA_FILE.write_text(json.dumps(data, indent=2))


def get_streak(data):
    streak = 0
    current = datetime.now().date()
    while current.strftime("%Y-%m-%d") in data:
        streak += 1
        current -= timedelta(days=1)
    return streak


def journal_today(data):
    today = datetime.now().strftime("%Y-%m-%d")
    if today in data:
        print("\nYou already journaled today.\n")
        return

    answers = {}
    print("\nAnswer the 9 Uta Hagen questions:\n")
    for question in QUESTIONS:
        answers[question] = input(f"{question}\n> ").strip()

    data[today] = {
        "timestamp": datetime.now().isoformat(),
        "answers": answers,
    }
    save_data(data)
    print("\nSaved your entry to ~/uta_journal_data.json\n")


def review_entries(data):
    if not data:
        print("\nNo entries yet.\n")
        return

    print("\nLast entries:\n")
    for date in sorted(data.keys(), reverse=True)[:10]:
        print(date)
        for question, answer in data[date]["answers"].items():
            print(f"- {question}: {answer}")
        print()


def main():
    data = load_data()
    while True:
        print("\nUta Hagen Daily Journal")
        print(f"Streak: {get_streak(data)} days")
        print("1) Journal today")
        print("2) Review past entries")
        print("3) Exit")
        choice = input("> ").strip()

        if choice == "1":
            journal_today(data)
            data = load_data()
        elif choice == "2":
            review_entries(data)
        elif choice == "3":
            break
        else:
            print("Invalid choice. Try again.")


if __name__ == "__main__":
    main()
