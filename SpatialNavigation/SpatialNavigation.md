#SpatialNavigation
##Outline
##Introduction
Spatial navigation allow you to change the focus of a web page.  Traditionally, users had to either use a mouse or have to tab between different elements of a web page.  In many cases using a mouse isn't possible.  For example, there are many people that can not or do not use a mouse, and there are many devices ranging from cell phones to televisions that do not have a mouse.  This leaves the user having to tab through the element in whatever order the page is laid out in.  However, if the user just wanted to move the focus one element to the right or left, it might require dozens of key presses.[1][mozillaSpatialNavigation]

##SpatialNavigation.jsm
It listen the arrow keycode and find the node related to current node spatially. Use focusManager.moveFocus to change current node.<br>
Find the first node: focusManager.moveFocus(currentlyFocusedWindow, null, focusManager.MOVEFOCUS_FIRST, 0);[2][findFirst]<br>
Find the node spatially: Use _getSearchRect to get possible range[3][getSearchRect]<br>
windowUtils.nodesFromRect() to get the nodes in range.<br>
spatialNavigation while[4][whileCondition], don't understand the searchRectOverflows

[mozillaSpatialNavigation]: https://www.mozilla.org/access/keyboard/snav/  "Optional Title Here"
[findFirst]:https://dxr.mozilla.org/mozilla-central/source/toolkit/modules/SpatialNavigation.jsm#121
[getSearchRect]:https://dxr.mozilla.org/mozilla-central/source/toolkit/modules/SpatialNavigation.jsm#153
[whileCondition]:https://dxr.mozilla.org/mozilla-central/source/toolkit/modules/SpatialNavigation.jsm#160