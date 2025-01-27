import sqlite3
from sqlite3 import Error
from datetime import datetime
import os
from PIL import Image

def create_connection(db_file):
    """ create a database connection to the SQLite database specified by db_file """
    conn = None
    try:
        conn = sqlite3.connect(db_file)
        print(f"Connected to {db_file} successfully.")
    except Error as e:
        print(e)
    
    return conn

def create_table(conn):
    """ create a table for daily reports """
    try:
        sql_create_reports_table = """
        CREATE TABLE IF NOT EXISTS reports (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            date TEXT NOT NULL,
            activity_name TEXT NOT NULL,
            location TEXT NOT NULL,
            photo BLOB,
            description TEXT
        );
        """
        cursor = conn.cursor()
        cursor.execute(sql_create_reports_table)
        print("Reports table created successfully.")
    except Error as e:
        print(e)

def insert_report(conn, report):
    """ Insert a new report into the reports table """
    sql = '''INSERT INTO reports(date, activity_name, location, photo, description) VALUES(?, ?, ?, ?, ?)'''
    cursor = conn.cursor()
    cursor.execute(sql, report)
    conn.commit()
    print("Report inserted successfully.")
    return cursor.lastrowid

def convert_to_binary_data(filename):
    """ Convert digital data to binary format """
    with open(filename, 'rb') as file:
        blob_data = file.read()
    return blob_data

def main():
    database = "daily_reports.db"

    # create a database connection
    conn = create_connection(database)
    
    # create reports table
    if conn is not None:
        create_table(conn)

        # Input data from user
        date = input("Enter the date (YYYY-MM-DD): ")
        activity_name = input("Enter the activity name: ")
        location = input("Enter the location: ")
        photo_path = input("Enter the path to the photo: ")
        description = input("Enter the description: ")
        
        # Convert photo to binary data
        if os.path.exists(photo_path):
            photo = convert_to_binary_data(photo_path)
        else:
            print("Photo file not found. Skipping photo upload.")
            photo = None
        
        # Insert the report into database
        report = (date, activity_name, location, photo, description)
        insert_report(conn, report)
        
        # Close the connection
        conn.close()
    else:
        print("Error! Cannot create the database connection.")

if __name__ == '__main__':
    main()
