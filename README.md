#Heavy Wizards Radar#

The most comprehensive radar & mini-map system we could think of.

[![Heavy Wizards Radar Trailer](/images/YoutubeTrailer.png)](https://www.youtube.com/watch?v=K-1kpFTDVMc)

This is the official repository for the **Heavy Wizards Radar** plugin for the **Unreal Engine 4**.
It contains all the documentation, tutorials, examples and external links necessary for a successful integration into your own game project.
This is also the place to submit any technical issues (bugs) or feature requests.

##Main features:##
* Customization: nearly anything can be tweaked.
* Scalability: works for any type of game where a 2D mini-map & radar is appropriate (side-scrollers are supported). Works on any platform.
* Precision: every map has a world representation to properly project actors & environment onto the radar.
* Power: all graphics is material based making the imagination the sole limitation of how the radar will look.
* Convenience: provides many out of the box features.
* Performance: written in C++ with performance in mind, maps serve to limit the amount of actors that need processing.

##In-depth features:##
* A map box is placed in a world and can be moved, rotated & scaled in any way to fit the actual map to the world.
* Actors with overlap events enabled & within a map box will be rendered according to a dynamic material that can be tweaked at any point.
* Schemes are used to customize which actors are rendered & material rendering.
* The map can be zoomed in & out to one's digression which is not bound to any particular method like using the mouse wheel.
* An option to add arrows pointing to actors which will by default be drawn on the edges of the radar, but that behavior can be overridden.
* Deproject click & touch events onto the map plane which is derived from the map normal, its rotation & position.
* A number of often used materials & material functions are provided out of the box.

##Key concepts:##
###Widget###

The radar widget itself is an UBorder that on the top level manages all of the aspects of the Heavy Wizards Radar.
Specific tasks are usually delegated to other classes listed below. The only very specific task that’s the responsibility of the widget is adding and managing arrows on the parent UCanvasPanel because
it is the closest entity to the task. The widget keeps track of all the necessary data and performs memory management. Since widget is an UBorder it provides click/touch events out of the box.

####Defaults####

*	*ShouldDrawByDefault* on the global level determines whether an actor's material should be drawn.
*	*ShouldRotateByDefault* on the global level determines whether an actor should rotate.
*	*ShouldRotateMapByDefault* on the global level determines whether a map representation should rotate around pivot.
*	*ShouldMoveMapByDefault* on the global level determines whether the representation of this particular map should move together with the pivot.
*	*RefreshInterval* sets the interval between canvas redraws.
*	*CleanUpInterval*, how often the Radar has to look whether an actor dropped out of the world to properly release all of the pointers necessary.
*	*Zoom*, starting zoom level.
*	*SchemesClasses*, scheme classes to be used when the widget starts.
*	*MaterialTemplate*, template containing the Canvas reference amongst other things.

####Methods and delegates####

*	*Start* starts the widget coroutines, provided a pivot actor is supplied.
*	*Deproject* method is used to project a click/touch onto the map plane (explained below). Useful for strategy games and calling a tactical nuclear strike on the unsuspecting heads of your enemies.
*	*CalculateRotation* method is provided to get the rotation of the actor projected onto the map plane (explained below) as it is sometimes necessary such as the case for 2D games where no rotation is used, but instead two different textures representing left and right orientation.
*	*ArrowPositionProvider* is the delegate to calculate a position on or around the widget where an arrow should be placed. By default arrows are just placed along the border of the Radar.

###Canvas###

The canvas is a UCanvasRenderTarget2D that draws the representation of a desired type of an actor on the map as well as the map itself which resides in the radar map.
It’s as loosely decoupled from everything else as possible because its single task is to draw a map material and a batch of materials associated with actors given location, rotation and size.

###Map###

The ATriggerBox representing a map in a world. It’s basically a plane having a position and rotation in the world and a normal.
It contains the material of a 2D world or level map. The map is projected onto the plane together with actors and their forward vectors for rotation,
hence the main responsibility of the map is to project and deproject entities. An architecture like this makes it possible to use the **Heavy Wizards Radar** for any kind of game where a 2D mini-map/radar will do.
It works well for FPS, TPS, RPG, strategy, 2D (or 2.5) side-scrollers and really any game where verticality is not too pronounced, the games such as air combat simulators and space games.

####Defaults####

*	*MinZoom* and *MaxZoom*, the min- and max zoom level for this map.
*	*MaterialTemplate*, template containing a 2D map.

####Overrides called once####

*	*OnCreateMaterial*, called once a material is created which can only happen once per map.

####Overrides called periodically by an interval####

*	*OnRefreshMaterial*, called every time before a map material is drawn.

####Overrides called periodically by an interval which work at the 2nd level above the widget####

#####This is the second level between the widget global and a scheme's own#####

*	*ShouldDraw* signals whether a material should be drawn or not.
*	*ShouldRotate* signals whether an actor’s representation on a map should be rotated.
*	*ShouldRotateMap* signals whether the representation of this particular map should rotate around pivot.
*	*ShouldMoveMap* signals whether the representation of this particular map should move together with the pivot.

###Scheme###

A Scheme is one of the most important entities of **Heavy Wizards Radar**. The entire point of a Scheme is to offer customization to the client.
Schemes are mostly concerned with material handling. A scheme has 8 potential overrides.

####Defaults####

* *Class*, which class this scheme should be used with. Schemes are ordered by classes, having the deepest classes first representing the class hierarchy perfectly.
* *MaterialTemplate* and *ArrowMaterialTemplate*, templates for the actor and arrow respectively.
* *DefaultSize* and *DefaultArrowSize*, how should the representations be sized by default? Can be overridden in material functions, but if left at zero the defaults are used instead.

####Overrides called periodically by an interval####

*	*ShouldPointToActor* is used to determine whether an arrows should be used to point to a particular actor. If *OnRefreshMaterial* or *OnRefreshArrowMaterial* are overridden returns true by default, otherwise returns false by default.
*	*ShouldRotate* signals whether an actor’s representation on a map should be rotated. If not overridden a map’s default is used first, then the global Radar default.
*	*ProvideForwardVector*, used to project onto map to determine actor rotation. Actor’s own forward vector is used by default.
*	*ProvideAdditionalForwardVector*, as of now is only used for the map rotation on a scheme used for a pivot actor. Actor’s own forward vector is used by default.
*	*OnRefreshMaterial* and *OnRefreshArrowMaterial*, called every time before a material is drawn after its creation for the actor and arrow materials respectively.

####Overrides called once####
*	*OnCreateMaterial*, called once a material is created which can only happen once for every actor.

####Overrides called occasionally####
* *OnCreateArrowMaterial*, called once an arrow material is created which can happen according to the *ShouldPointToActor*.
Both the UCanvasPanelSlot and the material are destroyed if *ShouldPointToActor* returns false.

####Overrides called periodically by an interval which work at the 3nd level above the map####

#####These overrides trump all on the other levels#####

*	*ShouldDraw* signals whether a material should be drawn or not.
*	*ShouldRotate* signals whether an actor’s representation on a map should be rotated.
*	*ShouldRotateMap* signals whether the representation of this particular map should rotate around pivot.
*	*ShouldMoveMap* signals whether the representation of this particular map should move together with the pivot.

[![Heavy Wizards Radar Forum Thread](/images/ForumThread.png)](https://forums.unrealengine.com/showthread.php?129826-Heavy-Wizards-Radar-C-UMG-plugin&highlight=Heavy+WIzards)
