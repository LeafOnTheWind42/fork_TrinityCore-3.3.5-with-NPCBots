#### Summary: 
This goal of this fork of [TrinityCore-3.3.5-with-NPCBots](https://github.com/trickerer/TrinityCore-3.3.5-with-NPCBots) by trickerer to merge in  [TransmogDisplayVendor](https://github.com/Rochet2/TrinityCore/tree/transmogvendor_3.3.5/src/server/scripts/Custom/TransmogDisplayVendor) by Rochet2, to then modify it to get transmogrified gear appearances to display on the NPCBots.

Now the goal is to add more tweaks to my own forked version to learn and change things to my taste.

See the parent repos' ReadMes for more detail, including how to install.  This fork's ReadMe will just discuss changes.


## Transmog

It looks like the parent branch has implemented some NPCBot transmog capabilities.  The difference is, this code can allow an item transmogged by the transmog vendor to be equipped onto a bot and the transmog appearance will be applied.  The parent version don't require a transmog vendor, you just apply the transmog to the item the bot has equipped via chatting with the bot, though it can only select appearances from items currently in your inventory.

Upon originally creating this fork, the following was needed to get the TransmogDisplayVendor to work.  I'll need to verify if this is up-to-date after all the various changes to the two parent repos.

You do need to make sure the /src/server/scripts/Custom/TransmogDisplayVendor/sql/updates/characters_update_6_1_to_6_2.sql file gets run once, whether by the worldserver auto updater running it, or running them some other other way.

Adding a Transmog Vendor: When I first merged the TransmogrificationDisplayVendor code in, then logged into the world, I didn't know where to find a transmog vendor.  To create one, log in as a GM character, move to the location you want to place the vendor, face the direction you want the vendor to face, then run this GM command in chat:

.npc add 190011

If you have both the Reforging and TransmogrificationDisplayVendor by Rochet2 implemented, then 190011 is the reforger NPC, below would be for the transmog NPC:

.npc add 190012

Usage
Equip an item that is suitable for transmogrification.
Talk to Transmogrifier and select the item slot. Then select the quality and then the item you want to transmogrify to.

# Some notes on the transmog code 

The NPCBots code uses the WorldSession::HandleMirrorImageDataRequest function in /src/server/game/Handlers/SpellHandler.cpp to manage the display of the NPCBots.  

This fork has a function called GetNPCBotTransmogDisplayId defined in botdatamgr.cpp.  If a transmog has been applied to an item that is passed in to that function, it will return the item_template.displayid for the transmog, otherwise the item_template.displayid for the passed-in item is returned.  This will cause items for slots defined in the HandleMirrorImageDataRequest function's botItemSlots array to display as the transmog item (if applicable) on the NPCBot.  That array doesn't contain weapons, so I haven't been able to get weapon transmogs to appear on the NPCBots yet.


#### To do items:
1. Try to determine how to show weapon transmogs on the NPCBots.  I'm going to try to get SMSG_MIRRORIMAGE_DATA packet parses to see if that helps.
2. Try to modify the TransmogDisplayVendor to be able to transmog items in a player's bag (currently it transmogs items equipped on the player).  This will allow, for example, a cloth-wearing player to transmog a plate item in their bags, which they can they have their plate-wearing NPCBot equip.
3. Unrelated to transmog, I want to put a cap on the number of NPCBots of each class that can be spawned into a battleground.  Currently it is randomized.  After having 7 hunters on the opposing team in Arathi Basin, I decided to put a cap on this.  I'll keeping track of the number of bots of each class spawned into the BG.  The cap will likely be the number of spaces that need filled with bots integer divided by 5.  If the class of the next bot selected in the loop already has reached that cap, it will skip that bot and continue to the next iteration e.g. in AB if all 15 slots need to be filled by bots, then 15 // 5 = 3 is the cap for any one class.

- **Version 0.1** (_06 Oct 2022_)
    - Make transmogged armor appearance show up on NPCBots

>>>>>>> upstream/npcbots_3.3.5