import json
import os
from datetime import date

def save_logs(daily_logs, filename='logs.json'):
    # Convert tuples to lists for JSON compatibility
    serializable_logs = {
        date: {
            "food_log": [list(entry) for entry in data["food_log"]],
            "totalCals": data["totalCals"],
            "totalProtein": data["totalProtein"]
        } for date, data in daily_logs.items()
    }
    with open(filename, 'w') as f:
        json.dump(serializable_logs, f)

def load_logs(filename='logs.json'):
    if os.path.exists(filename):
        with open(filename, 'r') as f:
            raw_logs = json.load(f)
        # Convert lists back to tuples
        return {
            date: {
                "food_log": [tuple(entry) for entry in data["food_log"]],
                "totalCals": data["totalCals"],
                "totalProtein": data["totalProtein"]
            } for date, data in raw_logs.items()
        }
    return {}

def calorieTracker():
    daily_logs = load_logs()  # Load saved logs
    today = str(date.today())  # Store today's date as a string

    print('Calorie & Protein Tracker')

    # Prompt user with choices
    while True:
        if today not in daily_logs:  # Create default values for every new day
            daily_logs[today] = {
                "food_log": [],
                "totalCals": 0,
                "totalProtein": 0
            }

        # User choices
        print(f"\nToday's date: {today}")
        print('Choose Next Option')
        print('1) Enter Food')
        print('2) Delete Entry')
        print('3) Display food log and total macros')
        print('4) Change Date')
        print('5) Exit Food Log')

        choice = input('\nEnter Choice: ')  # Store user's choice
        day_log = daily_logs[today]

        if choice == '1':  # Input food & its macros, update list & totals----------------------------
            food = input('Enter food name or enter "back" to go back: ').lower()
            if food == 'back':
                continue

            try:
                cals = float(input('Enter number of calories: '))
                protein = float(input('Enter number of protein: '))
                day_log["food_log"].append((food, cals, protein))
                day_log["totalCals"] += cals
                day_log["totalProtein"] += protein
                print(f"{food} added with {cals} calories & {protein} grams of protein.")
            except ValueError:
                print("Please enter a valid number.")

        elif choice == '2':  # Delete an entry by food name-------------------------------------------
            if not day_log["food_log"]:
                print('The food log is empty.')
                continue

            print('\nFood Log: ')
            for item, cal, protein in day_log["food_log"]:
                print(f"- {item}: {cal} cal, {protein} protein")

            removeChoice = input('Enter name of food to delete or "back" to go back: ').lower()
            if removeChoice == 'back':
                continue

            found = False
            for entry in day_log["food_log"]:
                if entry[0].lower() == removeChoice:
                    day_log["food_log"].remove(entry)
                    day_log["totalCals"] -= entry[1]
                    day_log["totalProtein"] -= entry[2]
                    print(f"{removeChoice} was removed from your log.")
                    found = True
                    break
            if not found:
                print(f"{removeChoice} is not in your log.")

        elif choice == '3':  # Display food log + totals-----------------------------------------------
            print(f"\nToday's Date: {today}")

            if not day_log["food_log"]:
                print('The food log is empty, try another input.')
                continue

            print('\nFood Log: ')
            for item, cal, protein in day_log["food_log"]:
                print(f"- {item}: {cal} cal, {protein} protein")

            print(f"\nTotal calories consumed: {day_log['totalCals']}")
            print(f"Total protein consumed: {day_log['totalProtein']} grams")

        elif choice == '4':  # Change active date------------------------------------------------------
            new_date = input('Enter a date (YYYY-MM-DD) or enter "back" to go back: ')
            if new_date == 'back':
                continue

            try:
                date.fromisoformat(new_date)
                today = new_date
            except ValueError:
                print('Invalid date format. Please use YYYY-MM-DD.')

        elif choice == '5':  # Exit--------------------------------------------------------------------
            print('Goodbye!')
            break

        else:  # Invalidates any other input-----------------------------------------------------------
            print('Invalid input, please try again: ')

    save_logs(daily_logs)  # Save logs on exit

calorieTracker()
