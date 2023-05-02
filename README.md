# La_Spyd3r
# Database_Script_Python
# using sqllit3, classes, csv file
# developed by John Tzolis


from PIL import Image, ImageTk
import sqlite3
import os
import sys
import csv
import re
import time
import datetime as dt
import tkinter as tk


class User():
    '''class to create instance of a user'''

    def __init__(self, fname: str = "", lname: str = "", birthdate: str = "", 
                 address: str = "", email: str = "") -> str:
        
        self.fname = fname
        self.lname = lname
        self.birthday = birthdate
        self.address = address
        self.email = email


    def displayUsers(self):

        message = f"Name: {self.fname} {self.lname}\nBirthdate: {self.birthday}\n"+\
        f"Address: {self.address}\nEmail: {self.email}"

        print(message)


    def displayNames(self):

        message = f"Name: {self.fname} {self.lname}"

        return message



class Income(User):

    total = 0

    def __init__(self, fname, lname, salary: int = 0, 
                 savings: int = 0, ssi: str = ""):
        
        super().__init__(fname=fname, lname=lname)
        self.salary = salary
        self.savings = savings
        self.ssi = ssi


    def __call__(self, value=None, *args, **kwds) -> str: # TODO
        return super().__call__(value, *args, **kwds)
    

    def displayUsers(self):

        message = self.displayNames() + "\nSalary: ${:,.0f}\n".format(self.salary)+\
        "Savings: ${:,.0f}\n".format(self.savings) + f"Social Security Number: {self.ssi}"

        print(message)


    @classmethod
    def increase_salary(cls, value, salary): # TODO
        '''User salary increased'''

        cls.total = salary + value

        return cls.total



# regex patterns
fname_pattern = "[A-Z][a-z]+$"                                      # first name regex
lname_pattern = "[A-Z][a-z]+$"                                      # last name regex

bday_pattern = '''(([0]?[1-9]|[1][0-2])
/
([0]?[1-9]|[1-2][0-9]|[3][0-1])
/
(19[0-9][0-9]|2[0-1][0-9][0-9]))'''  # complex MM/DD/YYYY birthdate regex

#bday_pattern = "(\d{1,2}/\d{1,2}/(19[0-9][0-9]|2[0-1][0-9][0-9]))"  # MM/DD/YYYY birthdate regex
address_pattern = "\d+\s\w+\s\w+\.?"                                # home address regex
email_pattern = "\w+\d*_?\w*\d*@\w+\.?\w*\.(com|edu|net|gov)"       # email address regex
ssi_pattern = "((\d{3}[-\s]\d{2}[-\s]\d{4})|(\d{9}))"               # ssi regex



def regex_check(table, fname, 
                lname, var_1, 
                var_2, var_3):
    
    '''Regex user check'''

    date_obj = dt.datetime.today()
    

    if (re.match(fname_pattern, fname)):
        pass
    else:
        return False, 1

    if(re.match(lname_pattern, lname)):
        pass
    else:
        return False, 2
    

    if(table == "1"):  
        
        if(re.match(bday_pattern, var_1) and int(var_1[-4]) < int(date_obj.year)):
            pass
        else:
            return False, 3
        
        if(re.match(address_pattern, var_2)):
            pass
        else:
            return False, 4

        if(re.match(email_pattern, var_3)):
            pass
        else:
            return False, 5
        

    if(table == "2"):
        
        if(var_1.isdigit()):# TODO
            pass
        else:
            return False, 3
        
        if(var_2.isdigit()):# TODO
            pass
        else:
            return False, 4
        
        if(re.match(ssi_pattern, var_3)):
            pass
        else:
            return False, 5

    return True, 0


    
def csv_writer(conn):
    '''writes info to csv file''' 

    csv_file = "SpyderDatabaseCSV.csv"

    # select all user data in the table
    cursor = conn.execute("SELECT * FROM users")

    with open(csv_file, mode="w", newline="") as csvfileObj:
        writer = csv.writer(csvfileObj)

        for row in cursor:
    
            writer.writerow([row[0], row[1], row[2], row[3], row[4], row[5]])


    print("\n[+] SQL Database successfully saved to CSV file!\n")



def max_length(conn, table): # finds and returns max length from name field

    '''newlist = [x if x != "banana" else "orange" for x in fruits]
    x = lambda a, b, c: a + b + c
    print(x(5, 5, 5))'''

    max = 0

    sql_name = f"SELECT * FROM pragma_table_info('{table}');" # selects field name
    cursor2 = conn.cursor()
    data_type2 = cursor2.execute(sql_name)
    list_name = data_type2.fetchall()

    #max = [f for f in data_type2 if len(f) > max][0]

    for x in list_name:

        if(len(x[1]) > max):
            max = len(x[1])

    return max



##################################################
####---       Connect to SQL Database      ---####

def sql_database():
    
    database_var = "SpyderDatabaseJT.db"

    # connect to the database (or create it if it doesn't exist)
    conn = sqlite3.connect(database_var)

    print(f"[+] Successfull connection to '{database_var}' database.")

    # commit the changes
    conn.commit()

    return conn

####---    End Connect to SQL Database     ---####
##################################################


def delete_sql_table(conn):
    '''delete table from database'''

    table = ""

    sql_table_list = display_sql_tables(conn=conn)

    print("\n[+]--- Delete table ---[+]\nChoose from options:")

    for e, t in enumerate(sql_table_list, 1):
        print(f"{e}) - {t}")

    index = input()

    for e, t in enumerate(sql_table_list, 1):
        if(index == str(e)):

            table = t

            sql_query = f"""drop table if exists {table};"""
            conn.execute(sql_query)
            conn.commit()

            print(f"\n[+] Table '{table}' successfully deleted!\n")

            break



#####################################################
####---           Display SQL Tables          ---####

def display_sql_tables(conn):

    '''display sql tables in database'''

    list_tables = []
    num_tables = 0

    sql_query = "SELECT name FROM sqlite_master WHERE type='table';"

    cursor = conn.cursor()
    cursor.execute(sql_query)

    temp = cursor.fetchall()

    conn.commit()

    for enum, table in enumerate(temp, 1):
        print(f"{enum})- {table[0]}")
        list_tables.append(table[0])

        num_tables += 1

    print(f"\nTotal tables: '{num_tables}'\n")

    return list_tables

####---       End of Display SQL Tables       ---####
#####################################################



def sql_table(conn): # static tables

    # create a users table
    conn.execute('''CREATE TABLE IF NOT EXISTS users
                    (id INTEGER PRIMARY KEY,
                    fname TEXT NOT NULL,
                    lname TEXT NOT NULL,
                    birthdate TEXT NOT NULL,
                    address TEXT NOT NULL,
                    email TEXT NOT NULL)''')
    
    # commit the changes
    conn.commit()

    # create a user income table
    conn.execute('''CREATE TABLE IF NOT EXISTS users_income
                    (id INTEGER PRIMARY KEY,
                    fname TEXT NOT NULL,
                    lname TEXT NOT NULL,
                    salary INTEGER,
                    savings INTEGER,
                    ssi TEXT NOT NULL)''')
    
    # commit the changes
    conn.commit()



def sql_create_user_table(conn, table, 
                          user_value_list, user_type_list):
    
    '''creates table from user input'''

    sql_item = []

    for value, type in zip(user_value_list, user_type_list):

        if type == "integer" or type == "INTEGER":
            sql_item.append(value + " INTEGER")


        elif type == "string" or type == "STRING":
            sql_item.append(value + " TEXT NOT NULL")
        
        
        elif type == "float" or type == "FLOAT":
            sql_item.append(value + " FLOAT")


        else:
            print("Unkown value type.")



    sql_joined = ", ".join(sql_item)

    sql_command = f"CREATE TABLE IF NOT EXISTS {table} (id INTEGER PRIMARY KEY, {sql_joined});"

    print()
    print(sql_command)
    print(f"\n[+] '{table}' table successfully created.\n")
    
    # create a user income table
    conn.execute(sql_command)
    
    # commit the changes
    conn.commit()



###------ SQL Data Insert Section ------###
###########################################

def uni_sql_insert_table(conn):
    '''Universal insert data table'''

    os.system("cls")

    print("\n[+]--- Insert Record into Table ---[+]")

    list_table = display_sql_tables(conn=conn)

    user_input = ""

    while(True):

        print("Pick a table to insert data or 'e' to exit:")

        for e, i in enumerate(list_table, 1):
            print(f"{e}) - {i}")

        user_input = input().lower().strip()

        if(user_input.isdigit()):
            
            if(int(user_input) > 0 and int(user_input) <= len(list_table)):
               break
            else:
                print("Out of range.")

        elif(user_input == "e"):
            print("\n[-] Exiting..")
            return

        else:
            print("\n[-] Please enter an integer.\n")


    sql = ""
    table = ""
    flag = False

    

    for e, i in enumerate(list_table, 1):

        if(int(user_input) == e):
            table = i    
            flag = True

    sql_type = f"SELECT type FROM pragma_table_info('{table}')" # selects datatype from field name

    if(flag):

        list_items = []
        list_value = []
        list_type = []

        sql = f"INSERT INTO {table} " 

        cursor = conn.cursor()
        cursor2 = conn.cursor()
        

        print(f"\nColumns in '{table}' table")
        fields = cursor.execute(f"select * from {table}")
        data_type = cursor2.execute(sql_type)

        sql += "("

        enum = 1

        for column, type in zip(fields.description, data_type):

            if(column[0] == "id"):
                continue

            print(f"{enum}) - Column '{column[0]}' with Datatype '{type[0]}'")
            sql += f"{column[0]}"

            list_items.append(column[0])
            list_type.append(type)

            if(enum == len(fields.description) - 1):
                sql += ") "

            else:
                sql += ", "

            enum += 1


        sql += f" VALUES ("

        for e, i in enumerate(list_items):

            print(f"Insert data into '{i}':")
            value = input().strip().strip("$").replace(",", "")

            if(value.lower() == "q"):
                return
            
         
            elif(value.isalpha() and list_type[e][0] == "string" or list_type[e][0] == "STRING" or list_type[e][0] == "TEXT"): # TODO

                sql += "?"

                list_value.append(value)

                if(e == len(list_items) - 1):
                    sql += ")"

                else:
                    sql += ","

            elif(value.isdigit() and list_type[e][0] == "integer" or list_type[e][0] == "INTEGER"):
                
                sql += "?"

                list_value.append(value)

                if(e == len(list_items) - 1):
                    sql += ")"

                else:
                    sql += ","
                
            else:
                print("<--Error-->")


        #print(sql)
        
        user = tuple(list_value)
        
        cursor.execute(sql, user)

        # commit the changes
        conn.commit()
        
        print(f"\n[+] Value successfully inserted into '{table}' table.\n")
        

    else:
        print("\n[-] Table does not exist.\n")



def example_pragma_table_info(conn): # Displays data type for each columns


    print("\n[+]--- Display Table Datatype ---[+]\nChoose from tables below: ")
    table_list = display_sql_tables(conn=conn)

    option = input()


    for enum, table in enumerate(table_list, 1):

        if(str(enum) == option):
            
            sql_type = f"SELECT name, type FROM pragma_table_info('{table}');" # selects datatype from field name
            cursor = conn.cursor()
            data_type = cursor.execute(sql_type)

            max = max_length(conn=conn, table=table) # returns max len value from table name

            for e, i in enumerate(data_type, 1):

                print(e, i[0].ljust(max + 1),":".ljust(5), i[1])

            conn.commit()
            print()

            return
        
    else:
        print("\n[-] Table does not exist.\n")



def sql_insert_table(conn, userObj, # Obsolete code
                     table, var_1, 
                     var_2, var_3, 
                     var_4, var_5,
                     option):
    
    '''set user information into database'''

    # insert some data
    sql = f'''INSERT INTO {table} ({var_1}, {var_2}, {var_3}, {var_4}, {var_5})
    VALUES (?,?,?,?,?)'''

    if(option == "1"):
        user = (userObj.fname, userObj.lname, userObj.birthday, 
            userObj.address, userObj.email)
        
    elif(option == "2"):
        user = (userObj.fname, userObj.lname, userObj.salary, 
            userObj.savings, userObj.ssi)
    

    cur = conn.cursor()

    cur.execute(sql, user)

    # commit the changes
    conn.commit()

##################################################
###------ End of SQL Insert Data Section ------###



def sql_get_users_record(conn, option, table):
    '''select a user to print'''

    id_num = ""
    fname = ""
    lname = ""
    row2 = ["Record", ""]
    choice = ""

    print("\n[+]--- Retrieve Data ---[+]")

    cursor = conn.execute(f"SELECT COUNT(*) AS CNTREC FROM pragma_table_info('{table}') WHERE name='name';")
    cursor2 = conn.execute(f"SELECT COUNT(*) AS CNTREC FROM pragma_table_info('{table}') WHERE name='id';")

    check = cursor.fetchone()
    check2 = cursor2.fetchone()

    if(check[0] == 1 and check2[0] == 1):

        print("[+] Search record by name or id?\n1) - ID number\n2) - Name")
        choice = input().strip()

    else:

        choice = "1"
        
        
    if(choice == "1"):


        print("Select ID Number: ")
        id_num = input().strip()

        # select and print data for a specific user
        cursor = conn.execute(f"SELECT * FROM {table} WHERE id=?", (id_num,))
        row = cursor.fetchone()

        try: # TODO

            cursor2 = conn.execute(f"SELECT fname, lname FROM {table} WHERE id=?", (id_num,))
            row2 = cursor2.fetchone()

        except Exception as e:
            print("[-] " + str(e))


    elif(choice == "2"):
   
        cursor = conn.execute(f"SELECT COUNT(*) AS CNTREC FROM pragma_table_info('{table}') WHERE name='fname';")
        cursor2 = conn.execute(f"SELECT COUNT(*) AS CNTREC FROM pragma_table_info('{table}') WHERE name='lname';")

        check = cursor.fetchone()
        check2 = cursor2.fetchone()


        if(check[0] == 1 and check2[0] == 1):

            print("Enter first name: ")
            fname = input().strip()

            print("Enter last name: ")
            lname = input().strip()

            # select and print data for a specific user
            try:

                cursor = conn.execute(f"SELECT * FROM {table} WHERE fname=? AND lname=?", (fname, lname,))
                row = cursor.fetchone()

            except Exception:
                print(f"[-] 'Name' column not available for search in '{table}' table.\n\tTry ID number instead.")
                return

        else:

            print("Enter Name: ")
            name = input().strip()

            # select and print data for a specific user
            try:

                cursor = conn.execute(f"SELECT * FROM {table} WHERE name=?", (name,))
                row = cursor.fetchone()

            except Exception:
                print(f"[-] 'Name' column not available for search in '{table}' table.\n\tTry ID number instead.")
                return

    else:
        print("\n[-] Exiting..")
        return


    print()
    print(f"[+] Following record in '{table}' table:")


    if(row):


        if(fname and lname):

            message = "[+] Info for '{0} {1}' printed below:".format(fname, lname)
            len_num = len(message)

            print(message)
            print("-" * len_num)


        elif(id_num):

            message = "[+] Info for '{0} {1}' printed below:".format(*row2)
            len_num = len(message)

            print(message)
            print("-" * len_num)


        elif(name):
            
            message = "[+] Info for '{0}' printed below:".format(row[1])
            len_num = len(message)

            print(message)
            print("-" * len_num)



        if(option == "1"):

            userObj = User(row[1], row[2], row[3], row[4], row[5])
            userObj.displayUsers()

        elif(option == "2"):

            userObj = Income(row[1], row[2], row[3], row[4], row[5])
            userObj.displayUsers()

        else:

            for i in row:
                print(str(i) + ", ", end="")

            print("\n")


    else:
        print(f"\n[-] Record does not exist in '{table}' table!\n")

    # commit the changes
    conn.commit()



###-------- SQL Table section --------###
#########################################

def sql_get_table(conn, table):
    '''Print table'''

    print(f"\n[+] Following records in '{table}' table:\n")

    # select and print all data in the table
    cursor = conn.execute(f"SELECT * FROM {table};")

    for row in cursor:
        for item in row:
            print(str(item) + ", ", end="")

        print()

    cursor = conn.execute(f"SELECT * FROM {table}")
    
    print("\n'{0}' record(s) in '{1}' table.".format(len(cursor.fetchall()), table))
    print()

    # commit the changes
    conn.commit()

#############################################
###------- End of SQL table section ------###



###------- SQL Update table section ------###
#############################################

def sql_update_table(conn, table, var_3, var_4, var_5, 
                     regex_pattern_3, regex_pattern_4, regex_patern_5): # TODO
    '''Update info'''

    print("Enter First Name: ")
    fname = input().strip()

    print("Enter Last Name: ")
    lname = input().strip()

    cursor = conn.execute(f"SELECT * FROM {table} WHERE fname=? AND lname=?", (fname, lname,))

    if(cursor.fetchone()):

        while True:

            # update data for a specific user  
            print(f"\n-- Info to update:\n1) - First Name\n2) - Last Name"\
                  f"\n3) - {var_3.title()}\n4) - {var_4.title()}\n5) - {var_5.title()}\n6) - Exit")
            
            option = input().strip()

            if(option == "1"):

                print("Replace 'First Name' with: ")
                replace_value = input().title().strip()

                if(re.fullmatch(fname_pattern, replace_value)):

                    conn.execute(f"UPDATE {table} SET fname=? WHERE fname=? AND lname=?", 
                                 (replace_value, fname, lname,)) # updates first name
                    
                    conn.commit()

                    print("\n[+] Update Successfull!")
                    break

                else:
                    print("\n[-] 'First Name': '{0}' is wrong format!".format(replace_value))


            elif(option == "2"):

                print("Replace 'Last Name' with: ")
                replace_value = input().title().strip()

                if(re.fullmatch(lname_pattern, replace_value)):

                    conn.execute(f"UPDATE {table} SET lname=? WHERE fname=? AND lname=?", 
                                 (replace_value, fname, lname,)) # updates last name
                    
                    conn.commit()

                    print("\n[+] Update Successfull!")
                    break

                else:
                    print("\n[-] 'Last Name': '{0}' is wrong format!".format(replace_value))


            elif(option == "3"):

                print(f"Replace '{var_3.title()}' with: ")
                replace_value = input().strip()

                if(re.fullmatch(regex_pattern_3, replace_value) or regex_pattern_3 == "pass" and replace_value.isdigit()):

                    conn.execute(f"UPDATE {table} SET {var_3}=? WHERE fname=? AND lname=?", 
                                 (replace_value, fname, lname,)) # updates bday
                    
                    conn.commit()

                    print("\n[+] Update Successfull!")
                    break

                else:
                    print("\n[-] '{0}': '{1}' is wrong format!".format(var_3.title(), replace_value))


            elif(option == "4"):

                print(f"Replace '{var_4.title()}' with: ")
                replace_value = input().strip()

                if(re.fullmatch(regex_pattern_4, replace_value) or regex_pattern_4 == "pass" and replace_value.isdigit()):

                    conn.execute(f"UPDATE {table} SET {var_4}=? WHERE fname=? AND lname=?", 
                                 (replace_value, fname, lname,)) # updates home address
                    
                    conn.commit()

                    print("\n[+] Update Successfull!")
                    break

                else:
                    print("\n[-] '{0}': '{1}' is wrong format!".format(var_4.title(), replace_value))


            elif(option == "5"):

                print(f"Replace '{var_5.title()}' with: ")
                replace_value = input().strip()

                if(re.fullmatch(regex_patern_5, replace_value)):

                    conn.execute(f"UPDATE {table} SET {var_5}=? WHERE fname=? AND lname=?", 
                                 (replace_value, fname, lname,)) # updates email address
                    
                    conn.commit()

                    print("\n[+] Update Successfull!")
                    break

                else:
                    print("\n[-] '{0}': '{1}' is wrong format!".format(var_5.title(), replace_value))


            elif(option == "6"):

                print("\n[-] Exiting..")
                break
            
            else:
                print("\n[-] Please choose from options!")

    else:
        print(f"\n[-] User '{fname} {lname}' does not exist!")

####################################################
###------- End of SQL Update table section ------###



###------- SQL Delete Record Section -------###
#############################################

def sql_delete_user(conn, table) -> None: # TODO
    '''Delete a specific user'''

    os.system("cls")

    print("[+]-- Delete User Income --[+]")

    print("Enter first name: ")
    fname = input().strip().title()

    print("Enter last name: ")
    lname = input().strip().title()


    cursor = conn.execute(f"SELECT * FROM {table} WHERE fname=? AND lname=?", (fname, lname,))

    if(cursor.fetchone()):

        print("Delete '{0} {1}' from '{2}' table? (y/n):".format(fname, lname, table))
        option = input().strip().lower()

        if(option == "y"):
        
            # delete data for a specific user
            conn.execute(f"DELETE FROM {table} WHERE fname=? AND lname=?", (fname, lname,))
            print("\n[+] User '{0} {1}' from '{2}' table successfully deleted!".format(fname, lname, table))

        else:
            print("\n[-] Returning to main menu.")

    else:
        print("\n[-] User '{0} {1}' does not exist!".format(fname, lname))

    conn.commit()

###################################################
###------ End of SQL Delete Record Sction ------###



###--------  SQL Check Record Exists Section   ---------###
###########################################################

def sql_check_record_exists(conn, table, fname = "", lname = ""):
    '''Check if record exists'''

    if(fname == "" and lname == ""):

        print("\n[+]-- Check Record Exists --[+]")

        print("Enter first name: ")
        fname = input().strip()

        print("Enter last name: ")
        lname = input().strip()
        

    user_exist = True

    cursor = conn.execute(f"SELECT * FROM {table} WHERE fname=? AND lname=?", (fname, lname,))

    if(cursor.fetchone()):

        print(f"\n[-] '{lname}, {fname}' exist in '{table}' table!")
        user_exist = False

    else:

        print(f"\n[+] '{lname}, {fname}' not in '{table}' table!")


    conn.commit()
    print()

    return user_exist

###########################################################
###------ End of SQL Check Record Exists Section -------###



def createDirectory():
    '''Create/change directory'''

    root_to_user = os.path.expanduser("~")

    directory_name = "SpyderFolder"
    onedrive = ""
    #onedrive = "OneDrive"         #<=== un-comment if user has onedrive

    print("Current Directory: ", end="")
    print(os.getcwd())

    path = os.path.join(root_to_user, onedrive, "Desktop")
    os.chdir(path)

    if os.path.isdir(directory_name):

        pass

    else:

        desk_top = "Desktop"
        print("[+] '\{0}' directory created in '\{1}' directory.".format(directory_name, desk_top))
        os.mkdir(directory_name)


    try:

        path = os.path.join(os.curdir, directory_name)
        os.chdir(path)

    except Exception as e:
        print("[-] Un-comment OneDrive for script to run!\n", e)
        sys.exit(0)


    print("Changed directory to: \\", end="")
    print(os.path.basename(os.getcwd()))



def calculation_menu(conn): # TODO
    '''calculate info menu'''

    print("[+] Choose from options:\n1) - Add to user salary\t2)")
    option = input().strip()

    if(option == "1"):

        print("[+] ")
        total = Income.increase_salary(value=value, salary=salary)
        print(total)

    elif(option == "2"):
        pass


def user_interface_tk(root, tk): # TODO

    '''Function to create user interface'''

    text="[+] Spyder SQL Datebase [+]"

    HEIGHT = 800
    WIDTH = 900

    canvas = tk.Canvas(root, width=WIDTH, height=HEIGHT)
    canvas.pack()

    header = tk.Label(text=text, 
                      fg="black",                    
                       width=60, 
                       height=10,
                       font=40,
                       )
    header.pack()

    frame = tk.Frame(root, bg='black', bd=2)
    frame.place(relx=0.5, rely=0.1, relwidth=0.75, relheight=0.1, anchor='n')

    label = tk.Label(frame)
    label.place(relwidth=1, relheight=1)

    entry = tk.Entry(frame, font=40)
    entry.place(relwidth=0.65, relheight=1)
    #name = entry.get() # recieve user input

    button = tk.Button(frame, text="Enter", font=40)
    button.place(relx=0.7, relheight=1, relwidth=0.3)

    lower_frame = tk.Frame(root, bg="black", bd=2)
    lower_frame.place(relx=0.5, rely=0.25, relwidth=0.75, relheight=0.6, anchor='n')

    label = tk.Label(lower_frame)
    label.place(relwidth=1, relheight=1)

    # Youtube video: Create a GUI app with Tkinter - Step by Step Tutorial
    # https://realpython.com/python-gui-tkinter/

    root.mainloop() 
    sys.exit(0)



####################################
###-------- Program Start -------###
####################################

# creates folder and opens database and opens user interface
createDirectory()
conn = sql_database()
root = tk.Tk()
user_interface_tk(root=root, tk=tk)

main_flag = True
print()


while main_flag:

    # start of program
    print("[+] Spyder SQL Database [+]".center(50))
    print(("~"*50).center(50))

    print("[+] Please choose from options:\n1) - Enter record\t2) - Display records\n3) - Update record"+\
    "\t4) - Retrieve user record\n5) - Delete record\t6) - Save SQL data to CSV file\n7) - Record exists"+\
    "\t8) - Create, Delete or Display SQL table\n9) - Insert record\t99) - Exit")

    # sql database tables
    table_users = "users"
    table_income = "users_income"


    option = input("root@kali> ").strip() 

    if option == "1":

        os.system("cls")

        sql_list_table = display_sql_tables(conn=conn)

        print("\n[+] Choose table to insert record [+]")

        for enum, table in enumerate(sql_list_table, 1):
            print(f"{enum}) - {table} Table")

        print("or 'q' to exit.")
        
        option = input("one@kali> ")

        if(option == "q"):

            print("\n[-] Exiting..")
            continue

        
        print("\nEnter user first name: ")
        fname = input().strip()

        print("Enter user last name: ")
        lname = input().strip()


        if(option == "1"):
            
            print("Enter user birth date: ")
            b_date = input().strip()

            print("Enter user home address: ")
            address = input().strip()

            print("Enter user email address: ")
            email_add = input().strip()


            # code validation check for existing users in database for no doubles
            bool, num = regex_check(option, fname=fname, 
                                    lname=lname, var_1=b_date, 
                                    var_2=address, var_3=email_add)
            
            

            if(num == 0):

                # check if name already exists in table
                bool = sql_check_record_exists(conn=conn, table=table_users, 
                                               fname=fname, lname=lname)


            if bool and num == 0:

                print("\nFollowing information has been entered:")
                print("---------------------------------------")

                userObj = User(fname=fname, 
                               lname=lname, birthdate=b_date, 
                               address=address, email=email_add)
                
                userObj.displayUsers()

                sql_table(conn=conn)

                print()

                # sql field names
                var_1 = "fname"; var_2 = "lname"; var_3 = "birthdate" 
                var_4 = "address"; var_5 = "email"

                sql_insert_table(conn=conn, userObj=userObj, 
                                 table=table_users, var_1=var_1, 
                                 var_2=var_2, var_3=var_3, 
                                 var_4=var_4, var_5=var_5, 
                                 option=option)


            else:

                if(num == 0):
                    value = "[-] Record Exists."

                elif(num == 1):
                    value = fname
                    format = "First Name"
                    example = "Format: Jane"

                elif(num == 2):
                    value = lname
                    format = "Last Name"
                    example = "Format: Doe"

                elif(num == 3):
                    value = b_date
                    format = "Birthdate"
                    example = "Year MUST be sub current year and formatted: 'MM/DD/YYYY'"

                elif(num == 4):
                    value = address
                    format = "Home Address"
                    example = "Format: 123 Home Drive OR Dr. | 1234 Home Pass"

                elif(num == 5):
                    value = email_add
                    format = "Email Address"
                    example = "Format: emailformat@whatever.(com|net|edu|gov)"

                else:
                    value = "Unknown"


                if(num == 0):
                    print(value)

                else:
                    print(f"\n[-] '{value}' incorrect format for '{format}'\
                          \n\r\tExample: '{example}'")
                    
                
                print("[-] Please try again.\n")
                continue

            

        if(option == "2"):

            print("Enter user SSN: ")
            ssi = input().strip()

            print("Enter user salary: ")
            salary = input().strip().strip("$").replace(",", "")

            print("Enter user savings: ")
            savings = input().strip().strip("$").replace(",", "")

            # code validation check for existing users in database for no doubles
            bool, num = regex_check(option, fname=fname, 
                                    lname=lname, var_1=salary, 
                                    var_2=savings, var_3=ssi)
            
            if(num == 0):

                # check if name already exists in table
                bool = sql_check_record_exists(conn=conn, table=table_users, 
                                               fname=fname, lname=lname)


            if bool and num == 0:

                print("\nFollowing information has been entered:")
                print("---------------------------------------")

                userObj = Income(fname=fname, 
                                 lname=lname, salary=int(salary), 
                                 savings=int(savings), ssi=ssi)
                
                userObj.displayUsers()

                sql_table(conn=conn)

                print()

                # sql field names
                var_1 = "fname"; var_2 = "lname"; var_3 = "salary"
                var_4 = "savings"; var_5 = "ssi"

                sql_insert_table(conn=conn, userObj=userObj, 
                                 table=table_income, var_1=var_1, 
                                 var_2=var_2, var_3=var_3, 
                                 var_4=var_4, var_5=var_5,
                                 option=option)
                

            else:

                if(num == 0):
                    value = "[-] Record Exists."

                elif(num == 1):
                    value = fname
                    format = "First Name"
                    example = "Format: Jane"

                elif(num == 2):
                    value = lname
                    format = "Last Name"
                    example = "Format: Doe"

                elif(num == 3):
                    value = salary
                    format = "Salary"
                    example = "Must be an integer! '1-9'"

                elif(num == 4):
                    value = savings
                    format = "Savings"
                    example = "Must be an integer! '1-9'"

                elif(num == 5):
                    value = ssi
                    format = "Social Security Number"
                    example = "MUST have 9 integers and formatted: 123-45-6789 | 123456789"
                    
                else:
                    value = "Unknown"


                if(num == 0):
                    print(value)

                else:
                    print(f"\n[-] '{value}' incorrect format for '{format}'\
                      \n\r\tExample: '{example}'")
                    
                
                print("[-] Please try again.")
                continue


        else:
            print("\n[-] Select universal \"9) - Insert record\" option to access other tables!\
                  \n\tThis option is obsolete!")
            continue


    elif option == "2":
        
        os.system("cls")

        print("\n[+] Choose table to display records [+]")
        
        table_list = display_sql_tables(conn=conn)

        option = input("two@kali> ")
        
        print()

        if(option.isdigit()):

            for e, table in enumerate(table_list, 1):
            
                if(int(option) == e):
                    sql_get_table(conn=conn, table=table)
                
        else:
            print("\n[-] Exiting..\n")   
            continue
            

       

    elif option == "3":

        os.system("cls")

        print("\n[+] Choose table to update record [+]\n1) - Users table"\
              "\n2) - Users Income table\n3) - Exit")
        
        option = input("three@kali> ")


        print()

        if(option == "1"):

            var_3 = "birthdate"; var_4 = "address"; var_5 = "email"

            sql_update_table(conn=conn, table=table_users, 
                             var_3=var_3, var_4=var_4, var_5=var_5, 
                             regex_pattern_3=bday_pattern, 
                             regex_pattern_4=address_pattern, 
                             regex_patern_5=email_pattern)
            

        elif(option == "2"):

            var_3 = "salary"; var_4 = "savings"; var_5 = "ssi"

            sql_update_table(conn=conn, table=table_income,
                            var_3=var_3, var_4=var_4, var_5=var_5,
                            regex_pattern_3="pass", 
                            regex_pattern_4="pass", 
                            regex_patern_5=ssi_pattern)

        else:
            print("\n[-] Exiting..\n")
            continue
        
        
    elif option == "4":

        os.system("cls")

        print("\n[+] Choose table to retrieve user record [+]")
        
        table_list = display_sql_tables(conn=conn)
        
        option = input("four@kali> ")

        for enum, table in enumerate(table_list, 1):

            if(option == str(enum)):

                sql_get_users_record(conn=conn, option=option, table=table)
        print()

    elif option == "5":

        print("\n[+] Choose table to delete record [+]\n1) - Users table"\
              "\n2) - Users Income table\n3) - Exit")
        
        option = input("five@kali> ")


        print()

        if(option == "1"):

            sql_delete_user(conn=conn, table=table_users)

        elif(option == "2"):

            sql_delete_user(conn=conn, table=table_income)

        else:
            print("[-] Exiting..\n")
            continue


    elif option == "6":
        print()
        csv_writer(conn=conn)


    elif option == "7": # checks for user in static table
        
        os.system("cls")

        print("\n[+] Choose table to check record exists [+]\n1) - Users table\
              \n2) - Users Income table\n3) - Exit")
        
        option = input("seven@kali> ")


        print()

        if(option == "1"):

            sql_check_record_exists(conn=conn, table=table_users)

        elif(option == "2"): # checks if user exists in table

            sql_check_record_exists(conn=conn, table=table_income)

        else:
            print("[-] Exiting..\n")
            continue


    elif option == "8": # Create, display or delete sql table

        flag = True

        os.system("cls")

        print("\n[+]--- Create, Delete or Display Table ---[+]")
        print("1) - Create Table\n2) - Delete Table\n3) - Display Tables\n4) - Exit")

        option = input()

        if(option == "1"):

            list_value = []
            list_type = []

            print("\n[+]--- Create Table ---[+]")

            while(flag):

                print("Name of table: ")
                new_table = input().lower().strip()

                print(f"\n-- Is this correct? (y/n): '{new_table}'")
                choice = input().lower().strip()

                if(choice == "y"):
                    flag = False


            flag = True

            while(flag):

                print("\nName of table field: ")
                value = input().lower().strip()

                print("What is field data type?: \n1) - INTEGER\n2) - TEXT\3) - REAL\n4) - NULL\nBLOB")
                type = input().lower().strip()

                if(value == "q"):
                    break

                if(type == "1" or type == "INTEGER"):
                    
                    type = "INTEGER"
                 

                elif(type == "2" or type == "TEXT"):

                    type = "TEXT"
                    

                elif(type == "3" or type == "REAL"):

                    type = "REAL"


                elif(type == "4" or type == "NULL"):

                    type = "NULL"
                    

                elif(type == "3" or type == "BLOB"):

                    type = "BLOB"
                    

                print(f"\n-- Is this correct? (y/n):\nField: '{value}'\nType: '{type}'")
                choice = input().lower().strip()


                if(choice == "y"):

                    list_value.append(value)
                    list_type.append(type)

                    print("Continue? (y/n): ")
                    choice = input().lower().strip()

                    if(choice == "n"):
                        flag = False
                      

            if(flag == False):

                sql_create_user_table(conn=conn, table=new_table, 
                                user_value_list=list_value, user_type_list=list_type)
                

        elif(option == "2"): # delete specific table

            delete_sql_table(conn=conn)

        elif(option == "3"): # display sql table
            
            print("\n[+]--- Display SQL Tables ---[+]")
            display_sql_tables(conn=conn)

        else:
            print("\n[-] Exiting..\n")
            continue
            
            
    elif option == "9":

        uni_sql_insert_table(conn=conn)


    elif option == "99":

        main_flag = False


    elif option == "10":

        example_pragma_table_info(conn=conn)


    elif(option.lower() == "c"):

        os.system("cls")


    else:
        print("\n[-] Please choose from options! [-]\n")
        

# close the connection
conn.close()


print("\n[+] Exiting SQL Database...")
time.sleep(3)


sys.exit(0)

'''<---| Script undergoing development |--->'''
