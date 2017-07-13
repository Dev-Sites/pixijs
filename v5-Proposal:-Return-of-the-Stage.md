## Problem

1. Child has no way to know whether parent was removed from the stage. 
2. The Destroy API is horrible, we just ask people to maintain it on their side, but we dont give any tools to determine the moment object leaves the stage, and user has to implement refcounting on his side.
3. There is no way to differentiate container from the root element, this flag is maintained by user.
4. Recursive calls of add/remove complicates the stacktrace, and problems in remove/destroy appear too often, users see that mess.

Possible problems with naive implementations:

1. If we simply pass 'added' and 'removed' messages recursively it will affect performance when we construct difficult scene.
2. Any simple manipulation within the stage, like moving an element from one subtree to another will fire those events and possibly trigger some destroy sequence.
3. If we just add special flag in "removeChild" that won't be good because people are having problems searching that flag in [cocos2d engine](http://discuss.cocos2d-x.org/t/why-removefromparent-removechild-could-be-dangerous/32223), its not intuitive.
4. Porting old code may be difficult if we force usage of the stage. We must ensure compatibility with old "container as root" approach.

## Solution

1. Put big sign for users **DO NOT MODIFY `children` PROPERTY**.
2. Mark all display objects with UniqueID
3. Stage is Container that must store all its children in set by their ID. Elements store a link to the stage they were added.
4. If subtree is added/removed to the stage, fire `added`/`removed` event through whole subtree and re-assign `stage` field for all elements.
5. Add `detachChild` method that does not fire `removed` but adds whole subtree to special stage's `detached` set of elements, that can be removed on next frame, if user says so, or if special flag is enabled. Calling "addChild" on detached elements adds them back, without `added` signal/event.
6. Add/remove/detach works like in pixi-v4 when there is no Stage parent.
