# my-first-project-repo
# The Event management calendar
#include <stdio.h>
#include <string.h>

#define MAX_EVENTS 100

// Structure to store events
typedef struct {
    int day, month, year;
    char description[100];
} Event;

Event events[MAX_EVENTS]; // Array to store events
int eventCount = 0;       // Number of stored events

// Function to get first day of the month using Zeller's Congruence
int getFirstDayOfMonth(int month, int year) {
    if (month < 3) {
        month += 12;
        year--;
    }
    int k = year % 100;
    int j = year / 100;
    int day = (1 + (13 * (month + 1)) / 5 + k + k / 4 + j / 4 + 5 * j) % 7;
    return day; // 0=Saturday, 1=Sunday, ..., 6=Friday
}

// Function to add an event
void addEvent() {
    if (eventCount >= MAX_EVENTS) {
        printf("Event list is full!\n");
        return;
    }

    printf("\nEnter event details:\n");
    printf("Day (1-31): ");
    scanf("%d", &events[eventCount].day);
    printf("Month (1-12): ");
    scanf("%d", &events[eventCount].month);
    events[eventCount].year = 2025; // Fixed year

    printf("Enter event description: ");
    getchar(); // Consume newline
    fgets(events[eventCount].description, 100, stdin);
    events[eventCount].description[strcspn(events[eventCount].description, "\n")] = '\0'; // Remove newline

    printf("Event added successfully!\n");
    eventCount++;
}
void deleteEvent() {
    if (eventCount == 0) {
        printf("No events to delete!\n");
        return;
    }

    int day, month;
    printf("\nEnter event date to delete:\n");
    printf("Day (1-31): ");
    scanf("%d", &day);
    printf("Month (1-12): ");
    scanf("%d", &month);

    int found = 0;
    for (int i = 0; i < eventCount; i++) {
        if (events[i].day == day && events[i].month == month) {
            found = 1;
            // Shift events left to remove the event
            for (int j = i; j < eventCount - 1; j++) {
                events[j] = events[j + 1];
            }
            eventCount--;
            printf("Event on %d/%d deleted successfully!\n", day, month);
            break;
        }
    }
}
// Function to check if an event exists for a given day
void printEventMarker(FILE *file, int day, int month) {
    for (int i = 0; i < eventCount; i++) {
        if (events[i].day == day && events[i].month == month) {
            printf("*"); // Show on screen
            fprintf(file, "*"); // Save to file
            return;
        }
    }
    printf(" "); // No event on screen
    fprintf(file, " "); // No event in file
}

// Function to print all events for a specific month
void printEventsForMonth(FILE *file, int month) {
    int found = 0;
    printf("\nEvents in this month:\n");
    fprintf(file, "\nEvents in this month:\n");

    for (int i = 0; i < eventCount; i++) {
        if (events[i].month == month) {
            printf("- %d/%d/%d: %s\n", events[i].day, events[i].month, events[i].year, events[i].description);
            fprintf(file, "- %d/%d/%d: %s\n", events[i].day, events[i].month, events[i].year, events[i].description);
            found = 1;
        }
    }

    if (!found) {
        printf("No events this month.\n");
        fprintf(file, "No events this month.\n");
    }
}

// Function to print the calendar for a given month
void printMonth(FILE *file, int month, int year) {
    int daysInMonth[] = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
    char *weekdays[] = {"Sat", "Sun", "Mon", "Tue", "Wed", "Thu", "Fri"};

    if (year % 4 == 0 && (year % 100 != 0 || year % 400 == 0)) {
        daysInMonth[1] = 29; // Leap year fix
    }

    int firstDay = getFirstDayOfMonth(month, year);

    // Print header
    printf("\n\n     %s %d\n", (month == 1) ? "January" : (month == 2) ? "February" : 
           (month == 3) ? "March" : (month == 4) ? "April" : (month == 5) ? "May" : 
           (month == 6) ? "June" : (month == 7) ? "July" : (month == 8) ? "August" : 
           (month == 9) ? "September" : (month == 10) ? "October" : 
           (month == 11) ? "November" : "December", year);

    fprintf(file, "\n\n     %s %d\n", (month == 1) ? "January" : (month == 2) ? "February" : 
            (month == 3) ? "March" : (month == 4) ? "April" : (month == 5) ? "May" : 
            (month == 6) ? "June" : (month == 7) ? "July" : (month == 8) ? "August" : 
            (month == 9) ? "September" : (month == 10) ? "October" : 
            (month == 11) ? "November" : "December", year);

    for (int i = 0; i < 7; i++) {
        printf("%s ", weekdays[i]);
        fprintf(file, "%s ", weekdays[i]);
    }
    printf("\n");
    fprintf(file, "\n");

    // Print leading spaces
    for (int i = 0; i < firstDay; i++) {
        printf("    ");
        fprintf(file, "    ");
    }

    // Print days with events marked
    for (int day = 1; day <= daysInMonth[month - 1]; day++) {
        printf("%2d", day);
        fprintf(file, "%2d", day);
        printEventMarker(file, day, month);
        printf(" ");
        fprintf(file, " ");
        
        firstDay++;
        if (firstDay == 7) {
            firstDay = 0;
            printf("\n");
            fprintf(file, "\n");
        }
    }
    printf("\n");
    fprintf(file, "\n");

    // Print all events for the selected month
    printEventsForMonth(file, month);
}

// Function to display the full year calendar
void printCalendar(int year) {
    FILE *file = fopen("calendar.txt", "a");
    if (file == NULL) {
        printf("Error opening file!\n");
        return;
    }

    for (int month = 1; month <= 12; month++) {
        printMonth(file, month, year);
    }

    fclose(file);
    printf("Calendar saved to 'calendar.txt' successfully!\n");
}

// Main function
int main() {
    int choice;
    int year = 2025;

    while (1) {
        printf("\nCalendar Menu:\n");
        printf("1. View Full Calendar\n");
        printf("2. View Specific Month\n");
        printf("3. Add Event\n");
        printf("4. Delete Event\n");
        printf("5. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        FILE *file = fopen("calendar.txt", "a"); // Append to file

        switch (choice) {
            case 1:
                printCalendar(year);
                break;
            case 2: {
                int month;
                printf("Enter month (1-12): ");
                scanf("%d", &month);
                if (month < 1 || month > 12) {
                    printf("Invalid month! Try again.\n");
                } else {
                    printMonth(file, month, year);
                }
                break;
            }
            case 3:
                addEvent();
                break;
            case 4:
                deleteEvent();
                break;
            case 5:
                printf("Exiting...\n");
                return 0;
            default:
                printf("Invalid choice! Try again.\n");
        }
        fclose(file);
    }
    return 0;
}

