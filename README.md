test-md2xliff
==

I'm trying to figure out if we really can use md2xliff for activate.mozilla.community. We've had several problems in the past and I'm trying to do a real use-case in this repository. I will write all commands I'm running and every step I take in this README and will commit after every step to be able to show results.

Initial commit
---

I've set up two scripts to do md -> xliff (extract) and xliff -> md (reconstruct). I've additionally created a ```test.md``` file which holds the first iteration of our content.

Do initial extraction
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
