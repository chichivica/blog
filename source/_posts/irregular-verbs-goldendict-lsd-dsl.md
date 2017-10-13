---
title: Irregular verbs dictionary for Goldendict
date: 2017-10-13 12:32:01
tags:
---
# Irregular verbs dictionary LSD, DSL

[Here is](https://dic.1963.ru/rec/194) irregular verbs dictionary made by **Igor Mostitsky** which contains:

_More than 500 irregular verbs; more than 300 verbs have translations (English to Russian actually). Most important for learning verbs have specific mark (importance rate):_ ![](https://i.imgur.com/lPNampD.png)
_one bold point corresponds to one rank level. 5 is the highest rank. Most popular verbs has transcription._

## Here are two links to download either DSL or LSD files
* Original archive [En-Ru_Most_Irregular_Verbs_v_2_1_LSD_x5.zip](https://yadi.sk/d/h6qf4Mp63NidmA) contains __*.ann__ __*.lsd__ and __*.dsl.files.zip__
* Coverted files in [Most_Irregular_Verbs.tar](https://yadi.sk/d/pqW29EQm3Nie7E) __*.ann__ __*.bmp__ __*.dsl__ and  dsl compressed file __*.dsl.dz__ which has the same content as __*.dsl__

__NB:__ *.lsd binary format intended for AbbyLingvo x5 and could not be used with GoldenDict desktop version only in mobile.


## Convertion

In order to convert __*.lsd__ AbbyLingvo binary file I used cool [lsdreader](https://github.com/sv99/lsdreader) project (python2 applicable):

```
pip2 install lingvoreader
lsdreader -i En-Ru_Most_Irregular_Verbs.lsd -o decoded/

```

In order to compress generated __DSL__ to __*.dsl.dz__ file i used [dictzip](https://linux.die.net/man/1/dictzip) executable which supplied in apt-get in Debian

```
sudo apt-get install dictzip
dictzip En-Ru_Most_Irregular_Verbs.dsl
```
will produce En-Ru_Most_Irregular_Verbs.dsl.dz

[Here is](https://github.com/goldendict/goldendict/wiki/Supported-Dictionary-Formats) comprehensive list of supported by GoldenDict formats

