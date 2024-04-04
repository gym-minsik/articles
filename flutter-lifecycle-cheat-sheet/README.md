## Life of StatelessWidget
1. Widget Inflating Phase
   - When: Creating a new Element from a StatelessWidget and mounting the created Element into the ElementTree
   - What:
      1. build()
      
2. Rebuild Phase
   - When: The parent is rebuilding.
   - What: 
      1. build()


## Life of StatefulWidget and State
**1. Widget Inflating Phase #1**
  - When: Creating a new Element from a StatefulWidget and mounting the created Element into the ElementTree
  - What:
    1. final State state = StatefulWidget.createState();
    2. state.initState();
    3. state.didChangeDependencies();
    4. state.build();
   
**2. Widget Inflating Phase #2**
  - When: A StatefulWidget has a GlobalKey and has been deactivated but not disposed. Since deactivated elements usually get disposed on every drawFrame, they have a brief chance to reactivate. An example is using the same widget on two consecutive screens.
  - What:
    1. activate()
    2. build()
     
**3. Rebuild Phase**
  - When:
    - `setState` was called and the widget got a chance to be rebuilt by Flutter Engine.
    - the parent is rebuilding.
  - What:
    1. didChangeDependencies() if didChangeDependencies
    2. build()
    3. didUpdateWidget(oldWidget);
   
**4. Dispose Phase**
  - When: The Flutter Framework periodically disposes of elements in BuildOwner.inactiveElements.
    ```
    WidgetsBinding.drawFrame
      -> BuildOwner.finalizeTree() 
      -> lockState(_inactiveElements._unmountAll())
      -> element.unmount()
      -> state.dispose()
      -> state._element = null // it makes mounted == false;
    ```
  - What:
      1. dispose(), called by `unmount`. in dispose block, `mounted` is still true
      2. mounted == false; 
      
**5. Reassemble Phase**
  - When: Hot Reload was requested in debugging mode.
  - Why?: It provides an opportunity to reinitialize any data that was prepared in the initState method.
  - What:
      1. reassemble
      1. didChangeDependencies() if didChangeDependencies
      2. build()
      3. didUpdateWidget(oldWidget)