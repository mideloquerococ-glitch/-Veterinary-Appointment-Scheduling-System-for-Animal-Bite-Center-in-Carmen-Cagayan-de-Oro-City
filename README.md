# Veterinary Appointment Scheduling System
# for Animal Bite Center in Carmen, Cagayan de Oro City
# Console-based | Python
from datetime import date, timedelta

#  DATA (stored in lists while the program runs) 
customers = []      # each: [username, password, name]
appointments = []   # each: [customer name, pet, doctor, date, time, status]

WEEKS_AHEAD = 4   # how many weeks of future dates can be booked

# each: [doctor name, weekday (0=Monday ... 6=Sunday), [times]]
doctor_schedule = [
    ["Dr. Santos", 0, ["9:00 AM", "10:00 AM", "11:00 AM"]],
    ["Dr. Reyes", 2, ["1:00 PM", "2:00 PM", "3:00 PM"]],
    ["Dr. Cruz", 4, ["10:00 AM", "11:00 AM", "1:00 PM"]],
]


def build_slots(weekday, times):
    """Future dates only: from tomorrow up to WEEKS_AHEAD weeks from today."""
    slots = []
    day = date.today() + timedelta(days=1)
    last_day = date.today() + timedelta(weeks=WEEKS_AHEAD)
    while day <= last_day:
        if day.weekday() == weekday:
            label = day.strftime("%A, %b %d, %Y")
            for t in times:
                slots.append([label, t])
        day += timedelta(days=1)
    return slots


# each: [doctor name, [[date, time], [date, time], ...]]
doctors = [[name, build_slots(weekday, times)]
           for name, weekday, times in doctor_schedule]

pet_types = ["Dog", "Cat", "Other"]

ADMIN_USERNAME = "admin"
ADMIN_PASSWORD = "admin123"


#  INPUT VALIDATION 
def get_text(prompt):
    """Letters, spaces, periods and hyphens only. Numbers are not accepted."""
    while True:
        value = input(prompt).strip()
        if not value:
            print("This field is required.")
        elif any(ch.isdigit() for ch in value) or not all(
                ch.isalpha() or ch in " .-" for ch in value):
            print("Invalid input. Letters only, numbers are not allowed.")
        else:
            return value


def get_username(prompt):
    """Letters and numbers only, no spaces or symbols."""
    while True:
        value = input(prompt).strip()
        if not value:
            print("This field is required.")
        elif not value.isalnum():
            print("Invalid input. Use letters and numbers only.")
        else:
            return value


def get_number(prompt):
    """Whole numbers only. Letters are not accepted."""
    while True:
        value = input(prompt).strip()
        if value.isdigit():
            return int(value)
        print("Invalid input. Numbers only, letters are not allowed.")


#  SHARED FUNCTIONS 
def slot_is_taken(doctor_name, day, time):
    for a in appointments:
        if (a[5] != "Cancelled" and a[2] == doctor_name
                and a[3] == day and a[4] == time):
            return True
    return False


def view_doctor_availability():
    print("\nDOCTOR AVAILABILITY")
    for d in doctors:
        print(d[0])
        for slot in d[1]:
            status = "Taken" if slot_is_taken(d[0], slot[0], slot[1]) else "Available"
            print(f"   {slot[0]}, {slot[1]} - {status}")


def show_appointments(customer_name=None):
    """Show all appointments, or only one customer's. Numbers stay the same."""
    shown = False
    for i, a in enumerate(appointments, start=1):
        if customer_name is not None and a[0] != customer_name:
            continue
        shown = True
        print(f"{i}. Customer: {a[0]} | Pet: {a[1]} | Doctor: {a[2]} | "
              f"Date: {a[3]} | Time: {a[4]} | Status: {a[5]}")
    if not shown:
        print("No appointments.")


def cancel_appointment(customer=None):
    """Customer cancels their own appointment; admin can cancel any."""
    print("\nCANCEL APPOINTMENT")
    name = customer[2] if customer else None
    show_appointments(name)

    mine = [i for i, a in enumerate(appointments, start=1)
            if name is None or a[0] == name]
    if not mine:
        return

    number = get_number("Enter appointment number to cancel: ")
    if number not in mine:
        print("Invalid appointment number.")
    elif appointments[number - 1][5] == "Cancelled":
        print("Appointment is already cancelled.")
    else:
        appointments[number - 1][5] = "Cancelled"
        print("Appointment cancelled.")


#  CUSTOMER 
def register():
    print("\nREGISTER")
    username = get_username("Enter username: ")
    password = input("Enter password: ").strip()
    while not password:
        print("This field is required.")
        password = input("Enter password: ").strip()
    name = get_text("Enter name: ")

    for c in customers:
        if c[0] == username:
            print("Username already exists.")
            return

    customers.append([username, password, name])
    print("Registration successful.")


def customer_login():
    print("\nCUSTOMER LOGIN")
    username = input("Enter username: ").strip()
    password = input("Enter password: ").strip()

    for c in customers:
        if c[0] == username and c[1] == password:
            print("Login successful.")
            customer_menu(c)
            return
    print("Invalid username or password.")


def book_appointment(customer):
    print("\nBOOK APPOINTMENT")

    # 1. choose a doctor
    for i, d in enumerate(doctors, start=1):
        print(f"{i}. {d[0]}")
    number = get_number("Select doctor number: ")
    if not 1 <= number <= len(doctors):
        print("Invalid doctor number.")
        return
    doctor = doctors[number - 1]

    # 2. choose a slot of that doctor
    print(f"\nSLOTS OF {doctor[0].upper()}")
    for i, slot in enumerate(doctor[1], start=1):
        status = "Taken" if slot_is_taken(doctor[0], slot[0], slot[1]) else "Available"
        print(f"{i}. {slot[0]}, {slot[1]} - {status}")
    number = get_number("Select slot number: ")
    if not 1 <= number <= len(doctor[1]):
        print("Invalid slot number.")
        return
    day, time = doctor[1][number - 1]

    if slot_is_taken(doctor[0], day, time):
        print("Time slot is unavailable. Please choose another.")
        return

    # 3. choose the type of pet
    print("\nTYPE OF PET")
    for i, p in enumerate(pet_types, start=1):
        print(f"{i}. {p}")
    number = get_number("Select pet number: ")
    if not 1 <= number <= len(pet_types):
        print("Invalid pet number.")
        return
    pet = pet_types[number - 1]
    if pet == "Other":
        pet = get_text("Enter type of pet: ")

    appointments.append([customer[2], pet, doctor[0], day, time, "Pending"])
    print("Appointment booked successfully.")


def view_my_appointments(customer):
    print("\nMY APPOINTMENTS")
    show_appointments(customer[2])


def customer_menu(customer):
    while True:
        print(f"\nCUSTOMER MENU - {customer[2]}")
        print("\n1. View Doctor Availability", "\n2. Book Appointment", "\n3. View My Appointment", "\n4. Cancel Appointment", "\n5. Logout")
        choice = input("Enter choice: ").strip()

        if choice == "1":
            view_doctor_availability()
        elif choice == "2":
            book_appointment(customer)
        elif choice == "3":
            view_my_appointments(customer)
        elif choice == "4":
            cancel_appointment(customer)
        elif choice == "5":
            print("Logged out.")
            return
        else:
            print("Invalid choice.")


#  ADMIN 
def approve_appointment():
    print("\nAPPROVE APPOINTMENT")
    if not appointments:
        print("No appointments.")
        return

    show_appointments()
    number = get_number("Enter appointment number: ")

    if 1 <= number <= len(appointments):
        if appointments[number - 1][5] != "Pending":
            print("Only Pending appointments can be approved.")
            return
        appointments[number - 1][5] = "Approved"
        print("Appointment approved.")
    else:
        print("Invalid appointment number.")


def admin_menu():
    while True:
        print("\nADMIN MENU")
        print("\n1. View Customer Appointments", "\n2. View Doctor Availability", "\n3. Approve Appointment", "\n4. Cancel Appointment", "\n5. Logout")
        choice = input("Enter choice: ").strip()

        if choice == "1":
            print("\nCUSTOMER APPOINTMENTS")
            show_appointments()
        elif choice == "2":
            view_doctor_availability()
        elif choice == "3":
            approve_appointment()
        elif choice == "4":
            cancel_appointment()
        elif choice == "5":
            print("Logged out.")
            return
        else:
            print("Invalid choice.")


def admin_login():
    print("\nADMIN LOGIN")
    username = input("Enter username: ").strip()
    password = input("Enter password: ").strip()

    if username == ADMIN_USERNAME and password == ADMIN_PASSWORD:
        print("Login successful.")
        admin_menu()
    else:
        print("Invalid username or password.")


#  MAIN MENU 
def main():
    while True:
        print("\n--- VETERINARY APPOINTMENT SCHEDULING SYSTEM ----")
        print("Animal Bite Center - Carmen, Cagayan de Oro City")
        print("\n1. Register", "\n2. Customer Login", "\n3. Admin Login", "\n4. Exit")
        choice = input("Enter choice: ").strip()

        if choice == "1":
            register()
        elif choice == "2":
            customer_login()
        elif choice == "3":
            admin_login()
        elif choice == "4":
            print("Thank you for using the system.")
            break
        else:
            print("Invalid choice.")


main()
