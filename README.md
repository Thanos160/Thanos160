#include <stdbool.h>

// Define input pins
#define B1 P2_0
#define B2 P2_1
#define B3 P2_2
#define D1 P2_3
#define D2 P2_4
#define ES P2_5

// Define output pins
#define C1 P2_6
#define C2 P2_7
#define S1 P2_8
#define S2 P2_9
#define S3 P2_10

// Define item types
#define ITEM_TYPE_UNKNOWN 0
#define ITEM_TYPE_NON_METALLIC_NON_REFLECTING 1
#define ITEM_TYPE_NON_METALLIC_REFLECTING 2
#define ITEM_TYPE_METALLIC 3

// Function prototypes
void setup();
void loop();
void feed_item();
void analyze_item();
void sort_item(int item_type);
void pause_operation();

int main() {
    setup();
    while (true) {
        loop();
    }
    return 0;
}

void setup() {
    // Initialize input pins
    B1 = 1;
    B2 = 1;
    B3 = 1;
    D1 = 1;
    D2 = 1;
    ES = 1;

    // Initialize output pins
    C1 = 0;
    C2 = 0;
    S1 = 0;
    S2 = 0;
    S3 = 0;
}

void loop() {
    // Check if emergency stop button is pressed or chute is full
    if (ES == 0 || (B2 == 0 && B3 == 0)) {
        pause_operation();
        return;
    }

    // Feed an item onto the second conveyor belt
    feed_item();

    // Analyze the type of item using detection sensors D1 and D2
    int item_type = ITEM_TYPE_UNKNOWN;
    if (D1 == 0) {
        item_type = ITEM_TYPE_METALLIC;
    } else if (D2 == 0) {
        item_type = ITEM_TYPE_NON_METALLIC_NON_REFLECTING;
    } else {
        item_type = ITEM_TYPE_NON_METALLIC_REFLECTING;
    }

    // Sort the item into the appropriate exit chute based on its type
    sort_item(item_type);
}

void feed_item() {
    // Start the first conveyor belt to feed an item onto the second conveyor belt
    C1 = 1;

    // Wait until the item is detected by through-beam sensor B1 at servomotor gate S1
    while (B1 == 1);

    // Stop the first conveyor belt and close servomotor gate S1 to block the fed package from continuing along C2
    C1 = 0;
    S1 = 1;
}

void analyze_item() {
}

void sort_item(int item_type) {
    // Set servomotor gates S2 and S3 based on the type of item
    switch (item_type) {
        case ITEM_TYPE_NON_METALLIC_NON_REFLECTING:
            S2 = 0;
            S3 = 0;
            break;
        case ITEM_TYPE_NON_METALLIC_REFLECTING:
            S2 = 1;
            S3 = 0;
            break;
        case ITEM_TYPE_METALLIC:
            S2 = 0;
            S3 = 1;
            break;
        default:
            break;
    }

    // Open servomotor gate S1 and start the second conveyor belt to move the item to the appropriate exit chute
    S1 = 0;
    C2 = 1;

    // Wait until either through-beam sensor B2 or B3 detects the sorted item, then stop the second conveyor belt and close servomotor gate S1 again to prepare for the next item.
while (B2 == 1 && B3 == 1);
C2 = 0; 
S1=1; 
}

void pause_operation() {
// Stop both conveyor belts 
C1=0; 
C2=0; 
}
