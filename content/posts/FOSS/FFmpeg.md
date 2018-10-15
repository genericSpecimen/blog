---
title: "FFmpeg"
date: 2018-10-14T15:38:55+05:30
draft: true
---

To quote the [FFmpeg webiste](https://ffmpeg.org/), FFmpeg is a complete, cross-platform solution to record, convert and stream audio and video. The documentation over there is beautiful. This post will give a basic overview of the ```ffmpeg``` tool.

```ffmpeg``` reads from an arbitrary number of input "files" (which can be regular files, pipes, network streams, grabbing devices, etc.), specified by the ```-i``` option, and writes to an arbitrary number of output "files", which are specified by a plain output url. Anything found on the command line which cannot be interpreted as an option is considered to be an output url. 

To refer to input files in options, you must use their indices (0-based). E.g. the first input file is ```0```, the second is ```1```, etc. Similarly, streams within a file are referred to by their indices. E.g. ```2:3``` refers to the fourth stream in the third input file. As a general rule, first specify all the input files and then all the output files. All options apply only to the next input or output file and are reset between files.

### Basic format conversions
