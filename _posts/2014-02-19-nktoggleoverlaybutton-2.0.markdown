---
layout: post
title:  "NKToggleOverlayButton 2.0"
date:   2014-02-19 21:15:40
---

I recently re-did an open source component of mine (because I needed to use it in a new project) and was amazed by how much I had learned in the couple of years since I first wrote it.  The control is called [NKToggleOverlayButton](https://github.com/neilkimmett/NKToggleOverlayButton/) and is a button that toggles between an on and off state. When it toggles it displays an overlay (hence the name) with an image and a line of text. A good use is a "favourite" button.

The first and most obvious improvement was converting the code to use ARC. My rationale at the time for not using ARC was derived from [Jiva DeVoe's excellent blog article on the matter](http://blog.random-ideas.net/?p=169). Since then I think its reasonably safe to say most iOS projects are developed with ARC turned on, and if they aren't then they probably should be. Seeing all those `retain`s and `release`s gave me pangs of nostalgia but were also very pleasing to delete.

The second thing Past Me had done was inherit from `UIView`. What a fool. What he should have done is inherit from `UIControl` so that the target-action pattern can be used. Shiny new v2.0 duly does, and will inform any registered targets (added using `addTarget:action:forControlEvents:`) of any touch up/down events, as well as `UIControlEventValueChanged` (used when the state of the button is toggled on/off). Previously behaviour was added to the control using either a pair of blocks, or a custom `delegate` property which only had one method `toggleButtonDidToggle:`. Another bonus `UIControl` gives you is the `selected` property, which supercedes the custom `isOn` property from `NKToggleOverlayButton` v1.0. I also added a public method `setSelected:animated:` for programmatically toggling the button. This makes for a much cleaner, more familiar public API than the custom `delegate` that v1 had (which has now been removed).

Speaking of public APIs, doxygen style comments have now been added to the header file. This means that you can ⌥-click (a.k.a Alt-click or option-click) on a method in Xcode and see a delightful little popup with all the info you need.

![Doxygen comments shown in Xcode](/assets/doxygen.png)

The look and feel of the control has been updated for iOS7. Previously the overlay was black and translucent, with white text. This felt nice in iOS6 and below but feels outdated in iOS7. Now the overlay is a nice frosted blurry background with black text. This feels more native and matches system controls like the volume changer.

<figure>
  <img src="/assets/nktoggleoverlay_ios6.png" alt="Overlay in iOS6" class="left half">
  <img src="/assets/nktoggleoverlay_ios7.png" alt="Overlay in iOS7" class="right half">
  <figcaption>iOS6 on the left, iOS7 on the right</figcaption>
</figure>

Finally the code got a big ol' hit with the refactor stick. Smaller methods, more code reuse, more easily readable code. All in all I'm very happy with v2.0, and very happy with the improvements I've personally made over the last 2 years.

[Check it out](https://github.com/neilkimmett/NKToggleOverlayButton/) and let me know what you think (contact details below)

