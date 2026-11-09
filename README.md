# codealpha_task 3
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FILE_NAME "students.dat"

// Structure to store Student details
typedef struct {
    int id;
    char name[50];
    int age;
    char course[50];
} Student;

// Function Prototypes
void addStudent();
void displayStudents();
void searchStudent();
void updateStudent();
void deleteStudent();

int main() {
    int choice;

    while (1) {
        printf("\n====================================");
        printf("\n     STUDENT MANAGEMENT SYSTEM      ");
        printf("\n====================================");
        printf("\n1. Add Student Record");
        printf("\n2. Display All Student Records");
        printf("\n3. Search Student Record");
        printf("\n4. Update Student Record");
        printf("\n5. Delete Student Record");
        printf("\n6. Exit");
        printf("\nEnter your choice (1-6): ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: addStudent(); break;
            case 2: displayStudents(); break;
            case 3: searchStudent(); break;
            case 4: updateStudent(); break;
            case 5: deleteStudent(); break;
            case 6: 
                printf("Exiting system. Goodbye!\n");
                exit(0);
            default: 
                printf("Invalid choice! Please try again.\n");
        }
    }
    return 0;
}

// 1. Function to add a new student record
void addStudent() {
    FILE *fp = fopen(FILE_NAME, "ab"); // Open in Append Binary mode
    if (fp == NULL) {
        printf("Error: Could not open file!\n");
        return;
    }

    Student s;
    printf("\nEnter Student ID (Integer): ");
    scanf("%d", &s.id);
    getchar(); // Clear input buffer newline

    printf("Enter Student Name: ");
    fgets(s.name, sizeof(s.name), stdin);
    s.name[strcspn(s.name, "\n")] = 0; // Strip trailing newline

    printf("Enter Age: ");
    scanf("%d", &s.age);
    getchar(); // Clear input buffer newline

    printf("Enter Course: ");
    fgets(s.course, sizeof(s.course), stdin);
    s.course[strcspn(s.course, "\n")] = 0; // Strip trailing newline

    fwrite(&s, sizeof(Student), 1, fp);
    fclose(fp);
    printf("Record added successfully!\n");
}

// 2. Function to display all student records
void displayStudents() {
    FILE *fp = fopen(FILE_NAME, "rb"); // Open in Read Binary mode
    if (fp == NULL) {
        printf("\nNo records found. File is empty!\n");
        return;
    }

    Student s;
    printf("\n%-10s %-20s %-10s %-20s\n", "ID", "Name", "Age", "Course");
    printf("------------------------------------------------------------\n");
    
    while (fread(&s, sizeof(Student), 1, fp) == 1) {
        printf("%-10d %-20s %-10d %-20s\n", s.id, s.name, s.age, s.course);
    }
    fclose(fp);
}

// 3. Function to search for a student record by ID
void searchStudent() {
    FILE *fp = fopen(FILE_NAME, "rb");
    if (fp == NULL) {
        printf("\nNo records found!\n");
        return;
    }

    int searchId, found = 0;
    printf("\nEnter Student ID to search: ");
    scanf("%d", &searchId);

    Student s;
    while (fread(&s, sizeof(Student), 1, fp) == 1) {
        if (s.id == searchId) {
            printf("\nRecord Found:");
            printf("\nID: %d", s.id);
            printf("\nName: %s", s.name);
            printf("\nAge: %d", s.age);
            printf("\nCourse: %s\n", s.course);
            found = 1;
            break;
        }
    }
    if (!found) {
        printf("Record not found with ID %d.\n", searchId);
    }
    fclose(fp);
}

// 4. Function to update an existing student record
void updateStudent() {
    FILE *fp = fopen(FILE_NAME, "rb+"); // Open in Read/Write Binary mode
    if (fp == NULL) {
        printf("\nNo records found!\n");
        return;
    }

    int updateId, found = 0;
    printf("\nEnter Student ID to update: ");
    scanf("%d", &updateId);

    Student s;
    while (fread(&s, sizeof(Student), 1, fp) == 1) {
        if (s.id == updateId) {
            found = 1;
            getchar(); // Clear buffer

            printf("Enter New Name: ");
            fgets(s.name, sizeof(s.name), stdin);
            s.name[strcspn(s.name, "\n")] = 0;

            printf("Enter New Age: ");
            scanf("%d", &s.age);
            getchar();

            printf("Enter New Course: ");
            fgets(s.course, sizeof(s.course), stdin);
            s.course[strcspn(s.course, "\n")] = 0;

            // Move the file pointer back to the beginning of this structure record
            fseek(fp, -sizeof(Student), SEEK_CUR);
            fwrite(&s, sizeof(Student), 1, fp);
            printf("Record updated successfully!\n");
            break;
        }
    }
    if (!found) {
        printf("Record not found with ID %d.\n", updateId);
    }
    fclose(fp);
}

// 5. Function to delete a student record
void deleteStudent() {
    FILE *fp = fopen(FILE_NAME, "rb");
    if (fp == NULL) {
        printf("\nNo records found!\n");
        return;
    }

    int deleteId, found = 0;
    printf("\nEnter Student ID to delete: ");
    scanf("%d", &deleteId);

    // Create a temporary file to clone records we wish to keep
    FILE *tempFp = fopen("temp.dat", "wb");
    if (tempFp == NULL) {
        printf("Error creating temporary file!\n");
        fclose(fp);
        return;
    }

    Student s;
    while (fread(&s, sizeof(Student), 1, fp) == 1) {
        // If the ID matches, omit it from the temp file to "delete" it
        if (s.id != deleteId) {
            fwrite(&s, sizeof(Student), 1, tempFp);
        } else {
            found = 1;
        }
    }

    fclose(fp);
    fclose(tempFp);

    // Swap the temporary file into place if deletion happened
    if (found) {
        remove(FILE_NAME);
        rename("temp.dat", FILE_NAME);
        printf("Record deleted successfully!\n");
    } else {
        remove("temp.dat"); // Delete unused temporary file
        printf("Record not found with ID %d.\n", deleteId);
    }
}
