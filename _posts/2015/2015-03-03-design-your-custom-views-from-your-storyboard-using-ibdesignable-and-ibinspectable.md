---
title: "Design Your Custom Views From Your Storyboard Using @IBDesignable and @IBInspectable"
date: "2015-03-03"
categories: 
  - "technical"
---

Latest Xcode 6 release introduces two new keywords particularly useful for advanced design in real-time in storyboard. Until now, it was impossible to preview custom views with corner radius or a border in your storyboard ; it's now possible.

- @IBDesignable tells the storyboard to update the view in real-time
- @IBInspectable exposes the related attributes to the storyboard to let you change values dynamically

We'll see how to create a basic custom view with rounded corners and a specific border, and how to update it from the storyboard.

* * *

**Step 0 : Create a new class inheriting from UIView**

Start by creating a new Cocoa Touch class inheriting from UIView, and add the keyword **@IBDesignable** right before the declaration to tell the storyboard to update all instances in real-time. Create three properties to adjust the border width, color and the corner radius, with the keyboard **didSet** to trigger a specific action once the property value changed.

```swift
@IBDesignable class CustomView: UIView {

    @IBInspectable var borderColor: UIColor = UIColor.clearColor() {
        didSet {
            updateUI()
        }
    }

    @IBInspectable var borderWidth: CGFloat = 0.0 {
        didSet {
            updateUI()
        }
    }

    @IBInspectable var cornerRadius: CGFloat = 0.0 {
        didSet {
            updateUI()
        }
    }

    func updateUI() -> Void {

        self.layer.borderColor   = self.borderColor.CGColor
        self.layer.borderWidth   = self.borderWidth
        self.layer.cornerRadius  = self.cornerRadius
        self.layer.masksToBounds = true
    }
}
```

Notice that once a property is changed, the method _updateUI()_ is called to update the instance. You can call different methods depending on the kind of property changed, I mixed the whole together to avoid getting complexity.

* * *

**Step 1 : Adjust these properties live**

Start by adding a standard UIView into your storyboard, and make it a subclass of your previously created class (_CustomView_ if you're following the tutorial).

[![Screen Shot 2014-11-05 at 10.48.25 pm](/assets/images/Screen-Shot-2014-11-05-at-10.48.25-pm-300x114.png)](http://nscurious.com/wp-content/uploads/2014/11/Screen-Shot-2014-11-05-at-10.48.25-pm.png)

You can see a new label _Designables_, telling you the UI is updated and ready to be live-adjusted. Now go into your Inspector attribute... Surprise, your properties are directly accessible from here!

[![Screen Shot 2014-11-05 at 10.49.36 pm](/assets/images/Screen-Shot-2014-11-05-at-10.49.36-pm-300x148.png)](http://nscurious.com/wp-content/uploads/2014/11/Screen-Shot-2014-11-05-at-10.49.36-pm.png)

Try to adjust these settings, and see the result live into your storyboard!

[![Screen Shot 2014-11-05 at 10.52.06 pm](/assets/images/Screen-Shot-2014-11-05-at-10.52.06-pm-284x300.png)](http://nscurious.com/wp-content/uploads/2014/11/Screen-Shot-2014-11-05-at-10.52.06-pm.png)

Here we are, you just discovered these two new keywords. Now, both your creativity and needs are the limits. You can find a similar example [here on Github](https://github.com/fiftydegrees/HLDesignableBootstrap).
<br>
<br>

---------------------------------------
<br>
Auteur: **herve.droit**
