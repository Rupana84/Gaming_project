# Gaming_project

#ifndef ITEM_H                 // [REQ: Structure] Header guard to prevent multiple inclusion
#define ITEM_H                 // [REQ: Structure] Good program structure with .h/.cpp separation

#include <string>              // [REQ: Quality] Use standard types; clear and descriptive

class Player;                  // [REQ: Structure] Forward declaration avoids circular include with Player

class Item {                   // [REQ: OOP] Abstract base class for all items (common interface)
public:                        // [REQ: OOP] Public polymorphic API shared by all derived items
    virtual ~Item() = default; // [REQ: Memory] Virtual destructor enables safe delete via base pointer

    // Pure virtuals define the abstract interface every item must implement
    virtual std::string getName() const = 0;   // [REQ: OOP] Abstraction; overridden by derived classes
    virtual std::string getType() const = 0;   // [REQ: OOP] Abstraction; overridden by derived classes
    virtual std::string describe() const = 0;  // [REQ: Polymorphism] Late-bound per item type
    virtual void use(Player& player) = 0;      // [REQ: Polymorphism] Virtual behavior on use/equip/consume
};

#endif // ITEM_H                // [REQ: Structure] End header guard

----------------------------------------------

#ifndef WEAPON_H                    // [REQ: Structure] Header guard for correct program structure
#define WEAPON_H                    // [REQ: Structure] Prevent multiple inclusion

#include "Item.h"                   // [REQ: Inheritance] Inherits from abstract base class Item
#include <string>                   // [REQ: Quality] Use of standard library types for clarity

// ------------------------------------------------------------
// Derived class: Weapon
// Demonstrates inheritance, overriding, and polymorphism
// ------------------------------------------------------------
class Weapon : public Item {        // [REQ: Inheritance] Derived class from Item
    std::string name;               // [REQ: OOP] Weapon-specific attribute (encapsulation)
    int damage;                     // [REQ: OOP] Unique field for damage
    bool equipped{false};           // [REQ: Functionality] Tracks equip status
public:
    // Constructor
    Weapon(const std::string& n, int dmg)
        : name(n), damage(dmg) {}   // [REQ: Quality] Proper initialization list and clear naming

    // Virtual overrides of Item base class methods
    std::string getName() const override { return name; }     // [REQ: Overriding] Implements abstract function
    std::string getType() const override { return "Weapon"; } // [REQ: Overriding] Type identification
    std::string describe() const override;                    // [REQ: Polymorphism] Late-bound behavior
    void use(Player& player) override;                        // [REQ: Polymorphism] Equip behavior

    // Additional Weapon-specific methods
    int getDamage() const { return damage; }                  // [REQ: Functionality] Used in Player::attack()
    bool isEquipped() const { return equipped; }              // [REQ: Functionality] Status check
    void setEquipped(bool v) { equipped = v; }                // [REQ: Functionality] Update equip flag
};

#endif // WEAPON_H                 // [REQ: Structure] End of header guard

---------------------------------------------------

#include "Weapon.h"               // [REQ: Structure] Include class declaration
#include "Player.h"               // [REQ: Coupling] Needed to call Player’s equipWeapon() function
#include <sstream>                // [REQ: Quality] Used for safe string building in describe()

// ------------------------------------------------------------
// Weapon::describe()
// Demonstrates overriding and polymorphism
// ------------------------------------------------------------
std::string Weapon::describe() const { // [REQ: Overriding] Implementation of virtual describe()
    std::ostringstream os;             // [REQ: Quality] Use of string stream for formatted text
    // Description includes weapon stats and equipped state
    os << name << " [Weapon] dmg=" << damage << (equipped ? " (equipped)" : ""); 
    // [REQ: Functionality] Clear and user-readable description for inventory listing
    return os.str();                   // [REQ: Functionality] Returns polymorphic description
}

// ------------------------------------------------------------
// Weapon::use()
// Demonstrates polymorphism: different behavior for each item type
// ------------------------------------------------------------
void Weapon::use(Player& player) {     // [REQ: Polymorphism] Behavior specific to Weapon
    player.equipWeapon(this);          // [REQ: Functionality] Calls Player to equip this weapon
}

------------------------------------------

#ifndef ARMOR_H                     // [REQ: Structure] Header guard for clean compilation
#define ARMOR_H                     // [REQ: Structure] Prevent multiple inclusion

#include "Item.h"                   // [REQ: Inheritance] Derives from abstract base Item
#include <string>                   // [REQ: Quality] Standard types for clarity

// ------------------------------------------------------------
// Derived class: Armor
// Demonstrates inheritance, overriding, and polymorphism
// ------------------------------------------------------------
class Armor : public Item {         // [REQ: Inheritance] Concrete subclass of Item
    std::string name;               // [REQ: OOP] Armor-specific attribute (encapsulation)
    int defense;                    // [REQ: OOP] Unique field for defense value
    bool equipped{false};           // [REQ: Functionality] Track whether this armor is equipped
public:
    // Constructor
    Armor(const std::string& n, int def)
        : name(n), defense(def) {}  // [REQ: Quality] Clear initialization of fields

    // Overrides from Item (abstract base)
    std::string getName() const override { return name; }      // [REQ: Overriding] Implement abstract API
    std::string getType() const override { return "Armor"; }   // [REQ: Overriding] Type identification
    std::string describe() const override;                     // [REQ: Polymorphism] Late-bound description
    void use(Player& player) override;                         // [REQ: Polymorphism] Equip behavior

    // Armor-specific accessors
    int getDefense() const { return defense; }                 // [REQ: Functionality] Used in Player::defense()
    bool isEquipped() const { return equipped; }               // [REQ: Functionality] Status check
    void setEquipped(bool v) { equipped = v; }                 // [REQ: Functionality] Update equip flag
};

#endif // ARMOR_H                   // [REQ: Structure] End header guard

--------------------------------------------------------------

#include "Armor.h"                 // [REQ: Structure] Include Armor class declaration
#include "Player.h"                // [REQ: Coupling] Needed to call Player::equipArmor()
#include <sstream>                 // [REQ: Quality] For building formatted text safely

// ------------------------------------------------------------
// Armor::describe()
// Demonstrates overriding, encapsulation, and polymorphism
// ------------------------------------------------------------
std::string Armor::describe() const {    // [REQ: Overriding] Implements Item::describe()
    std::ostringstream os;               // [REQ: Quality] Use stringstream for safer concatenation
    // Show armor name, defense value, and equipped state
    os << name << " [Armor] def=" << defense << (equipped ? " (equipped)" : "");
    // [REQ: Functionality] Clear text for inventory display
    return os.str();                     // [REQ: Functionality] Return string for polymorphic use
}

// ------------------------------------------------------------
// Armor::use()
// Demonstrates polymorphism — unique behavior for Armor objects
// ------------------------------------------------------------
void Armor::use(Player& player) {        // [REQ: Polymorphism] Called through Item* in Player
    player.equipArmor(this);             // [REQ: Functionality] Delegate equip logic to Player
}

-----------------------------------------------------

#ifndef POTION_H                    // [REQ: Structure] Header guard for modular structure
#define POTION_H                    // [REQ: Structure] Prevent multiple inclusion

#include "Item.h"                   // [REQ: Inheritance] Inherits from abstract base Item
#include <string>                   // [REQ: Quality] Use of standard library types

// ------------------------------------------------------------
// Derived class: Potion
// Demonstrates inheritance, overriding, polymorphism, and memory handling
// ------------------------------------------------------------
class Potion : public Item {        // [REQ: Inheritance] Derived class from Item
    std::string name;               // [REQ: OOP] Encapsulated name attribute
    int heal;                       // [REQ: OOP] Unique attribute for healing power
public:
    // Constructor initializes potion attributes
    Potion(const std::string& n, int h)
        : name(n), heal(h) {}       // [REQ: Quality] Proper constructor with initialization list

    // Overrides from Item base class
    std::string getName() const override { return name; }      // [REQ: Overriding] Implement abstract API
    std::string getType() const override { return "Potion"; }  // [REQ: Overriding] Identify item type
    std::string describe() const override;                     // [REQ: Polymorphism] Late-bound string info
    void use(Player& player) override;                         // [REQ: Polymorphism] Unique use behavior

    // Extra helper for Player to access potion amount
    int amount() const { return heal; }                        // [REQ: Functionality] Healing magnitude
};

#endif // POTION_H                   // [REQ: Structure] End header guard

-------------------------------------------------

#include "Potion.h"                 // [REQ: Structure] Include class declaration
#include "Player.h"                 // [REQ: Coupling] Access Player methods (healing, removal)
#include <sstream>                  // [REQ: Quality] Used to safely format description text

// ------------------------------------------------------------
// Potion::describe()
// Demonstrates overriding, encapsulation, and polymorphism
// ------------------------------------------------------------
std::string Potion::describe() const {   // [REQ: Overriding] Overrides Item::describe()
    std::ostringstream os;               // [REQ: Quality] Safer string formatting
    os << name << " [Potion] heal=" << heal; 
    // [REQ: Functionality] Displays potion name and healing amount
    return os.str();                     // [REQ: Functionality] Return polymorphic description
}

// ------------------------------------------------------------
// Potion::use()
// Demonstrates polymorphism and memory management
// ------------------------------------------------------------
void Potion::use(Player& player) {       // [REQ: Polymorphism] Specific runtime behavior for Potion
    player.addHealth(heal);              // [REQ: Functionality] Apply healing effect to player
    player.consumeCurrentItem(this);     // [REQ: Memory] Remove and delete potion after use
}


-----------------------------------------------------

#ifndef PLAYER_H                         // [REQ: Structure] Header guard for modular design
#define PLAYER_H                         // [REQ: Structure] Prevent multiple inclusion

#include "Item.h"                        // [REQ: OOP] Player owns and manages Item objects via base pointers
#include <vector>                        // [REQ: Data Structure] Inventory container for multiple items
#include <string>                        // [REQ: Quality] Standard library string
#include <iostream>                      // [REQ: I/O] For console output

class Weapon;                            // [REQ: Structure] Forward declaration for equipWeapon()
class Armor;                             // [REQ: Structure] Forward declaration for equipArmor()

// ------------------------------------------------------------
// Class: Player
// Demonstrates aggregation, polymorphism, manual memory management, and stack vs heap usage
// ------------------------------------------------------------
class Player {                           
    std::string name;                    // [REQ: Quality] Player’s name
    int health{100};                     // [REQ: Functionality] Base health points
    int baseAttack{5};                   // [REQ: Functionality] Default attack value
    int baseDefense{2};                  // [REQ: Functionality] Default defense value

    Weapon* equippedWeapon{nullptr};     // [REQ: Functionality] Pointer to equipped Weapon (not owned)
    Armor*  equippedArmor{nullptr};      // [REQ: Functionality] Pointer to equipped Armor (not owned)
    std::vector<Item*> inventory;        // [REQ: Memory/Data] Stores heap-allocated Item objects

public:
    explicit Player(const std::string& n) : name(n) {}  // [REQ: Quality] Constructor initializes name
    ~Player();                                          // [REQ: Memory] Destructor frees heap items

    // --------------------------------------------------------
    // Disable copying/moving to prevent double deletion
    // --------------------------------------------------------
    Player(const Player&) = delete;                    // [REQ: Memory Safety] Prevent shallow copy
    Player& operator=(const Player&) = delete;         // [REQ: Memory Safety] Prevent double-free
    Player(Player&&) = delete;                         // [REQ: Memory Safety] No move semantics
    Player& operator=(Player&&) = delete;              // [REQ: Memory Safety]

    // --------------------------------------------------------
    // Player stats and accessors
    // --------------------------------------------------------
    void addHealth(int v);                             // [REQ: Functionality] Increase/decrease HP
    int attack() const;                                // [REQ: Functionality] Base + weapon damage
    int defense() const;                               // [REQ: Functionality] Base + armor defense
    int getHealth() const { return health; }           // [REQ: Quality] Accessor for HP
    std::string getName() const { return name; }       // [REQ: Quality] Accessor for name

    // --------------------------------------------------------
    // Inventory management
    // --------------------------------------------------------
    void addItem(Item* item);                          // [REQ: Memory/Data] Takes ownership of heap item
    void listItems() const;                            // [REQ: Functionality] Display all items
    bool removeItemByIndex(size_t idx);                // [REQ: Memory] Delete + remove item from inventory
    Item* getItem(size_t idx);                         // [REQ: Functionality] Access item by index

    // --------------------------------------------------------
    // Polymorphic use
    // --------------------------------------------------------
    void useItem(size_t idx);                          // [REQ: Polymorphism] Calls virtual use() dynamically

    // --------------------------------------------------------
    // Equip helpers
    // --------------------------------------------------------
    void equipWeapon(Weapon* w);                       // [REQ: Functionality] Set active weapon
    void equipArmor(Armor* a);                         // [REQ: Functionality] Set active armor

    // --------------------------------------------------------
    // Potions: remove and delete after use
    // --------------------------------------------------------
    void consumeCurrentItem(Item* item);               // [REQ: Memory] Manual delete after consumption

    // --------------------------------------------------------
    // Stack vs Heap demonstration
    // --------------------------------------------------------
    void stackVsHeapDemo();                            // [REQ: Stack/Heap] Compare lifetimes of both
};

#endif // PLAYER_H                        // [REQ: Structure] End of header guard

-------------------------------

#include "Player.h"                  // [REQ: Structure] Include Player class definition
#include "Weapon.h"                  // [REQ: Coupling] For equipWeapon() implementation
#include "Armor.h"                   // [REQ: Coupling] For equipArmor() implementation
#include "Potion.h"                  // [REQ: Coupling] For Potion use and deletion
#include <iomanip>                   // [REQ: Quality] Used for formatted console output

// ------------------------------------------------------------
// Destructor
// Ensures all dynamically allocated memory is released
// ------------------------------------------------------------
Player::~Player() {                  
    for (Item* it : inventory)       // [REQ: Memory] Iterate over stored pointers
        delete it;                   // [REQ: Memory] Free each heap-allocated object
    inventory.clear();               // [REQ: Memory] Empty the container after deletion
}

// ------------------------------------------------------------
// Health control
// ------------------------------------------------------------
void Player::addHealth(int v) {      
    health += v;                     // [REQ: Functionality] Increase or decrease HP
    if (health > 100) health = 100;  // [REQ: Quality] Prevent exceeding max HP
    if (health < 0) health = 0;      // [REQ: Quality] Prevent negative HP
}

// ------------------------------------------------------------
// Attack and Defense calculations
// ------------------------------------------------------------
int Player::attack() const {         
    int bonus = 0;                   
    if (equippedWeapon)              // [REQ: Functionality] Add weapon bonus if equipped
        bonus += equippedWeapon->getDamage();
    return baseAttack + bonus;       // [REQ: Functionality] Total attack value
}

int Player::defense() const {        
    int bonus = 0;                   
    if (equippedArmor)               // [REQ: Functionality] Add armor bonus if equipped
        bonus += equippedArmor->getDefense();
    return baseDefense + bonus;      // [REQ: Functionality] Total defense value
}

// ------------------------------------------------------------
// Inventory management
// ------------------------------------------------------------
void Player::addItem(Item* item) {   
    inventory.push_back(item);       // [REQ: Memory/Data] Store base pointer (heap ownership)
}

void Player::listItems() const {     
    if (inventory.empty()) {         // [REQ: Quality] Graceful handling when empty
        std::cout << "Inventory is empty.\n";
        return;
    }
    for (size_t i = 0; i < inventory.size(); ++i) {   // [REQ: Functionality] Iterate items
        std::cout << std::setw(2) << i << ": " << inventory[i]->describe() << "\n";
        // [REQ: Polymorphism] Calls virtual describe() → dynamic binding
    }
}

bool Player::removeItemByIndex(size_t idx) {           
    if (idx >= inventory.size()) return false;         // [REQ: Quality] Prevent invalid access
    Item* it = inventory[idx];                         // [REQ: Functionality] Access pointer

    // If the item being removed is equipped, unequip it
    if (it == equippedWeapon) equippedWeapon = nullptr; // [REQ: Functionality]
    if (it == equippedArmor)  equippedArmor  = nullptr; // [REQ: Functionality]

    delete it;                                         // [REQ: Memory] Free allocated memory
    inventory.erase(inventory.begin() + idx);          // [REQ: Data Structure] Remove from vector
    return true;                                       // [REQ: Functionality]
}

Item* Player::getItem(size_t idx) {                    
    if (idx >= inventory.size()) return nullptr;       // [REQ: Quality] Bounds check
    return inventory[idx];                             // [REQ: Functionality]
}

// ------------------------------------------------------------
// Use Item — demonstrates polymorphism (dynamic dispatch)
// ------------------------------------------------------------
void Player::useItem(size_t idx) {                     
    Item* it = getItem(idx);                           // [REQ: Functionality] Retrieve by index
    if (!it) {                                         // [REQ: Quality]
        std::cout << "Invalid index.\n";               // [REQ: Functionality]
        return;                                        // [REQ: Quality]
    }
    it->use(*this);                                    // [REQ: Polymorphism] Dynamic dispatch based on item type
}

// ------------------------------------------------------------
// Equip functions
// ------------------------------------------------------------
void Player::equipWeapon(Weapon* w) {                  
    if (equippedWeapon) equippedWeapon->setEquipped(false);  // [REQ: Functionality] Unequip old
    equippedWeapon = w;                                      // [REQ: Functionality] Set new weapon
    if (equippedWeapon) equippedWeapon->setEquipped(true);   // [REQ: Functionality] Mark as equipped
    std::cout << "Equipped weapon: " << w->getName()         // [REQ: Functionality] Output confirmation
              << ". Attack=" << attack() << "\n";
}

void Player::equipArmor(Armor* a) {                    
    if (equippedArmor) equippedArmor->setEquipped(false);    // [REQ: Functionality] Unequip old
    equippedArmor = a;                                       // [REQ: Functionality]
    if (equippedArmor) equippedArmor->setEquipped(true);     // [REQ: Functionality]
    std::cout << "Equipped armor: " << a->getName()          // [REQ: Functionality]
              << ". Defense=" << defense() << "\n";
}

// ------------------------------------------------------------
// Consume Item — manual memory handling
// ------------------------------------------------------------
void Player::consumeCurrentItem(Item* item) {          
    for (size_t i = 0; i < inventory.size(); ++i) {    // [REQ: Data Structure] Search vector
        if (inventory[i] == item) {                    
            delete inventory[i];                       // [REQ: Memory] Delete consumed potion
            inventory.erase(inventory.begin() + i);     // [REQ: Data Structure] Remove from vector
            std::cout << "Potion consumed and removed.\n";
            return;
        }
    }
}

// ------------------------------------------------------------
// Stack vs Heap demonstration
// Shows lifetime difference between automatic and dynamic allocation
// ------------------------------------------------------------
void Player::stackVsHeapDemo() {                       
    Potion stackPotion("Stack Tonic", 5);               // [REQ: Stack] Local automatic object
    std::cout << "Stack object: " << stackPotion.describe() << "\n";
    // [REQ: Demonstration] Exists only in this scope

    Item* heapPotion = new Potion("Heap Elixir", 7);    // [REQ: Heap] Dynamically allocated object
    std::cout << "Heap object: " << heapPotion->describe() 
              << " (added to inventory)\n";             // [REQ: Demonstration]
    addItem(heapPotion);                                // [REQ: Memory/Data] Added to Player’s inventory (deleted later)
}

-----------------------------------------------

#include <iostream>                 // [REQ: I/O] Console-based interaction for menu system
#include <limits>                   // [REQ: Quality] Used to clear invalid input
#include <climits>                  // [REQ: Quality] Used for input validation bounds
#include "Player.h"                 // [REQ: Structure] Include Player class for main logic
#include "Weapon.h"                 // [REQ: Structure] Access Weapon class
#include "Armor.h"                  // [REQ: Structure] Access Armor class
#include "Potion.h"                 // [REQ: Structure] Access Potion class

// ------------------------------------------------------------
// Input helpers — maintain robustness and user validation
// ------------------------------------------------------------
static void clearInput() {          // [REQ: Quality] Prevent input stream errors
    std::cin.clear();               // [REQ: Quality] Reset input fail state
    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); // [REQ: Quality] Clear buffer
}

static int readInt(const std::string& prompt, int minV = INT_MIN, int maxV = INT_MAX) {
    // [REQ: Quality] Safe and validated numeric input
    int v;
    while (true) {
        std::cout << prompt;        // [REQ: I/O] Prompt user
        if (std::cin >> v && v >= minV && v <= maxV) return v;  // [REQ: Quality] Valid input range
        std::cout << "Please enter a valid number";             // [REQ: Quality] Feedback
        if (minV != INT_MIN || maxV != INT_MAX)
            std::cout << " (" << minV << "–" << maxV << ")";
        std::cout << ".\n";
        clearInput();               // [REQ: Quality] Retry safely
    }
}

static std::string readLine(const std::string& prompt) {
    // [REQ: Quality] Handles mixed getline and >> input safely
    std::cout << prompt;
    clearInput();
    std::string s;
    std::getline(std::cin, s);
    return s;
}

// ------------------------------------------------------------
// Menu: Add item (creates Weapon, Armor, or Potion dynamically)
// ------------------------------------------------------------
static void addItemMenu(Player& p) {
    int c = readInt("1) Weapon  2) Armor  3) Potion  0) Cancel\n> ", 0, 3);
    if (c == 0) return;             // [REQ: Functionality] Exit menu

    if (c == 1) {                   // [REQ: Functionality] Add Weapon
        std::string name = readLine("Weapon name: ");
        int dmg = readInt("Damage: ", 0);
        p.addItem(new Weapon(name, dmg));  // [REQ: Memory] Dynamic allocation on heap
    } else if (c == 2) {            // [REQ: Functionality] Add Armor
        std::string name = readLine("Armor name: ");
        int def = readInt("Defense: ", 0);
        p.addItem(new Armor(name, def));   // [REQ: Memory] Allocated with new
    } else {                        // [REQ: Functionality] Add Potion
        std::string name = readLine("Potion name: ");
        int heal = readInt("Heal amount: ", 1);
        p.addItem(new Potion(name, heal)); // [REQ: Memory] Added to inventory for later deletion
    }
}

// ------------------------------------------------------------
// Menu: Use or equip an item
// ------------------------------------------------------------
static void useItemMenu(Player& p) {
    p.listItems();                  // [REQ: Functionality] Show inventory
    size_t idx = static_cast<size_t>(readInt("Enter index to use: ", 0));
    p.useItem(idx);                 // [REQ: Polymorphism] Calls virtual use() dynamically
}

// ------------------------------------------------------------
// Menu: Remove an item manually
// ------------------------------------------------------------
static void removeItemMenu(Player& p) {
    p.listItems();                  // [REQ: Functionality]
    size_t idx = static_cast<size_t>(readInt("Enter index to remove: ", 0));
    if (!p.removeItemByIndex(idx))  // [REQ: Functionality]
        std::cout << "Invalid index.\n";
}

// ------------------------------------------------------------
// Main program entry point
// ------------------------------------------------------------
int main() {
    Player player("Hero");          // [REQ: Functionality] Create Player object on stack

    // Demonstrate stack vs heap usage
    player.stackVsHeapDemo();       // [REQ: Stack/Heap] Demonstrates difference in memory allocation

    int choice = -1;
    while (choice != 0) {           // [REQ: Functionality] Continuous menu loop
        std::cout << "\n=== RPG Inventory ===\n";    // [REQ: I/O] User interface
        std::cout << "Player: " << player.getName()
                  << " HP=" << player.getHealth()
                  << " ATK=" << player.attack()
                  << " DEF=" << player.defense() << "\n"; // [REQ: Functionality] Show current stats
        std::cout << "1) Add item\n";              // [REQ: Menu]
        std::cout << "2) Show inventory\n";        // [REQ: Menu]
        std::cout << "3) Use/equip item\n";        // [REQ: Menu]
        std::cout << "4) Remove item\n";           // [REQ: Menu]
        std::cout << "0) Exit\n> ";                // [REQ: Menu]

        if (!(std::cin >> choice)) {               // [REQ: Quality] Prevent invalid input
            clearInput();
            continue;
        }

        switch (choice) {                          // [REQ: Functionality] Menu logic
            case 1: addItemMenu(player); break;    // [REQ: Functionality]
            case 2: player.listItems(); break;     // [REQ: Functionality]
            case 3: useItemMenu(player); break;    // [REQ: Functionality]
            case 4: removeItemMenu(player); break; // [REQ: Functionality]
            case 0: break;                         // [REQ: Functionality] Exit loop
            default: std::cout << "Invalid choice.\n"; break;
        }
    }

    // [REQ: Memory] Player destructor automatically frees all heap-allocated items
    return 0;  
}
