test-md2xliff
==

I'm trying to figure out if we really can use md2xliff for activate.mozilla.community. We've had several problems in the past and I'm trying to do a real use-case in this repository. I will write all commands I'm running and every step I take in this README and will commit after every step to be able to show results.

The goal for using md2xliff is that we have Markdown files which are the base of all content. We want to make it easier for localizers to translate these files. There is [Pontoon](https://pontoon.mozilla.org) which would have a great interface to translate strings and add them back to any repository. This obviously doesn't handle markdown since markdown is not really used for translations. Our best option would be [xliff](https://en.wikipedia.org/wiki/XLIFF), which is a well-used standard.

This repository tries some stuff around the md2xliff script. If that will not work (which is possible), we will also go into further detail regarding other possibilities to have well-known string files.

1. Initial commit
---

I've set up two scripts to do md -> xliff (extract) and xliff -> md (reconstruct). I've additionally created a ```test.md``` file which holds the first iteration of our content.

2. Do initial extraction
---

Now we are doing the initial extraction to an xliff file. We are using the ```test.md``` (in English) to create the xliff to translate to de-CH.

```
mkdir de-CH
cp test.md de-CH/test.md
mkdir -p locales/de-CH/
./updateXLFFromMD.sh
```

Observations:

* Possible blocker: We have extracted "default" to the xliff file, which will get translated, this will break Jekylls. Would it be better to extract "layout: default"? Hard to say, since localizers might translate that as well.. (This is the fork I have done of md2xliff, this would be easily fixable I think)
* The skeleton looks good
* Apart from weird indetation in the XML file, the extracted strings look good (easily fixable, Pontoon would even do that for us at the end..)

So far so good!

3. Translation of initial text
----

Now the initial text get's translated. I didn't use any tool for this, I just edited the xliff manually. This leads to an updated xliff (as Pontoon would do it), and our content is not updated yet. I've taken care to not break Jekylls (see above, this might be a possible blocker).

Es sind jedoch noch nicht alle Strings übersetzt.

4. Reconstruction
----

Now we want to update our markdown file for German.

```
./updateMDFromXLF.sh
```

Observations:
* Fallback for the two empty strings worked
* Reconstruction worked perfectly fine, we now have all text in German apart from the missing translations which are still English

So far so good as well!

5a. English text gets updated
----

Now we are updating a single paragraph in English and we'll run an extraction again. This is the first case we had problems with MozActivate. Let's see how this turns out.

```
<edit test.md file>
./updateXLFFromMD.sh
```

which specifically does

```
node_modules/md2xliff/bin/extract de-CH/test.md locales/de-CH/test.xlf locales/de-CH/test.skl.md en-US de-CH
```

Observations:
* This didn't work. We now have the already translated strings as <source> in the xliff and not our en-US strings..

Example:

```
<trans-unit id="7" xml:space="preserve">
  <source xml:lang="en-US">Bin ich ein super Test? Dies könnte so weitergehen..</source>
  <target xml:lang="de-CH"></target>
</trans-unit>
```

This definitely makes sense, since it can't possibly know what I want to take as base. We need to find another way to do this. This will be step 5b.

Step 5a is not in the repository, as I did step 5b at the same time.

5b. Use EN test.md as base
---

We most certainly need to use the original test.md as a base. Let's try this.

```
node_modules/md2xliff/bin/extract test.md locales/de-CH/test.xlf locales/de-CH/test.skl.md en-US de-CH
```

This looks better in terms of the <source>, but we have lost all de-CH translations in the <target>. It basically created a new xlf file from what it had. We need to look further into the code on how we could do an update.

Questions to find an answer:
* How can we do updates at extraction?

Idea: as our structure hasn't changed, we still have the same IDs in the xlf. We could write a script that takes the previous xlf and puts the old <target> strings in it. For this we would need to make sure that Pontoon picks up the change in the <source> so it shows the localizer that it changed. As this is (hopefully) as Pontoon works, this shouldn't be a problem. However, that would most probably only work if there are no structural changes in the skeleton. This needs to be tested further (see next section covering what hasn't been tested yet).

TODO - not tested yet
----

* What happens if we delete whole lists? What happens to the skeleton? Can the extraction possibly match the different parts of the xliff it needs to update?
* What happens if we do a rewrite of a part of the text? What happens to the skeleton? Can the extraction possibly match the different parts of the xliff it needs to update?
* What if we change the structure of the text and add title in between? What happens to the skeleton? Can the extraction possibly match the different parts of the xliff it needs to update?
