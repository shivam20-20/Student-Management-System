# Student-Management-System

#include <iostream>
#include <string>
#include <sstream>
#include <iomanip>
#include <vector>
#include <sqlite3.h>

using namespace std;

// Define a structure to represent a student
struct Student {
    int id;
    string name;
    int rollNumber;
    string department;
    string grade;
    double feeDues;
    string address;
    int clubsParticipated;
    string libraryBook;
};

// SQLite database connection
sqlite3* db;

// Function to execute SQL queries
int executeSQL(const string& sql) {
    char* errorMsg = 0;
    int rc = sqlite3_exec(db, sql.c_str(), NULL, 0, &errorMsg);
    if (rc != SQLITE_OK) {
        cerr << "SQL error: " << errorMsg << endl;
        sqlite3_free(errorMsg);
    }
    return rc;
}

// Function to initialize the database
void initializeDatabase() {
    int rc = sqlite3_open("student.db", &db);
    if (rc) {
        cerr << "Can't open database: " << sqlite3_errmsg(db) << endl;
        exit(0);
    }

    // Create Students table if not exists
    string sql = "CREATE TABLE IF NOT EXISTS Students ("
                 "ID INTEGER PRIMARY KEY AUTOINCREMENT,"
                 "Name TEXT NOT NULL,"
                 "RollNumber INT NOT NULL,"
                 "Department TEXT NOT NULL,"
                 "Grade TEXT,"
                 "FeeDues REAL,"
                 "Address TEXT,"
                 "ClubsParticipated INT,"
                 "LibraryBook TEXT);";
    executeSQL(sql);
}

// Function to generate a unique student ID
int generateStudentID() {
    string sql = "SELECT MAX(ID) FROM Students;";
    sqlite3_stmt* stmt;
    int rc = sqlite3_prepare_v2(db, sql.c_str(), -1, &stmt, NULL);
    if (rc != SQLITE_OK) {
        cerr << "SQL error: " << sqlite3_errmsg(db) << endl;
        return -1;
    }
    rc = sqlite3_step(stmt);
    int maxID = rc == SQLITE_ROW ? sqlite3_column_int(stmt, 0) : 0;
    sqlite3_finalize(stmt);
    return maxID + 1;
}

// Function to add a new student
void addStudent(const Student& student) {
    string sql = "INSERT INTO Students (Name, RollNumber, Department, Grade, FeeDues, Address, ClubsParticipated, LibraryBook) "
                 "VALUES ('" + student.name + "', " + to_string(student.rollNumber) + ", '" + student.department + "', '" + student.grade + "', " +
                 to_string(student.feeDues) + ", '" + student.address + "', " + to_string(student.clubsParticipated) + ", '" + student.libraryBook + "');";
    executeSQL(sql);
    cout << "Student added successfully!\n";
}

// Function to find student data by ID number
void findStudentByID(int id) {
    string sql = "SELECT * FROM Students WHERE ID = " + to_string(id) + ";";
    sqlite3_stmt* stmt;
    int rc = sqlite3_prepare_v2(db, sql.c_str(), -1, &stmt, NULL);
    if (rc != SQLITE_OK) {
        cerr << "SQL error: " << sqlite3_errmsg(db) << endl;
        return;
    }
    rc = sqlite3_step(stmt);
    if (rc == SQLITE_ROW) {
        cout << "Student Details:\n";
        cout << "ID: " << sqlite3_column_int(stmt, 0) << endl;
        cout << "Name: " << sqlite3_column_text(stmt, 1) << endl;
        cout << "Roll Number: " << sqlite3_column_int(stmt, 2) << endl;
        cout << "Department: " << sqlite3_column_text(stmt, 3) << endl;
        cout << "Grade: " << sqlite3_column_text(stmt, 4) << endl;
        cout << "Fee Dues: " << sqlite3_column_double(stmt, 5) << endl;
        cout << "Address: " << sqlite3_column_text(stmt, 6) << endl;
        cout << "Clubs Participated: " << sqlite3_column_int(stmt, 7) << endl;
        cout << "Library Book: " << sqlite3_column_text(stmt, 8) << endl;
    } else {
        cout << "Student with ID " << id << " not found.\n";
    }
    sqlite3_finalize(stmt);
}

// Function to delete student data by ID number
void deleteStudentByID(int id) {
    string sql = "DELETE FROM Students WHERE ID = " + to_string(id) + ";";
    executeSQL(sql);
    cout << "Student data deleted successfully!\n";
}

// Function to modify student details by ID number
void modifyStudentByID(int id) {
    string name, department, grade, address, libraryBook;
    int rollNumber, clubsParticipated;
    double feeDues;

    cout << "Enter new name: ";
    getline(cin, name);
    cout << "Enter new roll number: ";
    cin >> rollNumber;
    cin.ignore(); // Consume the newline character
    cout << "Enter new department: ";
    getline(cin, department);
    cout << "Enter new grade: ";
    getline(cin, grade);
    cout << "Enter new fee dues: ";
    cin >> feeDues;
    cin.ignore(); // Consume the newline character
    cout << "Enter new address: ";
    getline(cin, address);
    cout << "Enter new number of clubs participated: ";
    cin >> clubsParticipated;
    cin.ignore(); // Consume the newline character
    cout << "Enter new library book: ";
    getline(cin, libraryBook);

    string sql = "UPDATE Students SET Name = '" + name + "', RollNumber = " + to_string(rollNumber) + ", Department = '" + department + "', "
                 "Grade = '" + grade + "', FeeDues = " + to_string(feeDues) + ", Address = '" + address + "', "
                 "ClubsParticipated = " + to_string(clubsParticipated) + ", LibraryBook = '" + libraryBook + "' WHERE ID = " + to_string(id) + ";";
    executeSQL(sql);
    cout << "Student details modified successfully!\n";
}

// Function to display student data entry format
void displayDataEntryFormat() {
    cout << "\nStudent Data Entry Format:\n";
    cout << "Name: [First Name Last Name]\n";
    cout << "Roll Number: [Integer]\n";
    cout << "Department: [Department Name]\n";
    cout << "Grade: [Grade]\n";
    cout << "Fee Dues: [Double]\n";
    cout << "Address: [Address]\n";
    cout << "Clubs Participated: [Integer]\n";
    cout << "Library Book: [Book Title]\n";
}

int main() {
    initializeDatabase();

    while (true) {
        cout << "\nStudent Data Management System\n";
        cout << "1. Add Student\n";
        cout << "2. Find Student by ID\n";
        cout << "3. Delete Student by ID\n";
        cout << "4. Modify Student by ID\n";
        cout << "5. Exit\n";
        cout << "Enter your choice: ";
        int choice;
        cin >> choice;
        cin.ignore(); // Consume the newline character
        switch (choice) {
            case 1: {
                Student newStudent;
                cout << "Enter student data:\n";
                displayDataEntryFormat();
                newStudent.id = generateStudentID();
                cout << "Student ID: " << newStudent.id << endl;
                cout << "Name: ";
                getline(cin, newStudent.name);
                cout << "Roll Number: ";
                cin >> newStudent.rollNumber;
                cin.ignore(); // Consume the newline character
                cout << "Department: ";
                getline(cin, newStudent.department);
                cout << "Grade: ";
                getline(cin, newStudent.grade);
                cout << "Fee Dues: ";
                cin >> newStudent.feeDues;
                cin.ignore(); // Consume the newline character
                cout << "Address: ";
                getline(cin, newStudent.address);
                cout << "Clubs Participated: ";
                cin >> newStudent.clubsParticipated;
                cin.ignore(); // Consume the newline character
                cout << "Library Book: ";
                getline(cin, newStudent.libraryBook);
                addStudent(newStudent);
                break;
            }
            case 2: {
                int id;
                cout << "Enter student ID: ";
                cin >> id;
                findStudentByID(id);
                break;
            }
            case 3: {
                int id;
                cout << "Enter student ID to delete: ";
                cin >> id;
                deleteStudentByID(id);
                break;
            }
            case 4: {
                int id;
                cout << "Enter student ID to modify: ";
                cin >> id;
                cin.ignore(); // Consume the newline character
                modifyStudentByID(id);
                break;
            }
            case 5:
                cout << "Exiting...\n";
                sqlite3_close(db);
                return 0;
            default:
                cout << "Invalid choice. Please try again.\n";
        }
    }

    return 0;
}
