# SIGNATURE

**Field Instrument 127** &middot; version 1.7.0

A bookbinding imposition bench that runs in a browser, offline, from a single file.

A manuscript, a PDF or a saved feed goes in. What comes out is a set of scrambled sheets that fold into a real book, a punching template for the sewing stations, a calibration sheet that proves your printer's duplex before you commit paper to it, the spine and case arithmetic that follows from the block you just made, and a bench punch list with every measurement in the order you actually work.

Open `signature.html` in a browser. There is no server, no build step and no network call. It works from `file://` on a machine that has never been online.

---

## Stations

**Copy** &middot; the manuscript, the type, and how it is set.
**Fold** &middot; sheet, format, formes, and the fold sequence to follow.
**Sew** &middot; stations, swell, and the punching template.
**Case** &middot; boards, cloth, and the cut list.
**Press** &middot; preflight, printing in order, handing off, and saving the project.

A job bar under the station tabs keeps the decisions you have made in view: structure, format, pages, sheets, block thickness, and whether anything is complaining.

### One list of complaints

Every warning the instrument can raise goes into a single list, tagged with the station that raised it. That list is shown in three places: beside the figure it concerns, as a count on the station tab, and in full under **Before you print**.

This matters because the station where you commit paper is not the station that knows something is wrong. A fore edge trim smaller than the push out is discovered at the Fold station; the consequence lands when you print forty sheets. Now the warning follows you, and clicking it in the preflight list takes you to the station that can fix it.

---

## How the fold is worked out

The imposition is not a lookup table copied out of a printer's manual. Each format is folded in simulation using explicit affine geometry: the sheet is subdivided, each fold reflects one half onto the other and reverses the layer order of the moving flap, and the page order, the rotations, the spine edge and the final role of every crease are read back off the folded result.

Because the roles are derived rather than asserted, the instrument can check its own work. Every format is required to fold into a section whose spine lands on the left edge of the packet. A fold sequence that puts the spine anywhere else is refused, because that section would have to be turned a quarter turn to read and its pages would come out landscape.

An independent verifier, written separately and sharing no code with the instrument, re-derives the same geometry and is diffed against it cell by cell for every format.

---

## Structures

| Sewing | Folded | Finished as | Template gives you |
|---|---|---|---|
| French link, unsupported | yes | hard case, flat or rounded | section punching |
| Sewn on tapes or cords | yes | hard case, flat or rounded | section punching, support widths |
| Pamphlet stitch | yes | soft cover | section punching, one section only |
| Coptic link stitch | yes | exposed spine, boards only | section punching **and** board drilling |
| Longstitch | yes | wrapper | section punching **and** wrapper slitting |
| Japanese stab | **no** | side sewn covers | stab pattern down the spine edge |

The sewing decides the structure, so choosing it sets the binding rather than leaving you to keep two menus in agreement.

Japanese stab is the odd one. A stab book is single leaves, not folded sections, so choosing it switches the whole instrument: the imposition becomes sequential, the fold line becomes a cut line, push out compensation is switched off because nothing nests, and the punching template becomes a pattern of holes running parallel to the spine edge instead of down a fold.

### Rounding and backing

A rounded spine is a circular arc, and everything the case needs follows from it. Set how much of a circle the spine carries, a third being the usual round, and the instrument works out the radius, how far the round stands proud, and the arc the covering has to travel. Backing splays the block into shoulders for the boards to sit against; the shoulder is checked against the board thickness, because a shoulder that does not match leaves the board riding on the joint.

A third of a circle costs about 21 percent more covering material than the flat chord. Earlier versions of this instrument used a flat 8 percent, which was a guess and was wrong.

Rounding is refused politely on a block with no swell to redistribute, and warned about on a block too thin to hold a round.

---

## Formats

| Format | Grid | Folds | Pages per sheet | Bolts |
|---|---|---|---|---|
| Folio (2&deg;) | 2 &times; 1 | V | 4 | none |
| Quarto (4to) | 2 &times; 2 | H, V | 8 | head |
| Octavo (8vo) | 4 &times; 2 | V, H, V | 16 | fore edge, head |
| Sextodecimo (16mo) | 4 &times; 4 | H, V, H, V | 32 | tail, fore edge, head |

V brings the left half onto the right half. H brings the top half down onto the bottom half. The last fold is always the spine.

---

## Imposing a real PDF

Version 1.3 reads a source PDF and imposes the actual pages, not just their order.

A source page is never decoded, only relocated. Its content stream and every resource it reaches are copied across byte for byte and wrapped in a Form XObject. Text stays text, images are not resampled, and a page that arrived rotated or cropped is placed the way it looked. Only cross reference streams and object streams are ever expanded, because the file cannot be navigated without them.

The reader handles classic cross reference tables and cross reference streams, object streams, Prev and XRefStm chains, PNG and TIFF predictors, and inherited page attributes. Files with damaged offsets fall back to a scan.

---

## Bridges

Nothing here invents a file format.

**Case parts to SVG and DXF.** The laser instruments, CUTLINE and CUTPLANNER, already read SVG and DXF, so that is what the boards, inlay, covering, endpapers, drilled Coptic boards and slit wrappers leave as. Everything is 1:1, laid out without overlaps, cut lines unfilled. The DXF is AutoCAD 2000 with closed polylines and circles, written with the subclass records a conforming reader requires.

**Materials to BOM CSV.** Paper by the sheet including the blanks the imposition padded to, every case part with its size, thread by length with a working allowance, headbands, adhesive. The columns are `Reference, MPN, Description, Qty, Unit, Note`.

**A saved feed to a manuscript.** Point it at an RSS or Atom file you have already saved and each entry becomes a chapter. Nothing is fetched, and the feed's markup is reduced to text through the parser rather than inserted, so nothing in it can execute. A year of reading becomes a commonplace book.

---

## Installing it

The honest position: this file is already as offline as software gets. It runs from `file://` with no server and nothing cached, so there is nothing to install and nothing to go stale.

What an install adds is a window and an icon, and a service worker can only be registered when the file is served over http. If you host it, the instrument registers one and offers the matching `sw.js` for download. If you are running it from disk it says so and does not pretend otherwise.

---

## Version history

**1.7.0** &middot; Workflow pass. One job state feeding station notes, tab badges and a preflight panel. A persistent job bar. Units moved to the masthead. Progressive disclosure for fine typesetting, back matter and diagnostics. Printing numbered in the order you do it.

**1.6.0** &middot; Bench punch list, and an install layer honest about what `file://` already gives you.

**1.5.0** &middot; Bridges. Case parts to SVG and DXF, materials to BOM CSV, saved RSS or Atom in as a manuscript.

**1.4.0** &middot; Binding breadth. Coptic, longstitch and Japanese stab. Side sewn books switch the instrument to single leaves and a cut line. Rounding and backing became real arc geometry. Headbands checked against the square.

**1.3.0** &middot; Real PDF re-imposition. Full source PDF reader and Form XObject embedding, with a choice of fit, fill, or original size on the trimmed page.

**1.2.0** &middot; Typesetting. Total fit line breaking after Knuth and Plass, conservative rule hyphenation with a user exception list, widow and orphan control, drop caps, small caps chapter openings, running heads, and half title, dedication and colophon.

**1.1.0** &middot; Push out compensation, per edge trim, and collation marks: a signature letter at the tail of each section and a stepped bar across the spine fold.

**1.0.1** &middot; **Correction.** Quarto and sextodecimo had the wrong fold order. Both put the spine on the head edge, producing a section that could not be read without turning it a quarter turn. Folio and octavo were correct. Anything printed at quarto or sextodecimo before this version should be discarded.

**1.0.0** &middot; First build.

---

## Known limitations

Hyphenation is a conservative rule hyphenator, not Liang's pattern method. It knows affixes, doubled consonants and the consonant clusters English refuses to split, and it declines to guess. It will miss legitimate breaks. It should very rarely invent a wrong one. Add your own exceptions for words you use often.

Reading a PDF requires the browser's `DecompressionStream`, which every current browser has. Encrypted PDFs are refused rather than guessed at. A page whose content is split across several streams has to be expanded and re-joined, so those pages come out larger than they went in; the common single stream case is copied untouched.

Lines carrying a drop cap or a small caps opening are positioned word by word rather than as a single run, which slightly weakens copy and paste on those few lines only. Every other line is set as one string with PDF word spacing, and copies cleanly.

Character coverage is the printable ASCII range plus a map of common punctuation. No fonts are embedded: the PDF names the Adobe base fourteen and the viewer supplies them.

Push out is a geometric model of nested leaves. Real paper crushes at the fold, so the retained fraction is yours to set, and the honest way to set it is to fold a dummy section and measure it.

Swell, board and cloth allowances are working estimates for a bench, not shop specifications. Measure the block you actually sewed.

Rounding and backing are modelled as a circular arc. Real backing depends on how the paper takes a hammer, so the arc fraction is yours to set.

A side sewn book does not open flat and no arithmetic will fix that. The gutter allowance is a judgement; the instrument only warns when the sewing would run through the text.

The DXF carries closed polylines and circles, which is all flat rectangular parts and round holes need. The BOM column names are a proposal, not a standard.

Reading a feed strips it to plain text. Images, links and formatting do not survive, because a manuscript here is text with chapter marks and nothing else.

---

## Self test

The instrument carries its own tests. Press **Self test** in the header. Version 1.7.0 runs 286 assertions covering fold geometry, imposition invariants, page placement, trim and push out arithmetic, hyphenation, line breaking, page makeup, spine and case arithmetic, rounding and backing, every sewing pattern, the PDF writer, the PDF reader, and every export.

A headless harness runs the same instrument under jsdom for release checks and adds 113 more. Some of those are deliberately not arithmetic tests: they change the sewing style through the actual controls and assert that the change reaches the sewing marks, the case and the parts list. A bug in v1.4 development left the case arithmetic stale whenever a side sewn book was chosen, and every one of the 281 in-app assertions passed while it was broken, because the fault was in the wiring rather than the mathematics.

Exports are checked outside the instrument as well. The DXF is read back with `ezdxf`, the SVG parsed as XML, the BOM parsed with a CSV reader, and re-imposed PDFs are verified by rasterizing each sheet and comparing every placed cell against a rasterization of its original source page.

---

## License

GPL-3.0

---

Make. Hack. Learn. Share. Repeat.
