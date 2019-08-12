This project was originally hosted at https://code.google.com/archive/p/unrealscript-gui-framework/

I have now moved it to GitHub to make it easier for people to fork it, but I won't maintain the project myself since my days of working with the UDK are long time gone. I started the project and later got help from the user "thommie". The original commit history (as much of it as Google Code still provides) are included in my first commit to this repo for reference.

# Canvas-based GUI Framework in UnrealScript
Pure UnrealScript implementation of a simple framework for creating graphical user interfaces in Unreal Engine 3 / Unreal Development Kit.

This project provides GUIComponents and the base for a GUI that relies entirely on UnrealScript and the classic Canvas to create interactive menus. The "officially supported" way of creating menus is to use Adobe Flash, Scaleform and ActionScript, but not every smalltime developer may wish to spend money on the full Adobe Suite for that - hence why this is a free alternative with the benefit of keeping everything in one place.

# Documentation
A full documentation and examples of this system can be found in the Wiki. The GUIFrameworkExample package also contains a ready-to use example gametype that adds a small menu using this framework.

Anyway, here is a basic quick start overview about the most important concepts:

## Introduction to this system

- The HUD class is the originator for all interaction of this framework. We provided a **GUICompatibleHUD** from which other HUDs can be derived. Your HUD should create your custom **GUIMenuScene** object which will take care of the whole rest. The HUD causes the MenuScene to draw it's components on every PostRender(). Keep in mind that your GameInfo's bUseClassicHUD must be set to True in order to have this function called.

- Another thing to take note of is the **GUICompatiblePlayerController**. It intercepts all mouse buttons via exec functions (some of them need to be bound in the INI first, others like StartFire check if the MenuScene's bCaptureMouseInput is True and then either route the event to the MenuScene or process the StartFire as usual). The **GUICompatiblePlayerInput** monitors the mouse movement (gamepad axes can be used as well) and updates the cursor coordinates in the HUD accordingly.

- Pressed mouse buttons cause the matching pending booleans in the MenuScene to be set to true, for instance bPendingLeftPressed and bPendingLeftReleased. The MenuScene then updates the location of the mouse according to the PlayerInput and calls UpdateHoverSelection() to recursively find the GUIComponent that is currently under the mouse cursor and notifies the new Component that it's now focused and the old one that it's no longer focused, respectively. Afterwards it processes the pending booleans for the mouse buttons and notifies the currently selected GUIComponent that it got clicked, scrolled, etc. (It is easy to enhance the system at this point to also allow clicking on ingame 3D objects by implementing the MouseInterface UDK Gem. We also provided a GUIScriptedTextureHandler class as starting point to draw menus on ingame surfaces instead of the HUD. You still need to take care of the involved tracing and coordinate conversions if you want to interact with them.)

- **GUIComponent** is the very base class of components in a MenuScene. It defines basic functionality such as helping the MenuScene to find out which of overlapping components take precedence for mouse cursor selection or calculating the actual screen position. Components can be set to have a position in absolute values, in relative values, with HUDResolutionScaling or without and can have fixed distances relative to other components.

- GUIComponents are stored in a dynamic array in the MenuScene and added via DefaultProperties. The MenuScene then goes through this array and calls each GUIComponent's DrawComponent() function, to which it passes the HUD's Canvas. So each GUIComponent gets a chance to draw itself and also gets ticked (the time since the last render call can be accessed via HUD.LastHUDRenderTime).

- **GUIContainerComponents** hold an array of several other GUIComponents. Whenever they are drawn, they will also tell each of their child components to draw as well. This way you can create occluding layers. A GUIPage is a subclass of the GUIContainerComponent and is meant to fill the entire screen. The GUIMenuScene provides additional functions to realize a stack-like concept where you push and pop pages onto and from the stack and only the topmost page is drawn, thus giving you some structure when navigating menus and saving unnecessary draw calls on components that are hidden anyway.

- **GUIVisualComponent** is the base class of all components that use textures, like buttons, background images and so on. (This might be refactored to comply more with the system of GUIMultiComponents, which is something used in UT2k4's GUI code. Basically speaking, it's a component that is made of several sub components, for instance a button out of a component that handles the "collision", one that handles the visuals and another one that handles the text label.)

- The way textures can be stretched in the Canvas allows for some nice lossless scaling, provided that the textures were especially designed to work with that approach: It simply repeats the pixels that are in the middle of the texture, so you always have the same edges at high resolution and don't lose any detail if your middle part is equally colored anyway. See https://api.unrealengine.com/udk/Two/CanvasReference.html#DrawTileStretched


# Workflow
It may be tricky to find the correct values to put into the properties of your GUIComponents if you just see things in code view at all times. To compensate for that, you should make use of the Remote Control feature (https://api.unrealengine.com/udk/Three/RemoteControl.html) that the Unreal Engine provides.

Find your MenuScene in the properties of your HUD/PlayerController in the Actors menu and change the properties of your components there to see the effects in real time. Just don't forget to update the DefaultProperties in your source code after you found the right values.
