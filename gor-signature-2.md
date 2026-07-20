# The Test That Passed While It Was Broken

*Field Instrument 127: SIGNATURE, versions 1.4 through 1.6*

Last time I wrote about SIGNATURE, my bookbinding imposition bench, I opened with a bug. The first version folded quarto and sextodecimo in the wrong order, and my tests all passed anyway because they checked which pages landed on each side of the sheet, a fact that is identical under both the right fold order and the wrong one.

I ended that post with a line I meant sincerely: a test that confirms a true fact is not the same as a test that would fail if you were wrong.

Then I built three more versions and did it again.

## What got built

**Binding breadth.** SIGNATURE could only make one kind of book: sections sewn on tapes or cords or in a French link, cased into hard boards. That is a Western trade binding and it is not the only thing a bindery does. So now it also does Coptic link stitch, where the sections sew directly into drilled boards and the spine stays exposed. Longstitch, where the thread comes through a wrapper and shows on the outside. And Japanese stab binding.

Stab binding is the interesting one, because it broke my assumptions rather than extending them. A stab book is not folded at all. It is a stack of single leaves with holes drilled parallel to the spine edge and sewn through the face. Choosing it has to switch the entire instrument: imposition becomes sequential instead of scrambled, the fold line becomes a cut line, push out compensation switches off because single leaves do not nest inside anything, and the punching template stops being a strip for the cradle and becomes a pattern down the edge of a clamped block.

**Rounding and backing, properly this time.** Version 1.0 computed the spine covering for a rounded book by multiplying the flat thickness by 1.08. I typed that number. I did not derive it and I did not check it. It is now real geometry: you say how much of a circle the spine carries, a third being the usual round, and the instrument computes the radius, the sagitta, and the arc the covering actually has to travel. A third of a circle costs about 21 percent more material than the flat chord, not 8. Every book I would have cut cloth for using that number would have come up short.

**Bridges to the rest of the bench.** The case parts now leave as SVG and DXF, and the materials leave as a BOM CSV. This was the one place I was most at risk of doing something stupid, and the thing that saved me was a rule I set before writing any code: do not invent an interchange format. My laser instruments already read SVG and DXF. My quoting bench already reads a BOM CSV. The bridge is to speak what they already speak, not to design a bespoke JSON schema and then maintain it in four places forever.

**A bench punch list**, which is every measurement in the order you actually work, with a box to tick beside each step and a blank line at the bottom to write the block thickness you really got, as opposed to the one the arithmetic predicted.

**And an install layer** that tells the truth. SIGNATURE runs from a file on disk with no server. That is already more offline than a progressive web app, which caches things over a network it needed in the first place. A service worker cannot even be registered from a `file://` URL. So when it is running from disk it says exactly that: there is nothing to install, nothing to cache, and nothing to go stale. If you host it, it registers a worker and offers you the file. What it does not do is show you an install button that quietly does nothing.

## Now the bug

While building the stab binding, I needed the fold instruction panel to say something different for a book that has no folds. So I put a branch at the top of that section: if this is a side sewn book, write the cutting instructions, then stop.

I wrote `return`.

That block was not inside a small function that renders the fold panel. It was inside `recompute`, the function that rebuilds *everything*: the imposition, the sewing marks, the spine arithmetic, the case dimensions, the cutting list. My `return` did not end the fold panel. It ended the entire recalculation, silently, every time you chose a stab binding.

The screen did not go blank, which is what made it dangerous. Every readout kept showing the values from the last structure you had selected. Pick longstitch, then pick stab, and the instrument would cheerfully show you a wrapper, a set of slit positions, and a sewing pattern belonging to a book you were no longer making. It looked completely normal. It was completely wrong.

I ran the self test. Two hundred and eighty one assertions, zero failures.

They all passed because every one of them called the mathematics directly. `caseMath` with a stab binding returns a stab case, correctly, and always did. `sewStations` with a stab style returns four holes down the spine edge, correctly, and always did. Every piece of arithmetic in the instrument was right. The fault was that under real conditions the arithmetic was never called, and no test that pokes a function with arguments can ever notice that nobody is poking it.

I only found it because I dumped every export to disk and read them back, and the parts list for the stab binding came out saying "Wrapper".

## The two tests I did not have

The fix took a minute. What took longer, and mattered more, was working out what kind of test would have caught it, because clearly the kind I had would not.

The first is a wiring test. Not "does this function compute the right answer" but "does choosing this option through the actual controls reach the actual outputs." It drives the real select elements, fires real change events, runs the real recalculation, and then asserts that the case, the sewing marks and the parts list all match the structure that was chosen. It is unglamorous and it is the only test in the suite that could have seen this.

The second is the one I should have been doing all along: after writing a test, put the bug back and watch it fail. I reintroduced the early `return`, ran everything, and got exactly what I needed to see. The in-app suite still reported 281 passed, zero failed, cheerful as ever. The new wiring test failed on three assertions and named the problem: the case is stale, the marks are stale, the parts list says Wrapper. Then I took the bug back out and everything went green.

A test you have never seen fail is not a test. It is a hope with a green tick next to it.

## And a third thing, from an outside opinion

There is a related failure worth mentioning, because it happened in the same session and has the same shape.

The DXF exporter I wrote passed its tests. I checked that the file opened and closed with the right markers, that every group code was paired with a value, that there was one polyline per part, that the units were declared. Sensible checks. All true. All passing.

Then I opened the file with `ezdxf`, a real DXF library written by people who actually know the format, and it refused to read it. AutoCAD 2000 requires each entity to carry subclass records that say what kind of record it is, and mine had none. My tests could not have caught this, because my tests only knew what I knew, and what I did not know was precisely the problem.

That is the general case, and it is worth saying plainly. Your test suite is written from your own understanding. Where your understanding has a hole, your tests have the same hole in the same place. The only way out is to hand your output to something that was not written by you and does not share your assumptions: a different library, an independent implementation, a rasterizer, a person. For the fold geometry that was a separately written verifier. For re-imposed PDFs it is a rasterizer comparing every placed cell against the original page pixel by pixel. For the DXF it was `ezdxf` telling me no.

Three tiers, three bugs, all three found by something other than my own tests. I would rather report that than a clean run, because a clean run mostly means you have not looked hard enough yet.

## Get it

SIGNATURE is Field Instrument 127. One file, offline, GPL-3.0. It carries 286 self tests, and 88 more in the release harness, some of which now check the wiring rather than the sums.

Print the calibration sheet first. Every printer lies about which edge it flips on, and folding a sheet with big numbers on it settles the argument in about four seconds.

Then fold something. Or, if you have gone with stab binding, cut something, stack it, clamp it, and drill straight through. The grin is the same either way.

Make. Hack. Learn. Share. Repeat.
