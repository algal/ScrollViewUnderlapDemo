
# What this is

This repo accompanies an answer on Stack Overflow, showing how to use UIScrollView, Auto Layout, and layout guides to position content so that it can scroll under a translucent nav bar.

The project "temo" shows it done in the way I describe below.

The project "temo2" shows it done in the way I do not recommend, because it effectively bakes hardcoded nav bar metrics into your layout constraints. This wrong way is the way IB pushes you toward.

Both of these ways will produce the desired effect.

# Answer


You are right to be confused. It is a bit counterintuitive but the *top and bottom layout guides are irrelevant to configuring a UIScrollView* so that its scrollable content will underlap the translucent navigation bar, which is the effect you are trying to achieve.

## what to do
Given the view hierarchy you've shown in the second picture, this is what you need to do on iOS8:

1. configure the view controller so that "Extend Edges Under Top Bar" is checked (in code, use `edgesForExtendedLayout `). This will ensure that the view controller's lays out its root view so that it underlaps the nav bar.

2. configure the scroll view constraints so that the scroll view's top edge has a zero offset from the top edge of its superview, *not* zero space from the top layout guide. This will ensure that the collection view fills the root view and thus also underlaps the nav bar, which is necessary for the scroll view's content to be able to scroll under the nav bar. (IB might fight you on this. See the footnote below.)  

3. So now how do you make sure that the scroll view has any idea where the nav bar is, so that (for instance) it doesnt't _always_ position its content under the nav bar? The answer has nothing to do with layout guides. In the view controller, check the box "Adjusts Scroll View Insets" (or in code, `automaticallyAdjustsScrollViewInsets`). This will cause the view controller to automatically adjust the scroll view's `contentInset` property so that the scroll view positions its content appropriately.

This will work.

## what's going on

So why is this the answer? And why is it so confusing?

Frankly, it's easy to get confused because the top and bottom layout guides are prominently presented to us as elements that convey layout information about translucent overlaid elements. However, they are not the only "translucency-aware" layout mechanism. They are directly relevant only for positioning of "normal" subviews, i.e., not the view controller's root view, and not content within a UIScrollView.

Content within a scroll view (or a subclass like UICollectionView and UITableView) will always be positioned in a more complicated way involving the scroll the view itself, affected by properties like `contentInset`, `contentOffset`, etc.. (Really, if scroll view layout were a straightforward thing, why would Apple have dedicated WWDC sessions to scroll view layout for the last four years running?!)

To summarize, as the steps above indicate, the *three* distinct translucency-aware mechanisms for managing layout are as follows:

1. "Extends Edges" determines if the view controller positions its root view so that it underlaps the nav bar.

2. Layout Guides provide a metric that tells where the "main" content area is, taking translucent bars into account. You can use these with Auto Layout to position normal views so they don't underlap. Or you can access the numerical values in code.

3. Scroll View Insets are the right way to ensure that a scroll view's contents can underlap but doesn't always underlap. The `automaticallyAdjustsScrollViewInsets` property on the view controller can do this for you automatically in simple cases. (Presumably, this property just causes the view controller to update the scroll view's `contentInset` based on the same values it exposes via the layout guides. So if you needed to manage the insets yourself, that's how you would do it.)


## fighting IB's layout guide mania

A footnote on "Fighting with IB":

Unfortunately, Interface Builder might fight you when you try to constrain the scroll view edge to its superview's edge. Instead, it might try to get you to constrain the scroll view against the view controller's layout guides. This is because IB mindlessly prefers layout guides to superview edges, when the superview is the root view. But when you're working with a scrollview, this is *the wrong thing*.

Why? For instance, suppose you accept a constraint against the layout guide. Then you will end up with a top constraint on your scrollview that constrains it to topLayoutGuide-64.0. That -64.0 is a hard-coded value compensating for the exact height of a nav bar. So what happens when one fine day the nav bar does not equal 64pt? Or when you simply turn off the nav bar entirely? Or want to re-use this scene in a context without a nav bar? Answer: then you get a broken layout.

So how do you force IB to add a constraint from the scroll view to its superview's edge, as opposed to the layout guide? Ugly admission: I don't know. I have had luck doing a control-drag from scroll view to superview in the canvas rather than the document outline, and using "add missing constraints". But I have not found a 100% reliable method. This feels like a bug in IB.
