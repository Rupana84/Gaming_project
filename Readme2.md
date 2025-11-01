#ifndef ITEM_H
#define ITEM_H

#include <string>

 Item.h 
 ------------
// Forward declaration of Player class
// We only tell the compiler that a class named Player exists
// because some Item functions will take a Player reference.
class Player;

// The Item class is a base class for all game items.
// It defines the general interface (what all items can do).
class Item {
public:
    // Virtual destructor so deleting derived objects through
    // an Item pointer works safely.
    virtual ~Item() = default;

    // Each item must have a name.
    virtual std::string getName() const = 0;

    // Each item has a type (Weapon, Armor, Potion, etc.).
    virtual std::string getType() const = 0;

    // Returns a text description of the item.
    virtual std::string describe() const = 0;

    // Defines what happens when the player uses the item.
    virtual void use(Player& player) = 0;
};
----------------------
Weapon.h
----------------------

#ifndef WEAPON_H
#define WEAPON_H

#include "Item.h"
#include <string>

// The Weapon class represents any weapon the player can use.
// It inherits from the base class Item and adds weapon-specific details.
class Weapon : public Item {
    std::string name;   // The weapon's name, e.g., "Sword"
    int damage;         // How much damage this weapon can deal
    bool equipped{false}; // Tracks whether the weapon is currently equipped

public:
    // Constructor: sets the name and damage when a weapon is created.
    Weapon(const std::string& n, int dmg) : name(n), damage(dmg) {}

    // Returns the weapon's name.
    std::string getName() const override { return name; }

    // Returns the item type as "Weapon".
    std::string getType() const override { return "Weapon"; }

    // Returns a short description of the weapon.
    std::string describe() const override;

    // Defines what happens when the player uses the weapon.
    // In this case, it equips the weapon.
    void use(Player& player) override;

    // Returns how much damage this weapon provides.
    int getDamage() const { return damage; }

    // Checks if this weapon is equipped.
    bool isEquipped() const { return equipped; }

    // Sets the equipped status.
    void setEquipped(bool v) { equipped = v; }
};

#endif // WEAPON_H

#endif // ITEM_H
----------------------
Weapon.cpp 
----------------------
#include "Weapon.h"
#include "Player.h"
#include <sstream>

// Builds a text description of the weapon.
// Example output: "Sword [Weapon] dmg=10 (equipped)"
std::string Weapon::describe() const {
    std::ostringstream os;
    os << name << " [Weapon] dmg=" << damage;

    // Add a note if the weapon is currently equipped.
    if (equipped)
        os << " (equipped)";

    return os.str();
}

// Called when the player uses this weapon.
// Instead of consuming it, it tells the Player to equip this weapon.
void Weapon::use(Player& player) {
    player.equipWeapon(this);
}

-----------------------
Armor.h
-----------------------
#ifndef ARMOR_H
#define ARMOR_H

#include "Item.h"
#include <string>

// The Armor class represents protective gear the player can wear.
// It inherits from Item and adds defense-related attributes.
class Armor : public Item {
    std::string name;   // The armor's name, e.g., "Steel Shield"
    int defense;        // How much protection it provides
    bool equipped{false}; // True if this armor is currently being worn

public:
    // Constructor: sets the armor name and defense value when created.
    Armor(const std::string& n, int def) : name(n), defense(def) {}

    // Returns the armor's name.
    std::string getName() const override { return name; }

    // Returns the item type as "Armor".
    std::string getType() const override { return "Armor"; }

    // Builds and returns a text description of the armor.
    std::string describe() const override;

    // Defines what happens when the player uses this armor.
    // It equips the armor.
    void use(Player& player) override;

    // Returns how much defense this armor adds.
    int getDefense() const { return defense; }

    // Checks if the armor is equipped.
    bool isEquipped() const { return equipped; }

    // Sets the equipped state (true or false).
    void setEquipped(bool v) { equipped = v; }
};

#endif // ARMOR_H

------------------------
Armor.cpp
-------------------------
#include "Armor.h"
#include "Player.h"
#include <sstream>

// Returns a string description of the armor.
// Example: "Shield [Armor] def=5 (equipped)"
std::string Armor::describe() const {
    std::ostringstream os;
    os << name << " [Armor] def=" << defense;

    // Add "(equipped)" if this armor is currently worn.
    if (equipped)
        os << " (equipped)";

    return os.str();
}

// Defines what happens when the player uses this armor.
// When used, it tells the Player to equip this armor.
void Armor::use(Player& player) {
    player.equipArmor(this);
}

-----------------
 Potion.h
 -----------------

 #ifndef POTION_H
#define POTION_H

#include "Item.h"
#include <string>

// The Potion class represents a consumable item that heals the player.
// It inherits from Item and adds a healing effect.
class Potion : public Item {
    std::string name;   // The potion's name, e.g., "Health Potion"
    int heal;           // How many health points it restores

public:
    // Constructor: sets the potion name and healing amount.
    Potion(const std::string& n, int h) : name(n), heal(h) {}

    // Returns the potion's name.
    std::string getName() const override { return name; }

    // Returns the item type as "Potion".
    std::string getType() const override { return "Potion"; }

    // Builds a text description for displaying in the inventory.
    std::string describe() const override;

    // Defines what happens when the player uses this potion.
    // It heals the player and removes itself from the inventory.
    void use(Player& player) override;

    // Returns how much health this potion restores.
    int amount() const { return heal; }
};

#endif // POTION_H

******
Potion.cpp
******

#include "Potion.h"
#include "Player.h"
#include <sstream>

// Returns a text description of the potion.
// Example output: "Health Potion [Potion] heal=20"
std::string Potion::describe() const {
    std::ostringstream os;
    os << name << " [Potion] heal=" << heal;
    return os.str();
}

// Defines what happens when the player uses this potion.
// When used, it heals the player by 'heal' points
// and then removes itself from the player's inventory.
void Potion::use(Player& player) {
    player.addHealth(heal);          // Increases player's health
    player.consumeCurrentItem(this); // Removes and deletes this potion
}

************
Player.h
************
#ifndef PLAYER_H
#define PLAYER_H

#include "Item.h"
#include <vector>
#include <string>
#include <iostream>

// Forward declarations so we can use these types later in the file
class Weapon;
class Armor;

// The Player class represents a game character that owns and manages items.
// It can equip weapons, wear armor, and use potions.
class Player {
    std::string name;               // The player's name
    int health{100};                // Player's current health points
    int baseAttack{5};              // Base attack power without a weapon
    int baseDefense{2};             // Base defense without armor

    Weapon* equippedWeapon{nullptr}; // Pointer to currently equipped weapon
    Armor*  equippedArmor{nullptr};  // Pointer to currently equipped armor
    std::vector<Item*> inventory;    // List of all items the player owns

public:
    // Constructor: create a player with a given name.
    explicit Player(const std::string& n) : name(n) {}

    // Destructor: called automatically when the Player object is destroyed.
    // It deletes all dynamically allocated items in the inventory.
    ~Player();

    // Prevent copying or moving Player objects (to avoid double-deleting items).
    Player(const Player&) = delete;
    Player& operator=(const Player&) = delete;
    Player(Player&&) = delete;
    Player& operator=(Player&&) = delete;

    // -----------------------------
    // Player stats and health
    // -----------------------------
    void addHealth(int v);          // Increases or decreases the player's health
    int attack() const;             // Returns total attack power (base + weapon)
    int defense() const;            // Returns total defense (base + armor)
    int getHealth() const { return health; } // Returns current health
    std::string getName() const { return name; } // Returns player's name

    // -----------------------------
    // Inventory management
    // -----------------------------
    void addItem(Item* item);             // Adds a new item to inventory
    void listItems() const;               // Prints all items to the console
    bool removeItemByIndex(size_t idx);   // Removes and deletes an item by index
    Item* getItem(size_t idx);            // Returns a pointer to an item

    // -----------------------------
    // Using and equipping items
    // -----------------------------
    void useItem(size_t idx);        // Uses an item (Weapon, Armor, or Potion)
    void equipWeapon(Weapon* w);     // Equips a weapon
    void equipArmor(Armor* a);       // Equips armor
    void consumeCurrentItem(Item* item); // Removes a used potion from inventory

    // -----------------------------
    // Demonstration function
    // -----------------------------
    void stackVsHeapDemo();          // Shows the difference between stack and heap memory
};

#endif // PLAYER_H

**********
Player.cpp
**********

#include "Player.h"
#include "Weapon.h"
#include "Armor.h"
#include "Potion.h"
#include <iomanip>

// Destructor — runs automatically when a Player object is destroyed.
// It makes sure that all items stored on the heap are properly deleted.
Player::~Player() {
    for (Item* it : inventory) {
        delete it;      // Free each heap-allocated item
    }
    inventory.clear();  // Empty the inventory after deletion
}

// Increases or decreases the player's health.
void Player::addHealth(int v) {
    health += v;                 // Add or subtract health
    if (health > 100) health = 100;  // Cap health at 100
    if (health < 0) health = 0;      // Prevent negative health
}

// Calculates total attack power.
// Adds the weapon’s damage to the base attack if one is equipped.
int Player::attack() const {
    int bonus = 0;
    if (equippedWeapon)
        bonus += equippedWeapon->getDamage();
    return baseAttack + bonus;
}

// Calculates total defense.
// Adds armor’s defense value to the base defense if one is equipped.
int Player::defense() const {
    int bonus = 0;
    if (equippedArmor)
        bonus += equippedArmor->getDefense();
    return baseDefense + bonus;
}

// Adds a new item (Weapon, Armor, or Potion) to the player’s inventory.
void Player::addItem(Item* item) {
    inventory.push_back(item);
}

// Displays all items currently in the player’s inventory.
void Player::listItems() const {
    if (inventory.empty()) {
        std::cout << "Inventory is empty.\n";
        return;
    }

    // Loop through all items and print their descriptions.
    for (size_t i = 0; i < inventory.size(); ++i) {
        std::cout << std::setw(2) << i << ": " << inventory[i]->describe() << "\n";
    }
}

// Removes an item from inventory using its index and deletes it from memory.
bool Player::removeItemByIndex(size_t idx) {
    if (idx >= inventory.size())
        return false;   // Invalid index

    Item* it = inventory[idx];

    // Unequip if the item being removed is currently equipped.
    if (it == equippedWeapon) equippedWeapon = nullptr;
    if (it == equippedArmor)  equippedArmor  = nullptr;

    delete it;  // Free memory
    inventory.erase(inventory.begin() + idx);  // Remove from vector
    return true;
}

// Returns a pointer to an item by its index, or nullptr if invalid.
Item* Player::getItem(size_t idx) {
    if (idx >= inventory.size())
        return nullptr;
    return inventory[idx];
}

// Uses an item based on its index in the inventory.
// Calls the correct version of "use()" depending on the item type.
void Player::useItem(size_t idx) {
    Item* it = getItem(idx);
    if (!it) {
        std::cout << "Invalid index.\n";
        return;
    }
    it->use(*this);   // Calls the derived class's use() function
}

// Equips a new weapon and unequips any previously equipped one.
void Player::equipWeapon(Weapon* w) {
    if (equippedWeapon)
        equippedWeapon->setEquipped(false);

    equippedWeapon = w;
    if (equippedWeapon)
        equippedWeapon->setEquipped(true);

    std::cout << "Equipped weapon: " << w->getName()
              << ". Attack=" << attack() << "\n";
}

// Equips a new armor and unequips any previously equipped one.
void Player::equipArmor(Armor* a) {
    if (equippedArmor)
        equippedArmor->setEquipped(false);

    equippedArmor = a;
    if (equippedArmor)
        equippedArmor->setEquipped(true);

    std::cout << "Equipped armor: " << a->getName()
              << ". Defense=" << defense() << "\n";
}

// Removes a used potion from the inventory and deletes it.
void Player::consumeCurrentItem(Item* item) {
    for (size_t i = 0; i < inventory.size(); ++i) {
        if (inventory[i] == item) {
            delete inventory[i];                     // Free potion memory
            inventory.erase(inventory.begin() + i);  // Remove from list
            std::cout << "Potion consumed and removed.\n";
            return;
        }
    }
}

// Demonstrates the difference between stack and heap allocation.
void Player::stackVsHeapDemo() {
    // This potion is created on the stack — it is automatically destroyed
    // when this function ends.
    Potion stackPotion("Stack Tonic", 5);
    std::cout << "Stack object: " << stackPotion.describe() << "\n";

    // This potion is created on the heap — it stays alive until we delete it.
    Item* heapPotion = new Potion("Heap Elixir", 7);
    std::cout << "Heap object: " << heapPotion->describe()
              << " (added to inventory)\n";

    // Add the heap-allocated potion to the player's inventory.
    // The Player destructor will delete it later.
    addItem(heapPotion);
}

**********
main.cpp
**********

#include <iostream>
#include <limits>
#include <climits>
#include "Player.h"
#include "Weapon.h"
#include "Armor.h"
#include "Potion.h"

// ------------------------------------------------------------
// Helper functions for safe and clean user input
// ------------------------------------------------------------

// Clears any bad input from the console so the next input works correctly.
static void clearInput() {
    std::cin.clear();
    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
}

// Reads an integer from the user with optional minimum and maximum values.
static int readInt(const std::string& prompt, int minV = INT_MIN, int maxV = INT_MAX) {
    int v;
    while (true) {
        std::cout << prompt;
        if (std::cin >> v && v >= minV && v <= maxV)
            return v;  // Valid number entered
        std::cout << "Please enter a valid number";
        if (minV != INT_MIN || maxV != INT_MAX)
            std::cout << " (" << minV << "–" << maxV << ")";
        std::cout << ".\n";
        clearInput();  // Retry input
    }
}

// Reads a full line of text from the user (for names, etc.)
static std::string readLine(const std::string& prompt) {
    std::cout << prompt;
    clearInput();
    std::string s;
    std::getline(std::cin, s);
    return s;
}

// ------------------------------------------------------------
// Menu function: Add a new item to the player’s inventory
// ------------------------------------------------------------
static void addItemMenu(Player& p) {
    int c = readInt("1) Weapon  2) Armor  3) Potion  0) Cancel\n> ", 0, 3);
    if (c == 0)
        return;  // Exit if user cancels

    if (c == 1) {
        std::string name = readLine("Weapon name: ");
        int dmg = readInt("Damage: ", 0);
        p.addItem(new Weapon(name, dmg));  // Create and add a new Weapon
    } else if (c == 2) {
        std::string name = readLine("Armor name: ");
        int def = readInt("Defense: ", 0);
        p.addItem(new Armor(name, def));   // Create and add a new Armor
    } else {
        std::string name = readLine("Potion name: ");
        int heal = readInt("Heal amount: ", 1);
        p.addItem(new Potion(name, heal)); // Create and add a new Potion
    }
}

// ------------------------------------------------------------
// Menu function: Use or equip an item
// ------------------------------------------------------------
static void useItemMenu(Player& p) {
    p.listItems();  // Show all current items
    size_t idx = static_cast<size_t>(readInt("Enter index to use: ", 0));
    p.useItem(idx); // Use the selected item
}

// ------------------------------------------------------------
// Menu function: Remove an item manually
// ------------------------------------------------------------
static void removeItemMenu(Player& p) {
    p.listItems();
    size_t idx = static_cast<size_t>(readInt("Enter index to remove: ", 0));
    if (!p.removeItemByIndex(idx))
        std::cout << "Invalid index.\n";
}

// ------------------------------------------------------------
// Main program — handles game setup and menu loop
// ------------------------------------------------------------
int main() {
    // Create a Player object named "Hero"
    Player player("Hero");

    // Show a small demo of stack vs heap memory usage
    player.stackVsHeapDemo();

    int choice = -1;

    // Main game menu loop
    while (choice != 0) {
        std::cout << "\n=== RPG Inventory ===\n";
        std::cout << "Player: " << player.getName()
                  << " HP=" << player.getHealth()
                  << " ATK=" << player.attack()
                  << " DEF=" << player.defense() << "\n";
        std::cout << "1) Add item\n";
        std::cout << "2) Show inventory\n";
        std::cout << "3) Use/equip item\n";
        std::cout << "4) Remove item\n";
        std::cout << "0) Exit\n> ";

        if (!(std::cin >> choice)) {
            clearInput();  // Handle invalid input
            continue;
        }

        // Perform the action the user selected
        switch (choice) {
            case 1: addItemMenu(player); break;   // Add a new item
            case 2: player.listItems(); break;    // Show all items
            case 3: useItemMenu(player); break;   // Use an item
            case 4: removeItemMenu(player); break; // Remove an item
            case 0: break;                        // Exit the game
            default: std::cout << "Invalid choice.\n"; break;
        }
    }

    // When the program ends, the Player destructor deletes all items.
    return 0;
}
