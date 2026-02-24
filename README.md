# Creative Filter Patch

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

## 🧩 Designed for
[![Nukkit-MOT](https://avatars.githubusercontent.com/u/62042238?s=48&v=4)](https://github.com/MemoriesOfTime/Nukkit-MOT)

---

## ⚠️ The Problem

Modifying the creative content via `CreativeContentPacket` is unreliable and often crashes the client.
<br>(by EventListener in your plugin)

## ✨ The Solution
This patch implements a **permission-based filtering system** for creative items. It allows server administrators and plugin developers to control which items appear in the creative menu based on player permissions, without breaking client compatibility.

### Key Features
*   **Permission-Based Filtering:** Hide specific items or groups of items from players.
*   **Plugin API:** Easily register custom item groups and define complex logic via Java predicates.
*   **Safe Integration:** Modifies minimally to prevent client crashes while maintaining full functionality.
*   **Quick-Crafts Support:** Adjusts to the game mode and sends all creative items (if survival).

---

## 🚀 How to Apply

1.  **Clone the patch repository:**
    ```bash
    git clone https://github.com/labarjni/Nukkit-MOT-CF-Patch.git
    ```

2.  **Navigate to the directory:**
    ```bash
    cd Nukkit-MOT-CF-Patch
    ```

3.  **Clone the target Nukkit-MOT repository:**
    ```bash
    git clone https://github.com/MemoriesOfTime/Nukkit-MOT.git
    ```

4.  **Apply the patches:**
    ```bash
    ./gradlew applyPatches
    ```

✅ **Done!** You can now build Nukkit-MOT (`./gradlew build`) and the creative filter system will be active.

---

## 🛠️ What's Inside? (Patch Summary)

This patch introduces a new class `ItemCreativePermissions` and modifies `Item.java` and `PlayerInventory.java` to intercept creative item list generation.

### Core Changes:
1.  **New Class:** `cn.nukkit.item.ItemCreativePermissions`
    *   Manages a registry of `Predicate<Item>` linked to permission nodes.
    *   Base permission format: `nukkit.creativeitem.<suffix>`
2.  **Modified `Item.java`:**
    *   Overloaded `getCreativeItems()` and `getCreativeItemsAndGroups()` to accept a `Player` instance.
    *   Filters the item list dynamically based on the player's permissions before sending it to the client.
    *   Added `getCreativeItemsAndGroupsRaw()` to access the unfiltered list internally.
3.  **Modified `PlayerInventory.java`:**
    *   Updated the creative content packet logic to pass the `Player` object to the item retrieval methods, enabling per-player filtering.

---

## 📖 Plugin Developer API

You can integrate this system into your own plugins to restrict access to specific items (e.g., admin-only tools).
<br>Or create specific tabs with items of different accessibility

### 1. Registering a Permission Group

Use `ItemCreativePermissions.registerPermissionGroup` to define items and the permission required to see them.

**Example: Restricting Bedrock to Admins**

```java
import cn.nukkit.item.Item;
import cn.nukkit.plugin.PluginBase;

public class MyPlugin extends PluginBase {

    @Override
    public void onEnable() {
        ItemCreativePermissions.registerPermissionGroup("admin.bedrock", item -> item.getId() == Item.BEDROCK);

        getLogger().info("Bedrock is now restricted to players with 'nukkit.creativeitem.admin.bedrock'");
    }
}
```

Players will need the permission node `nukkit.creativeitem.admin.bedrock` to see Bedrock in the creative.

### 2. Complex Logic (Multiple Items)

You can check for multiple IDs, meta values, or even custom names.

**Example: Restricting all "Command Blocks" and "Structure Blocks"**

```java
import cn.nukkit.item.Item;

ItemCreativePermissions.registerPermissionGroup("builder.structure",item ->{
    int id = item.getId();
    return id == Item.COMMAND_BLOCK ||
        id == Item.CHAIN_COMMAND_BLOCK ||
        id == Item.REPEATING_COMMAND_BLOCK ||
        id == Item.STRUCTURE_BLOCK;
});
```

*Required permission: `nukkit.creativeitem.builder.structure`*

### 3. Using Custom Predicates for Dynamic Groups

Since the condition is a `java.util.function.Predicate<Item>`, you can implement any logic you want.

**Example: Hiding items with a specific custom name (e.g., "Dev Tool")**

```java
import cn.nukkit.item.Item;

ItemCreativePermissions.registerPermissionGroup("dev.tools",item ->{
        if (!item.hasCustomName()) return false;
        return item.getCustomName().contains("[DEV]");
});
```

*Required permission: `nukkit.creativeitem.dev.tools`*

### 4. Direct Usage in Plugin Logic (`hasPermission`)

Besides registering global groups, you can manually check if a specific player is allowed to have a specific item using `ItemCreativePermissions.hasPermission(Item, Player)`. This is useful for custom GUIs, event listeners, or command executors.

**Example: Blocking item pickup in Creative mode via Event Listener**

```java
import cn.nukkit.event.EventHandler;
import cn.nukkit.event.Listener;
import cn.nukkit.event.player.PlayerInventoryTransactionEvent;
import cn.nukkit.item.Item;
import cn.nukkit.plugin.PluginBase;

public class CreativeGuard extends PluginBase implements Listener {

    @Override
    public void onEnable() {
        getServer().getPluginManager().registerEvents(this, this);
    }

    @EventHandler
    public void onTransaction(PlayerInventoryTransactionEvent event) {
        if (!event.getPlayer().isCreative()) {
            return;
        }

        Item item = event.getTransaction().getSourceItem();

        if (!ItemCreativePermissions.hasPermission(item, event.getPlayer())) {
            event.setCancelled();
            event.getPlayer().sendMessage("§cYou do not have permission to use this item");
        }
    }
}
```

### 📝 Permission Node Format
When you register a group with the suffix `my.suffix`, the system automatically requires:
`nukkit.creativeitem.my.suffix`

If the player **does not** have this permission, any item matching your predicate will be **hidden** from their creative.

---

## ⚠️ Notes & Credits

*   **Gradle Structure:** Thanks to [LuminiaDev/Nukkit-MOT-UUID-Patch](https://github.com/LuminiaDev/Nukkit-MOT-UUID-Patch) for the gradle patch structure.
*   **Issues:** If the patches do not work, please create a [new issue](https://github.com/labarjni/Nukkit-MOT-CF-Patch/issues/new) with a description of the problem.