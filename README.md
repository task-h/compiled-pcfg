# compiled_pcfg

> **Fork notice:** This is a modified fork of
> [lakiw/compiled-pcfg](https://github.com/lakiw/compiled-pcfg) by Matt Weir,
> with Linux (LF line-ending) build/run fixes. See [`CHANGES.md`](CHANGES.md)
> for the list of modifications. Licensed under GPL v2, same as upstream.

A version of the PCFG password guess generator written in pure C. Goal is to make it faster and easier to include in other password cracking tools such as Hashcat and John the Ripper. 


**Note:** This is only the guess generator. 


Current plans are to leverage the Python PCFG trainer to generate rulesets. The trainer can be found at: https://github.com/lakiw/pcfg_cracker
