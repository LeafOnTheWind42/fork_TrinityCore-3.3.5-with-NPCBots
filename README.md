# Summary: 

This goal of this fork of [TrinityCore-3.3.5-with-NPCBots](https://github.com/trickerer/TrinityCore-3.3.5-with-NPCBots) by trickerer was originally to merge in  [TransmogDisplayVendor](https://github.com/Rochet2/TrinityCore/tree/transmogvendor_3.3.5/src/server/scripts/Custom/TransmogDisplayVendor) by Rochet2, to then modify it to get transmogrified gear appearances to display on the NPCBots.  That has now been accomplished and seems to be working.

Now the goal is to add more tweaks to my own forked version to learn the code base and change things to my taste.

See the parent repos' ReadMes for more detail, including how to install.  This fork's ReadMe will just discuss changes.

This repo also has [Reforging](https://github.com/Rochet2/TrinityCore/tree/reforging_3.3.5/src/server/scripts/Custom/Reforging) by Rochet2 merged into it.


## Transmog

It looks like the parent branch has implemented some NPCBot transmog capabilities, which allows a player to chat with the bot and have it transmogs items they have equipped with appearances from items in the player's bags.  This code can allow an item transmogged by the TransmogDisplayVendor to be equipped onto a bot and the transmog appearance will be applied.  So you get a wider selection of appearance for the item from the vendor, plus the transmog works on the player if they equip the item too.

You do need to make sure the /src/server/scripts/Custom/TransmogDisplayVendor/sql/world_NPC.sql and /src/server/scripts/Custom/TransmogDisplayVendor/sql/updates/characters_update_6_1_to_6_2.sql file gets run once, whether by the worldserver auto updater running it, or running them some other other way.

I tried to make sure to add comments to code I added or modified, specifically mentioning "zzTransmogCompatibility".

Before you compile, you can optionally tweak the config settings in src/server/game/Entities/Item/TransmogDisplayVendor.cpp, which determines which types of transmog a player and item can receive.  I have the settings pretty wide open already, with a few exceptions:

>// A multiplier for the default gold cost (change to 0.0f for no default cost)  
>const float TransmogDisplayVendorMgr::ScaledCostModifier = 1.0f;  
>// Cost added on top of other costs (can be negative)  
>const int32 TransmogDisplayVendorMgr::CopperCost = 0;  
>// For custom gold cost set ScaledCostModifier to 0.0f and CopperCost to what ever cost you want
>
>const bool TransmogDisplayVendorMgr::RequireToken = false;  
>const uint32 TransmogDisplayVendorMgr::TokenEntry = 49426;  
>const uint32 TransmogDisplayVendorMgr::TokenAmount = 1;  
>
>const bool TransmogDisplayVendorMgr::AllowPoor = false;  
>const bool TransmogDisplayVendorMgr::AllowCommon = false;  
>const bool TransmogDisplayVendorMgr::AllowUncommon = true;  
>const bool TransmogDisplayVendorMgr::AllowRare = true;  
>const bool TransmogDisplayVendorMgr::AllowEpic = true;  
>const bool TransmogDisplayVendorMgr::AllowLegendary = false;  
>const bool TransmogDisplayVendorMgr::AllowArtifact = false;  
>const bool TransmogDisplayVendorMgr::AllowHeirloom = true;  
>
>const bool TransmogDisplayVendorMgr::AllowMixedArmorTypes = true;  
>const bool TransmogDisplayVendorMgr::AllowMixedWeaponTypes = false;  
>const bool TransmogDisplayVendorMgr::AllowMixedInventoryTypes = false;  
>const bool TransmogDisplayVendorMgr::AllowFishingPoles = false;  
>
>const bool TransmogDisplayVendorMgr::IgnoreReqRace = true;  
>const bool TransmogDisplayVendorMgr::IgnoreReqClass = true;  
>const bool TransmogDisplayVendorMgr::IgnoreReqSkill = true;  
>const bool TransmogDisplayVendorMgr::IgnoreReqSpell = true;  
>const bool TransmogDisplayVendorMgr::IgnoreReqLevel = true;  
>const bool TransmogDisplayVendorMgr::IgnoreReqEvent = true;  
>const bool TransmogDisplayVendorMgr::IgnoreReqStats = true;


### Adding a Transmog Vendor

When I first merged the TransmogrificationDisplayVendor code in, then logged into the world, I didn't know where to find a transmog vendor.  To create one, log in as a GM character, move to the location you want to place the vendor, face the direction you want the vendor to face, then run this GM command in chat:

>.npc add 190012

### Usage

Equip an item that is suitable for transmogrification.
Talk to Transmogrifier and select the item slot. Then select the quality and then the item you want to transmogrify to.

### Interaction between NPCBot Dialog Transmog and TransmogDisplayVendor Transmog

TLDR:
1. If you transmog an item via TransmogDisplayVendor, then give that item to a bot; it will overwrite the NPCBot dialog transmog, if any, that bot already had for that item.
2. If an item is already transmogged by the TransmogDisplayVendor, trying to transmog that same item instace via NPCBot dialog menu will tell you it is already transmogrified.  If you still want to do the transmog via the the NPCBot menu, go to the TransmogDisplayVendor and remove the transmog, then retry.

Long version:
I had to figure out how to resolve any potential conflicts between the two ways to transmog an item, since each uses a different table in the characters database.  For items transmogged by the TransmogDisplayVendor, I was able to give an message to the player that it was already transmogged, since the vendor can transmog an item with more appearances.

I was going to apply similar logic if the player tried to transmog an item via the TransmogDisplayVendor that was already transmogged by the NPCBot dialog.  However, the way transmogs are stored in the characters_npcbot_transmog table, it isn't by item guid.  So either: 1. if any instance of that item, for any bot, has an NPCBot transmog, it would prevent the player from getting the transmog from the vendor (not a good solution) or 2. I could have checked to see which bots the player currently own, see if any of those bots has a transmog on that item template, then deny if so (a less bad option, but what if the player wants one of their bots to have the NPCBot dialog transmog, but a vendor transmog for another instance of that item for another bot).  


## Reforging

You do need to make sure the /src/server/scripts/Custom/Reforging/sql/world_npc.sql file gets run once, whether by the worldserver auto updater running it, or running them some other other way.

### Adding a Reforger Vendor

Log in as a GM character, move to the location you want to place the vendor, face the direction you want the vendor to face, then run this GM command in chat:

>.npc add 190011

### Usage

Equip an item that is suitable for reforging.
Talk to Reforger and select the item slot. Then select the reforging you want.


## Battleground Class Mix Balance

I placed a cap on the number of NPCBots of each class that can be spawned into a battleground.  Currently it is randomized.  After having 7 hunters on the opposing team in Arathi Basin, I decided to put a cap on this.  

The cap by default is 20% per this world.conf setting that you can change: NpcBot.WanderingBots.BG.MaxClassPercent.

It keeps track of the number of bots of each class spawned into the BG.  If the class of the next bot selected in the loop already has reached that cap, it will skip that bot and continue to the next iteration.  

Examples: 
* In AB if 13 slots need to be filled by bots, then round(13 * 20 / 100) = 3 is the cap for any one class
* If only 12 needed, then round(12 * 20 / 100) = 2

In order to avoid issues, the cap will be skipped if it would prevent the BG from being filled up completely.  In other words, if the pool of available wandering bots wouldn't be large enough to fill up the BG if the cap was implemented, no cap will be applied.  If that case you would want to add more wandering bots to balance out the classes more.

I tried to make sure to add comments to code I added or modified, specifically mentioning "zzBgBotClassLimit".


## To do list:

1. Try to modify the TransmogDisplayVendor to be able to transmog items in a player's bag (currently it transmogs items equipped on the player).  This will allow, for example, a cloth-wearing player to transmog a plate item in their bags, which they can they have their plate-wearing NPCBot equip.
2. Since this code already has Reforger patched in, I checked to see how the reforged item is handled by the bot.  It seems to be ignored based on the /bonk stats.  A goal is to figure out how the items affect the stats on the bot, to see how hard it would be to use the reforged value.  This might also help in figuring out how to get heirlooms to scale on bots, since I noticed their stats and performance are through the roof when I give them heirlooms.
3. It would be interesting to learn more about the logic that the NPCBots use to operate.  Along those lines I'd like to figure out how to get them to stop leaving resources undefended in AB.  Though I'd want to include some conditions where it is okay to abandon the resource, and instead laser focus on taking enemy-owned resources, such as when the team is down a lot of points and getting somewhat close to the end, where stopping the enemy team from accumulating any more points is a hail mary chance at winning.