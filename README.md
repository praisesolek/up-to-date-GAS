# up-to-date-GAS
A fork of Unreal Engine's [GameplayAbilitiesSystem](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/GameplayAbilitySystem/) (GAS) plugin that keeps up to date with all the changes in the repository that it can.

## Introduction
I work intensely with GameplayAbilities. I've always seen it as a powerful tool for creating multiplayer gameplay. However, it's very far-removed from Unreal Engine's main development plans and processes; it's one of those modules in the engine that just gets updated in the open-source and the changes just make it to the main releases. The result of this disconnect is a delay in my ability to use the new innovations and changes.
I like solving problems, and this one is actually shockingly easy to begin solving. Since I work in a compiled version of the engine already, so that I can make specialized server and client builds, I have granular access to the editor and engine code. The only thing stopping me now is the work behind going through an inordinate number of commits to the main Unreal repo and figuring out which things to keep and which things to leave out. More on that in a moment.
Before I get deep into the specifics of the changes made, I want to ward off any misconceptions and make sure we're all on the same page.
* This is only reasonably implementable in a [compiled editor instance](https://docs.unrealengine.com/4.27/en-US/ProductionPipelines/DevelopmentSetup/BuildingUnrealEngine/). This one's fairly self-explanatory since the versions installed from the Epic Games Launcher, when they update, could cause unpredictable behavior with edited code. NOTE: this is based off an assumption of software development to not mess with things that update and edit themselves without full knowledge of that, and I plan on doing more research in the future. This will be updated accordingly once that's done.
* Implementing this is still a fairly risky territory. These features are not yet released and any testing that needs to be done on them is sometimes not yet complete. I'm doing this for my own install because the ability to use up-to-date GAS features outweighs potential risks which I can still try and fix myself. I would *not* recommend this for people who won't be heavily using GAS.
* This is being implemented on my end by literally replacing every file in the GAS plugin folder in my engine code with the files here and adjusting tiny things in the other engine code as necessary. I have not tested this as a project plugin but I would not recommend that as some features are better off from going into engine code and altering/adding things there.

## Implementation
This section is fairly short. I mainly just want to elaborate on how I intend to do things.
* I am leaning closer to the side of self-contained changes to code. Things that depend on outside features to make sense, like GameplayCues supporting MetaSounds, is not making it into the plugin for 4.27.2 because, in that example, that's a separate module for 5.0 entirely. 
* An exception to the above rule is if the change is extremely minimal. For example, there is a [commit](https://github.com/EpicGames/UnrealEngine/commit/e24e5d7ae418dc80b97495f7fb157f043025dadf) that adds a single constant to `NetworkVersion.h` to work. I saw that as wildly unobtrusive and trivial so I just did it.

## Diff list *(still a work-in-progress to retroactively document all of this)*
This is a section dedicated to describing the differences between code in the current iteration of the 5.0 branch and this repo. Note that commits made that don't have to be adjusted to work on 4.27.2 won't show up here. This is only to detail commits I modified to retrofit into 4.27.2.
* There's a change to a more lightweight representation of Actors in memory and the TargetActor classes that do collision traces are all affected by it. It seems totally self-contained as a rewrite of the way `FHitResult`'s hit actor is accessed. Reverting to the old calls made no difference. [Commit](https://github.com/EpicGames/UnrealEngine/commit/eb2edb9b69ffcc8e83d54018fd0f271c4dce6c68)
* The new GameplayCues support a lens effect from Niagara particles that is not yet in UE release. Reverting to using the `AEmitterCameraLensEffectBase` class is just fine, though. [Commit](https://github.com/EpicGames/UnrealEngine/commit/bb38d4afa77c5bf23947dc6a247ac9b4bafe7609)
* The new GameplayCues support MetaSounds (the procedural sounds). This is all new code, so deleting it should make zero difference. [Commit](https://github.com/EpicGames/UnrealEngine/commit/bf19dcdaee0bd35d4dcd212dd19f35608d1a0d19)
* Both the `GameplayAbilities` and `GameplayAbilitiesEditor` modules have a dependency to a module called "EditorFramework" which isn't present in UE4.27.2. The dependency doesn't seem to be relevant so it's safe to remove. [Commit](https://github.com/EpicGames/UnrealEngine/commit/48113fc77eeb98f282f175d01ff2338f56f169ef)
* GameplayEffects use the new PreSave and deprecate the old one. I'm confident in removing the new stuff and relying on what worked in 4.27 for now. [Commit](https://github.com/EpicGames/UnrealEngine/commit/fd97028c280c33056f0925655fc6b2bbede408eb)
* `GameplayEffect.cpp` has a macro that uses the new UE5 `ToRawPtr`. I can't track down the commit that changed it, but removing it works OK seeing as it's been that way most of this file's life.
* `MarkPendingKill()` and `IsPendingKill()` have been deprecated and replaced with `MarkAsGarbage()` and `IsValidChecked()` respectively. Code has been replaced to follow this change, but reverting it should be fine. [Commit 1](https://github.com/EpicGames/UnrealEngine/commit/d6feb51b95193ffd0db963f3b14e5e2c75417c9e), [commit 2](https://github.com/EpicGames/UnrealEngine/commit/a8f27e9fa87dbf01a754cc2090e5e5e8bb846593)
* Net serialization works a bit differently and there's a constant being added at Engine/Source/Runtime/Core/Public/Misc/NetworkVersion.h. No reason not to just add it. [Commit](https://github.com/EpicGames/UnrealEngine/commit/e24e5d7ae418dc80b97495f7fb157f043025dadf)
* The class viewer module was expanded to support multiple class filters. This can be reverted. [Commit](https://github.com/EpicGames/UnrealEngine/commit/696bd28875f0f2b3ff14db01bb25b0589524559b)
* The LiveCoding rework affected some preprocessors in GameplayAbilitiesEditorModule.cpp, this can be reverted. [Commit](https://github.com/EpicGames/UnrealEngine/commit/a2237e050de4abad4b06219ede2d866ccb81ebb7)
* `FArchive` had `UE4Ver()` renamed to `UEVer()`. This can be reverted. [Commit](https://github.com/EpicGames/UnrealEngine/commit/bd6185935009aef14af992dd414ad79b3a0e1cc3)
* The AnimNotify for GameplayCues was actually a two-part change. The first was the addition, then was the modification to accommodate the new notify functions. This causes references to the new AnimNotify stuff which may actually be desirable to folks doing what I am, so I thought I might as well document it specifically. Personally, I am reverting the second change and keeping everything on the old AnimNotify system for now. I'll check back in here if I change my mind. [Commit](https://github.com/EpicGames/UnrealEngine/commit/f4c11a45c947a052869d4f2fbc5ae51f79cf17b9)
