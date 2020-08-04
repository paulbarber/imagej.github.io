== Introduction ==

Since [[2014-06-04 - ImageJ 2.0.0 release candidate|releasing ImageJ2 into the wild]], there has been a tremendous amount of user feedback. For those who have [[Report a Bug|reported bugs]]: thank you!

Of all the issues reported, one has stood out as the most tenacious and difficult to solve: [https://fiji.sc/bugzilla/show_bug.cgi?id=805 Fiji bug #805, "Fiji not closing"], colloquially known as the "Fiji won't quit!" problem. (Actually, versions of this bug existed before bug #805 was reported, but that bug is where we have been tracking the issue most recently.)

As of this writing, we have just released ImageJ 2.0.0-rc-9 which we believe finally solves this issue after [https://github.com/imagej/imagej-legacy/commit/f2ba2b2645fe6aa5f0a0b5591defd37881dba31f six] [https://github.com/imagej/imagej-legacy/commit/c441b81ac5b830ee8752038b3d9d86858b552634 different] [https://github.com/imagej/imagej-legacy/commit/8af9bfc4a0010374fa2390041c3735a7bbcc7e6f attempts] [https://github.com/imagej/imagej-legacy/commit/62259dab7bbe70064ccaed621ac3940ffc6aaf61 to] [https://github.com/imagej/imagej-legacy/commit/fe237a23fbde86171b8d574bdeb9c34a397dcfff fix] [https://github.com/imagej/imagej-legacy/commit/1f9b76f270e08a9c50abaf09c5938c4e08733892 it].

There were actually two different classes of misbehavior:

# '''Being unable to quit.''' The main window refuses to disappear, and the program becomes largely nonfunctional from that point forward and must be force closed.
# '''Process not terminating.''' ImageJ completes its shutdown routine and all windows disappear, but the Java process continues running in the background because <code>System.exit(0)</code> is never called.

== Being unable to quit ==

This class of bug was introduced because [https://github.com/imagej/imagej-legacy/commit/11fa5cb86112ae381448bf15a40fa29aeb32d553 we tried to improve upon ImageJ1's behavior relating to the closing of windows and dialogs]. It was previously the case that if you quit ImageJ while the [[Script Editor]] had tabs with unsaved changes, those changes would be discarded with no chance to save first. The behavior of ImageJ1 with other types of "non-sanctioned" windows (i.e., {{GitHub|org=imagej|repo=ImageJA|tag=v1.49d|source=ij/WindowManager.java#L393-L400|label=all <code>Window</code> instances other than <code>ImageWindow</code>, <code>TextWindow</code> or non-<code>Editor</code> <code>PlugInFrame</code>}}) is to dispose them immediately without warning just prior to shutdown.

To address the problem, we [https://github.com/imagej/ij1-patcher/commit/7b202c6c826e870c23a1fb0b91ebb86c217c133c added a callback hook to ImageJ1] on the <code>WindowManager.closeAllWindows()</code> method, so that we could inject additional behavior. We then [https://github.com/imagej/imagej-legacy/commit/11fa5cb86112ae381448bf15a40fa29aeb32d553#diff-fb9b7c8be0fd7333a89ddb85e48390e5R508 dispatched a windowClosing event to each remaining window] to give them a chance to opt out of being closed. But it turns out that Java's standard paradigm of allowing windows to cancel their own closing conflates the ideas of confirming the close (e.g., with the user) with the process of actually doing the close/dispose on the window afterward. The approach proved too fragile and prone to deadlocks, so we ended up opting for a [https://github.com/imagej/imagej-legacy/commit/a1b3987e0302c270f80b0847ce86ca1ce1dd6861 different approach] instead. With a [https://github.com/imagej/imagej-ui-swing/commit/b830cf749115065407564e3c5a65dae3ec74ab09 minor update to the Script Editor], its windows now still prompt to save changes when quitting (whoo hoo!), but without the fragility of the original approach.

== Process not terminating ==

The second class of problem was introduced because ImageJ2 does not launch ImageJ via the {{GitHub|org=imagej|repo=ImageJA|tag=v1.49d|source=ij/ImageJ.java#L624|label=<code>ij.ImageJ.main</code>}} method. There are several reasons for this from an architectural and technical standpoint, which are outside the scope of this blog post. But suffice to say that <code>ij.ImageJ.main</code> performs many actions which the ImageJ2 startup routine also needs to perform (but not in exactly the same way), such as handling command line arguments, most of which we covered&mdash;except for {{GitHub|org=imagej|repo=ImageJA|tag=v1.49d|source=ij/ImageJ.java#L666|label=a call setting the <code>exitWhenQuitting</code> flag to true}}. By default, ImageJ1 does ''not'' call <code>System.exit(0)</code> when quitting, but whenever it is launched via its main method, it does. So even after [https://github.com/imagej/imagej-legacy/commit/428b93d7420649f843128fb9a992f53579105d39 we updated ImageJ2's quitting routine to lean on ImageJ1 as much as possible] (which seemed to fix the problem in many of our tests), it was still not always enough since <code>System.exit(0)</code> was never ultimately called. We fixed this discrepancy in the ImageJ legacy layer by [https://github.com/imagej/imagej-legacy/commit/fe237a23fbde86171b8d574bdeb9c34a397dcfff always setting exitWhenQuitting to true] as well.

== Other complications ==

There were other considerations too, of course. It needs to be possible to:
# Shut down ImageJ through the UI via the ImageJ1-based code path of {{GitHub|org=imagej|repo=ImageJA|tag=v1.49d|source=ij/ImageJ.java#L603|label=ij.ImageJ#quit()}}.
# Dispose of ImageJ programmatically via the ImageJ2-based code path of {{GitHub|org=scijava|repo=scijava-common|tag=scijava-common-2.27.0|source=org/scijava/Context.java#L367|label=org.scijava.Context#dispose()}} which in turn {{GitHub|org=imagej|repo=imagej-legacy|tag=imagej-legacy-0.5.20|source=net/imagej/legacy/DefaultLegacyService.java#L402|label=disposes the <code>LegacyService</code> and hence ImageJ1}}.

These two code paths need to behave very differently. In the case of (1), ImageJ1's quitting routine happens, but some extra logic is injected to handle window closing better (see "Being unable to quit" above) as well as shut down the ImageJ2 application context in addition to ImageJ1 itself. In the case of (2), ImageJ2 disposes its entire context including the <code>LegacyService</code> which is responsible for managing ImageJ1, meaning that ImageJ1 gets disposed as ''part of'' ImageJ2's disposal&mdash;all of which must {{GitHub|org=imagej|repo=imagej-legacy|tag=imagej-legacy-0.6.0|source=net/imagej/legacy/IJ1Helper.java#L199-L219|label=happen ''without'' calling <code>System.exit(0)</code> and ''without'' prompting the user to save any changes}}.

== Conclusion ==

We believe we have finally ironed out all these problems, but as the above explanation hopefully conveys, ImageJ1 is fraught with multiple code paths, and quitting the program is no exception. Searching the ImageJ1 codebase for <code>System.exit(0)</code> yields five separate places where it gets called, one of which, astonishingly, is a {{GitHub|org=imagej|repo=ImageJA|tag=v1.49d|source=ij/IJ.java#L1464|label=potential code path of the <code>ij.IJ#getImage()</code> method}}! Adding on the ImageJ legacy layer on top is a very complex endeavor, but we are proud to say that instances of ImageJ2 can be cleanly disposed of ''without'' shutting down the entire JVM, which provides a substantial boost to ImageJ's usability as a software library.

P.S. We [https://github.com/imagej/imagej-legacy/commit/8ead1c0a5eeb1a040d2eb473d8420c695a487709 added regression tests] for all the major scenarios, even including [https://github.com/imagej/imagej-legacy/commit/3dedc32cb4609f2840db65947d6adedbcba29400 an integration test for verifying that <code>System.exit(0)</code> is called] under the correct circumstances.

P.P.S. The "Fiji won't quit" T-shirt is coming soon! :-)

[[Category:News]]