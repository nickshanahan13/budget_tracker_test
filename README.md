# Budget Tracker Unittest

```python
#!/usr/bin/env python3


import sys
import unittest
from unittest.mock import patch
from io import StringIO

class BudgetManager:
    def __init__(self, amount):
        self.available = amount
        self.budgets = {}
        self.expenditure = {}

    def add_budget(self, name, amount):
        if name in self.budgets:
            raise ValueError("Budget exists")
        if amount > self.available:
            raise ValueError("Insufficient funds")
        self.budgets[name] = amount
        self.available -= amount
        self.expenditure[name] = []
        return self.available

    def change_budget(self, name, new_amount):
        if name not in self.budgets:
            raise ValueError("Budget does not exist")
        old_amount = self.budgets[name]
        if new_amount > old_amount + self.available:
            raise ValueError("Insufficient funds")
        self.budgets[name] = new_amount
        self.available -= new_amount - old_amount
        return self.available

    def spend(self, name, amount):
        if name not in self.expenditure:
            raise ValueError("No such budget")
        self.expenditure[name].append(amount)
        budgeted = self.budgets[name]
        spent = sum(self.expenditure[name])
        return budgeted - spent

    def print_summary(self):
        print("Budget          Budgeted    Spent       Remaining")
        print("--------------- ---------- ---------- ----------")
        total_budgeted = 0
        total_spent = 0
        total_remaining = 0
        for name in self.budgets:
            budgeted = self.budgets[name]
            spent = sum(self.expenditure[name])
            remaining = budgeted - spent
            print(f"{name:15s} {budgeted:10.2f} {spent:10.2f} {remaining:10.2f}")
            total_budgeted += budgeted
            total_spent += spent
            total_remaining += remaining
        print("--------------- ---------- ---------- ----------")
        print(f'{"Total":15s} {total_budgeted:10.2f} {total_spent:10.2f} {total_budgeted - total_spent:10.2f}')

class TestBudgetManager(unittest.TestCase):

    def setUp(self):
        self.budget_manager = BudgetManager(1000)

    def test_add_budget_success(self):
        remaining = self.budget_manager.add_budget("Groceries", 200)
        self.assertEqual(remaining, 800)
        self.assertEqual(self.budget_manager.budgets["Groceries"], 200)
        self.assertEqual(self.budget_manager.expenditure["Groceries"], [])

    def test_add_budget_budget_exists(self):
        self.budget_manager.add_budget("Groceries", 200)
        with self.assertRaises(ValueError) as context:
            self.budget_manager.add_budget("Groceries", 100)
        self.assertEqual(str(context.exception), "Budget exists")

    def test_add_budget_insufficient_funds(self):
        with self.assertRaises(ValueError) as context:
            self.budget_manager.add_budget("Groceries", 1100)
        self.assertEqual(str(context.exception), "Insufficient funds")

    def test_change_budget_success(self):
        self.budget_manager.add_budget("Groceries", 200)
        remaining = self.budget_manager.change_budget("Groceries", 300)
        self.assertEqual(remaining, 700)
        self.assertEqual(self.budget_manager.budgets["Groceries"], 300)

    def test_change_budget_budget_not_exists(self):
        with self.assertRaises(ValueError) as context:
            self.budget_manager.change_budget("Groceries", 300)
        self.assertEqual(str(context.exception), "Budget does not exist")

    def test_change_budget_insufficient_funds(self):
        self.budget_manager.add_budget("Groceries", 200)
        with self.assertRaises(ValueError) as context:
            self.budget_manager.change_budget("Groceries", 1100)
        self.assertEqual(str(context.exception), "Insufficient funds")

    def test_spend_success(self):
        self.budget_manager.add_budget("Groceries", 200)
        remaining = self.budget_manager.spend("Groceries", 50)
        self.assertEqual(remaining, 150)
        self.assertEqual(self.budget_manager.expenditure["Groceries"], [50])

    def test_spend_budget_not_exists(self):
        with self.assertRaises(ValueError) as context:
            self.budget_manager.spend("Groceries", 50)
        self.assertEqual(str(context.exception), "No such budget")

    def test_print_summary(self):
        self.budget_manager.add_budget("Groceries", 200)
        self.budget_manager.spend("Groceries", 50)
        self.budget_manager.add_budget("Entertainment", 100)
        self.budget_manager.spend("Entertainment", 25)
        expected_output = """Budget          Budgeted    Spent       Remaining
--------------- ---------- ---------- ----------
Groceries       200.00     50.00    150.00
Entertainment   100.00     25.00     75.00
--------------- ---------- ---------- ----------
Total           300.00     75.00    225.00
"""
        with patch('sys.stdout', new_callable=StringIO) as mock_stdout:
            self.budget_manager.print_summary()
            self.assertEqual(mock_stdout.getvalue(), expected_output)

if __name__ == '__main__':
    unittest.main(argv=['first-arg-is-ignored'], exit=False)

    if len(sys.argv) < 2:
        print("Usage: budget_manager.py <initial_amount> [command] [arguments]")
        sys.exit(1)

    try:
        initial_amount = float(sys.argv[1])
        budget_manager = BudgetManager(initial_amount)

        if len(sys.argv) > 2:
            command = sys.argv[2]
            args = sys.argv[3:]

            if command == "add":
                if len(args) != 2:
                    print("Usage: budget_manager.py <initial_amount> add <name> <amount>")
                else:
                    name = args[0]
                    amount = float(args[1])
                    try:
                        remaining = budget_manager.add_budget(name, amount)
                        print(f"Budget '{name}' added. Remaining funds: {remaining:.2f}")
                    except ValueError as e:
                        print(f"Error: {e}")

            elif command == "change":
                if len(args) != 2:
                    print("Usage: budget_manager.py <initial_amount> change <name> <new_amount>")
                else:
                    name = args[0]
                    new_amount = float(args[1])
                    try:
                        remaining = budget_manager.change_budget(name, new_amount)
                        print(f"Budget '{name}' changed. Remaining funds: {remaining:.2f}")
                    except ValueError as e:
                        print(f"Error: {e}")

            elif command == "spend":
                if len(args) != 2:
                    print("Usage: budget_manager.py <initial_amount> spend <name> <amount>")
                else:
                    name = args[0]
                    amount = float(args[1])
                    try:
                        remaining = budget_manager.spend(name, amount)
                        print(f"Spent {amount:.2f} on '{name}'. Remaining budget: {remaining:.2f}")
                    except ValueError as e:
                        print(f"Error: {e}")

            elif command == "summary":
                budget_manager.print_summary()

            else:
                print("Invalid command. Available commands: add, change, spend, summary")

        else:
            print("Initial budget manager created.")
            budget_manager.print_summary()

    except ValueError:
        print("Error: Invalid number format for initial amount or command arguments.")
        sys.exit(1)

```
