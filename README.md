# ModCommAutoRegister
 Ark Survival Evolved assets for use in creating mods. This provides a base actor BP for creating a singleton that will auto register with another mod's singleton, assuming it is also using the same auto registration process. By registering with each other, the mods will then be able to ensure that requests on the Mod Communication interface are only allowed to modify info relevant to them, via their mod parent folder compared with class paths.

The contents of this repo are setup to test this functionality in PIE. Each folder is intended to reside under the /Mods folder directly. 

*NOTE: This is relating to making your mod communicate with other mods via Wildcard's Mod Communication Interface. Not for beginners.*

The intent with this is to hopefully spark discussion around mod comm integration in general by providing a way for mods to automatically register with each other in live game using a common, simple handshake process. If you have any feedback, questions, etc please ping me on the Ark Modding Discord or send DM. @Koz

## Contents
- **/ModCommRegInitiator & /ModCommRegRecipient**
  - Each intended to represent separate mod for testing
  - Both contain same base actor BP as well an example mod comm singleton for testing 
- **/ModCommTest**
  - Contains test PGD & Level for PIE 
- **/ModCommRegInitiator/ModCommReg_Base & /ModCommRegRecipient/ModCommReg_Base**
  - Base Actor BP to enable Auto Registration process in your Mod
  - *this is all you need to copy to your mod for use*

## First, The Duh
1. Download this source
2. [Unblock](https://lmgtfy.com/?q=unblock+windows+file) the .asset files in Windows File Explorer
3. Place assets in your /Mods folder *(best if ADK is closed when you do this)*

## How To Run in PIE 
1. Open ADK 
2. Open the level /Mods/ModCommTest/TestMapArea_ModCommReg
3. Run PIE *(possibly need to compile assets first)*
4. Look at the Output window to see the log output / tests results
5. Compare with PNG in this repo: Tests_Log.PNG 
6. *Note: If you examine properties on the Initiator/Recipient after tests have finished their arrays will be empty and not match the other PNG images in the repo. Those images were taken with the final revoked test (in the Initiator) unpinned because that clears the registration info for both mods.*

## How To Add to Your Mod
1. Open ADK 
2. Copy to the base actor BP to your mod: /Mods/ModCommRegInitiator/ModCommReg_Base
3. *Rename if desired,* Save/Compile the new copied asset
4. Create BP Child of the copied ModCommReg_Base, Rename as desired, Save/Compile the child 
5. Go to BP Defaults, modify 'CAL Tag' to be your own value. This will be your mod's tag identifier with other mods, use a relevant value. 
6. Add the new BP to your mod PGD's Singleton Array. 
7. *Alernatively, if you already have a singleton, you could re-parent to use ModCommReg_Base*

## Notes On Use
- ModCommReg_Base uses function Begin Play 
- Make sure you add call to parent Begin Play if using that override in your child BP 
- You should not implement Mod Communication Interfaces RequestModDataProcessing or SendModData when using this base BP as parent. Instead add overrides for these as desired/needed: 
  - Will run if other mod calls mod comm interface and has already completed registration. This *should* be all that is needed since mods should register before trying to use your mod's interface calls.
    - VerifiedRequestModDataProcessing 
    - VerifiedSendModData
  - Will run if other mod calls mod comm interface too earlier, without allowing for the delay needed during registration (0.05s). 
    - PendingVerifyRequestModDataProcessing 
    - PendingVerifySendModData 
  - Will run if other mod calls mod comm interface before attempting registration, or if they sent an unrecognized key such as possibly having a bug on their end. 
    - UnknownRequestModDataProcessing 
    - UnknownSendModData
- By default ModCommReg_Base does not perform handling for the RequestModData interface. 
  - If you want to enforce registration for this interface, enable the BP Default 'bRequest Mod Data Verify' 
  - Mods calling this interface will then need to send the secret in the Key param 
  - If your mod only has one thing to request data for then there is no need but...
  - If your mod has multiple data requests to handle for then the calling mod needs to delimit the secret with the type of data request in the Key param. 
  - In this case the secret is expected to come first 
- Depending on needs you can implement as Initiator or Recipient or both, see below. 
  - The example Initiator/Recipient BPs in this repo only implement the one relevant to them. 

## How To Use as Initiator
1. You need to know what the other mod's CAL Tag is. Developer would provide this.
2. Then you just call 'Begin Register Other Mod' and pass the other mod's CAL Tag. 
3. *There is a small delay required in order to complete the registration process (0.05s)*
4. This means you need to wait before trying to send any Mod Communication Interface calls to the other mod. 
5. You can bind on the event 'OnModRegistrationVerified' to know that the handshake has at least completed on your end. This means the other mod has called back to you successfully. 
6. You still need slight delay after the verified event to allow for the other mod to receive your success reply on the callback and complete registration on their end. 
7. When calling RequestModDataProcessing/SendModData interfaces the Key needs to be from the 'My Mod Keys' array for the registered mod's index. 
8. Use 'Lookup Mod Registration' with other mod's CAL tag to find the index, then use that with Get on My Mod Keys array. That goes to the Key param on the interface calls. 
9. Since the Key is used to let the other mod know who you are, you will need to provide within the In Json the nature of your request, if multiple types of requests are possible. Such as a 'command' field in the json etc. 

## How To Use as Recipient
1. Implement overrides for the 'Verified' functions mentioned in Notes On Use above. 
2. That's it. Handle the requests as needed. 




