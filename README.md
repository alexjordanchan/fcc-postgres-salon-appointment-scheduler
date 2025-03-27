# freeCodeCamp: Build a Salon Appointment Scheduler

## Instructions
Follow the instructions and get all the user stories below to pass to finish the project. 

Create your database by logging in to psql with `psql --username=freecodecamp --dbname=postgres`. 

You can query the database in your script with `psql --username=freecodecamp --dbname=salon -c "SQL QUERY HERE"`, add more flags if you need to. 

Be sure to get creative, and have fun!

**Don't forget to connect to your database to add tables after you create it** 😄

**Hints:**

- Your script needs to finish running after doing any of the tasks described below or the tests won't pass
- The tests check the script output so don't use `clear` or other commands which might erase it
- See `examples.txt` for example output of a passing script
- The tests may add data to your database, feel free to delete it

## Requirements & My Solutions
- You should create a database named salon
```
#Connect to psql
psql --username=freecodecamp --dbname=postgres

#Create a database 'salon'
CREATE DATABASE salon;
```

- You should connect to your database, then create tables named customers, appointments, and services
```
#Connect to database 'salon'
\c salon

#Create tables 'customers', 'appointments', 'services'
CREATE TABLE customers();
CREATE TABLE appointments();
CREATE TABLE services();
```

- Each table should have a primary key column that automatically increments
- Each primary key column should follow the naming convention, `table_name_id`. For example, the `customers` table should have a `customer_id` key. Note that there’s no `s` at the end of `customer`
```
#Alter table 'customers' - add column 'customer_id'
ALTER TABLE customers ADD COLUMN customer_id SERIAL PRIMARY KEY NOT NULL;

#Alter table 'appointments - add column 'appointment_id'
ALTER TABLE appointments ADD COLUMN appointment_id SERIAL PRIMARY KEY NOT NULL;

#Alter table 'services' - add column 'service_id'
ALTER TABLE services ADD COLUMN service_id SERIAL PRIMARY KEY NOT NULL;
```

- Your `appointments` table should have a `customer_id` foreign key that references the `customer_id` column from the `customers` table
- Your `appointments` table should have a `service_id` foreign key that references the `service_id` column from the `services` table
```
# Alter table 'appointments' - add column 'customer_id' (foreign key)
ALTER TABLE appointments ADD COLUMN customer_id INT NOT NULL;
ALTER TABLE appointments ADD FOREIGN KEY(customer_id) REFERENCES customers(customer_id);

# Altertable 'appointments' - add column 'service_id' (foreign key)
ALTER TABLE appointments ADD COLUMN service_id INT NOT NULL;
ALTER TABLE appointments ADD FOREIGN KEY(service_id) REFERENCES services(service_id);
```

- Your `customers` table should have `phone` that is a `VARCHAR` and must be unique
- Your `customers` and `services` tables should have a `name` column
- Your `appointments` table should have a `time` column that is a `VARCHAR`
```
# Alter table 'customers' - add column 'phone'
ALTER TABLE customers ADD COLUMN phone VARCHAR(255) UNIQUE NOT NULL;

# Alter table 'customers' - add column 'name'
ALTER TABLE customers ADD COLUMN name VARCHAR(255) NOT NULL;

# Alter table 'services' - add column 'name'
ALTER TABLE services ADD COLUMN name VARCHAR(255) NOT NULL;

# Alter table 'appointments' - add column 'time'
ALTER TABLE appointments ADD COLUMN time VARCHAR(255) NOT NULL;
```

- You should have at least three rows in your services table for the different services you offer, one with a service_id of 1
```
# Insert into table 'servicse' - add value to 'name' column
INSERT INTO services(name) VALUES('cut');
INSERT INTO services(name) VALUES('color');
INSERT INTO services(name) VALUES('perm');
INSERT INTO services(name) VALUES('style');
INSERT INTO services(name) VALUES('trim');
```

- You should create a script file named `salon.sh` in the `project` folder
- Your script file should have executable permissions
```
# Split terminal, then Create a script file on terminal
touch salon.sh

# Allow executable permissions
chmod +x salon.sh
```

- Your script file should have a “shebang” that uses bash when the file is executed (use `#! /bin/bash`)
- You should not use the `clear` command in your script

- You should display a numbered list of the services you offer before the first prompt for input, each with the format `#) <service>`. For example, `1) cut`, where `1` is the `service_id`
- If you pick a service that doesn't exist, you should be shown the same list of services again

- Your script should prompt users to enter a `service_id`, phone number, a name if they aren’t already a customer, and a time. You should use `read` to read these inputs into variables named `SERVICE_ID_SELECTED`, `CUSTOMER_PHONE`, `CUSTOMER_NAME`, and `SERVICE_TIME`
- If a phone number entered doesn’t exist, you should get the customers name and enter it, and the phone number, into the `customers` table

- You can create a row in the `appointments` table by running your script and entering `1`, `555-555-5555`, `Fabio`, `10:30` at each request for input if that phone number isn’t in the `customers` table. The row should have the `customer_id` for that customer, and the `service_id` for the service entered
- You can create another row in the `appointments` table by running your script and entering `2`, `555-555-5555`, `11am` at each request for input if that phone number is already in the `customers` table. The row should have the `customer_id` for that customer, and the `service_id` for the service entered

- After an appointment is successfully added, you should output the message `I have put you down for a <service> at <time>, <name>.` For example, if the user chooses `cut` as the service, `10:30` is entered for the time, and their name is `Fabio` in the database the output would be `I have put you down for a cut at 10:30, Fabio.` Make sure your script finishes running after completing any of the tasks above, or else the tests won't pass

```
#! /bin/bash

PSQL="psql -X --username=freecodecamp --dbname=salon --tuples-only -c"

echo -e "\n ~~~~~ MY SALON ~~~~~\n"
echo -e "Welcome to My Salon, how can I help you?\n"

CUSTOMER_NAME=""
CUSTOMER_PHONE=""
SERVICES=$($PSQL "SELECT service_id, name FROM services ORDER BY service_id")

CREATE_APPOINTMENT(){
  SERVICE_NAME=$($PSQL "SELECT name FROM services WHERE service_id=$SERVICE_ID_SELECTED")
  CUST_NAME=$($PSQL "SELECT name FROM customers WHERE phone='$CUSTOMER_PHONE'")
  CUST_ID=$($PSQL "SELECT customer_id FROM customers WHERE phone='$CUSTOMER_PHONE'")
  SERVICE_NAME_FORMATTED=$(echo $SERVICE_NAME | sed 's/\s//g' -E)
  CUST_NAME_FORMATTED=$(echo $CUST_NAME | sed 's/\s//g' -E)
  echo -e "\nWhat time would you like your $SERVICE_NAME_FORMATTED, $CUST_NAME_FORMATTED?"
  read SERVICE_TIME
  INSERTED=$($PSQL "INSERT INTO appointments (customer_id, service_id, time) VALUES ($CUST_ID, $SERVICE_ID_SELECTED, '$SERVICE_TIME')")
  echo -e "\nI have put you down for a $SERVICE_NAME_FORMATTED at $SERVICE_TIME, $CUST_NAME_FORMATTED."
}

MAIN_MENU(){
echo -e "\nWhat's your phone number?"
read CUSTOMER_PHONE

HAVE_CUST=$($PSQL "SELECT customer_id, name FROM customers WHERE phone='$CUSTOMER_PHONE'")

if [[ -z $HAVE_CUST ]]
then
  echo -e "\nI don't have a record for that phone number, what's your name?"
  read CUSTOMER_NAME

  INSERTED=$($PSQL "INSERT INTO customers (name, phone) VALUES ('$CUSTOMER_NAME', '$CUSTOMER_PHONE')")

  CREATE_APPOINTMENT
  else

  CREATE_APPOINTMENT
fi
}



LIST_SERVICES(){
  if [[ $1 ]]
  then
    echo -e "\n$1"
  fi

  echo "$SERVICES" | while read SERVICE_ID BAR NAME
  do
    echo "$SERVICE_ID) $NAME"
  done

  read SERVICE_ID_SELECTED
  # if input is not a number
  if [[ ! $SERVICE_ID_SELECTED =~ ^[0-9]+$ ]]
  then
    LIST_SERVICES "I could not find that service. What would you like today?"
  else
    HAVE_SERVICE=$($PSQL "SELECT service_id FROM services WHERE service_id=$SERVICE_ID_SELECTED")

    if [[ -z $HAVE_SERVICE ]]
    then
      LIST_SERVICES "I could not find that service. What would you like today?"
    else
      MAIN_MENU
    fi
  fi

}

LIST_SERVICES
```
