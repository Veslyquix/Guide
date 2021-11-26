# Guide
 
Table of Contents
-

---



[Introduction](https://github.com/Veslyquix/Guide#understanding-ea-event-assembler)  
[Labels vs Definitions](https://github.com/Veslyquix/Guide#labels-vs-definitions)  
[Graphics Overview](https://github.com/Veslyquix/Guide#graphics-overview)  
[To `POIN` or not to `POIN`?](https://github.com/Veslyquix/Guide#to-poin-or-not-to-poin)  
[Text](https://github.com/Veslyquix/Guide#text)  
[Chapter Events](https://github.com/Veslyquix/Guide#chapter-events)  
[Conditional Installing](https://github.com/Veslyquix/Guide#conditional-installing)  
[Additional Terms/Commands](https://github.com/Veslyquix/Guide#additional-terms)  
[Item Icons](https://github.com/Veslyquix/Guide#item-icons)  
[Map Sprites](https://github.com/Veslyquix/Guide#map-sprites)  
[~~Portraits~~]  
[Resources](https://github.com/Veslyquix/Guide#resources)  
[Advanced](https://github.com/Veslyquix/Guide#advanced)  

---





Understanding EA (Event Assembler)  
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>



Your .gba rom is made up of `BYTE`s, or digits from $00 to $FF, expressed as 0 to 255 in decimal. It is mostly unintelligable. 

![Alt-text](/png/Gibberish.png?raw=true "Optional Title")

But I don't understand the hex data! Is this a problem? 

Good. Raw hex data is not supposed to be read by humans. 

Your rom can fit about 32 million of these `BYTE`s, half or so of which is free space. 


- Goal: Write BYTEs to the rom in fancy ways to minimize work in making a hack. 


.event is a .txt, best viewed in a program like notepad++ or sublime.


In EA, numbers are written in decimal unless prefixed by 0x or $. 

Eg.
``` 
30 = #0x1E
255 = $FF 
```

Calc.exe can convert between hex and dec in Programmer mode. We'll mostly use decimal. 






![Alt-text](/png/Seth_Str.png?raw=true "Optional Title")

`#define BaseStrOffset 0x0D `

// We'll need this soon 



EA automatically writes to our `CURRENTOFFSET`, which is where we've `ORG`'d to or more typically at the next part of free space (eg. expanding the rom past 16 mb). 


`PUSH` and `POP` are essentially brackets ( ) to put around `ORG`. 

Eg. 

```
PUSH 
ORG $803DA5 // Character Table - Seth's Str (see above image)
BYTE 7+5 10+3 // Give him base 12 Str and 13 Skl
POP 
```

Failing to put these around an `ORG` will brick your resulting rom. 

In EA, we can do simple math and define things for ease.  

`#define Seth 2`  
// Seth is character ID of 2.  

We can also comment out the rest of a line with slashes `//` or comment out multiple lines with `/*    */`. 


Let's generalize a way to write base stats to *any* character!

Your CharacterTable probably starts at $803D30. see root/Tables/Table Definitions.txt

> `#define CharacterTable 0x803D30`



![Alt-text](/png/CharTableSize.png?raw=true "Optional Title")


```
#define CharTableSize 52
```
First we define its size.


```
#define CurrentChar Franz
```
First, we define which character to edit. `Franz` is defined as his character ID, `4`, in EA's standard library (`EAstdlib.event`).  


`PUSH`  
Open bracket.  

`ORG CharacterTable`  
Address to originate at.   

`ORG CURRENTOFFSET + (CharTableSize * CurrentChar)`   
Each entry has 52 bytes, so the character entry we want is equal to Size*CharacterID.   

`ORG CURRENTOFFSET + BaseStrOffset`  
Now we go forward $0D bytes to get to Str.    

`BYTE (5*2)`   
Give Franz base 10 Str   

`POP`   
Close bracket.   

`#undef CurrentChar`  
Undefine `CurrentChar` so we can use it again without EA giving us a warning.   


This absolutely works, but it's not ideal to write out so much every time. Let's consolidate it!  


```
#define CharEntry(CharID) "ORG CharacterTable + (CharTableSize * CharID) + BaseStrOffset"
``` 
We've put all of the ORG part into one line now, as the math could be done without `CURRENTOFFSET`. 

```
PUSH 
CharEntry(Gilliam)
BYTE $A $B // Base 10 Str and 11 Skl

CharEntry(Matthew)
BYTE 8 14 // Base 8 Str and 14 Skl
POP 
```

Or...

```
#define CharBaseStr(CharID, Value) "PUSH; ORG CharacterTable + (CharTableSize * CharID) + BaseStrOffset; BYTE Value; POP"
```
We've added all lines into our command by using `;` for line breaks. 

Now we simply type this to edit a unit's base str: 
```
CharBaseStr(Eirika, 14) 
CharBaseStr(Tana, 11) 
```
However, this version only edits Str, rather than all the stats in a row. 

Typically for repetitive tasks, we write out **macros** like the above ones to consolidate. In this case, most users edit Character & Class data in a .CSV file. 

For large tables we usually use **.CSV** (which can be processed as part of MAKEHACK_FULL), while for smaller ones and frequent commands we use **macros**. 

---


Labels vs Definitions
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>

A Label marks an address, while a definition lets a written word equate to a number. 


```
#define MyDefinition 13 // I'm unlucky 

PUSH 
ORG $6789A

ALIGN 4
MyLabel: // Equivalent to $6789C, as it's the next multiple of 4 (ending in 0, 4, 8, or C) after where we ORG'd to. 
BYTE MyDefinition // Write the byte $0D (13) at this address. 
POP 
```

Labels have a colon `:` at the end and almost always have `ALIGN 4` before them.

Tip: Typically definition files are `#include`d before others so that your .CSV tables and .event files can refer to them. 

Neither a label nor definition actually adds any data to the rom. 


---


Graphics Overview
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>

Generally speaking, these are the steps: 

1. Format image correctly (FEBuilder's "Color Reduction Tool" may be of use here). 
2. Run the corresponding batch script to process your images.
3. #incbin them. Some installers can be generated automatically. Please see this: https://github.com/Veslyquix/EasyBuildfile/tree/main/Graphics
4. Add an entry to the relevant table. 

Warning!

Your ~16+ mb of free space would be used up by about 90 seconds of uncompressed audio or 4 seconds of uncompressed gba screen sized video. You may be able to squeeze longer lengths in with compression, but when it comes to graphics & sound, we must be efficient with the data. A buildfile is much more efficient with space than FEBuilderGBA, though, so you probably have nothing to worry about. 

---


To `POIN` or not to `POIN`? 
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>

$00123456 vs $08123456 

`ORG` does not include the `8` or `9` at the start. But we need it the rest of the time when referring to an address. 

Therefore, we use POIN while using EA, or we use the defined `IsPointer` (as `0x8000000`) and append it in CSV cells. 

```
ALIGN 4
ImageData:
#incbin "Filename.dmp"

TableOfImages:
POIN ImageData

// Or we could do:
TableOfImages:
WORD ImageData|IsPointer
```

`#incbin "filename.dmp"` will write the contained hex data to your rom. Similarly, `#include "filename.lyn.event"` and `#incext "filename.png" png2dmp` will install hex data. 


---


Text
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>

Text is typically only edited within the `Root/Text` subfolder. You *can* use the String("Text") macro for the rare simple thing, but I believe it's only useful for raw text (eg. lacking shortforms like [OpenRight]). 

![Alt-text](/png/TextFolderLayout.png?raw=true "Optional Title")

`InstallTextData.event` and `TextDefinitions.event` are generated and should not be edited. 

Within `text_buildfile.txt` you can `#include "folder/filename.txt"` to better organize your files. 

You may sometimes edit `ParseDefinitions.txt` to add new shortforms for loading a specific portrait. 
[LoadInnes] = [LoadFace][0xf][0x1]

Text uses specific formatting that I won't go into detail on here, but as a quick and dirty introduction:

```
# 0x0903 // Text ID 
Text goes here.[N]
And here.[A][X]

## OpeningDialogue // "OpeningDialogue" will become a definition
Welcome to the[N]
world of Pokemon![A][X]
```

In `TextDefinitions.event`, we'll see:
`#define OpeningDialogue $904`, such that we don't need to remember text IDs. 

We can also use NarrowFont (if it's installed):

```
#0x955 SteelGreatlanceName ^
Steel Greatlance[X]
```

Trainer Tips: Be careful not to have a space after [X], as TextProcess will error! 

Further reading: https://feuniverse.us/t/the-ins-and-outs-of-text-editing/6820


---

Chapter Events
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>

Sme wrote an excellent guide on this. You must read it. https://feuniverse.us/t/fe8-ea-eventing-guide/7080

I don't have much helpful advice here. I suggest organizing one chapter to your preference and then using it as a template. 

You can also use custom definitions here to make things easier on yourself. 

Eg.

```
#define GenericEnemyLevel 3 
...
GenericEnemyUnitGroup:
UNIT 0x50  Brigand 0x0 Level(GenericEnemyLevel+1, Enemy, 1)   [8,3] 0b 0x0 0x01 REDA8R3 [IronAxe,0x0,0x0,0x0] AttackInRangeAI 		// Level 4
UNIT ONeil Brigand 0x0 Level(GenericEnemyLevel+2, Enemy, 1)   [10,5] 0b 0x0 0x00 0x00   [SteelAxe,0x0,0x0,0x0] NeverMoveAI 		// Level 5
UNIT 0x50  Brigand 0x0 Level(GenericEnemyLevel,   Enemy, 1)   [5,2] 0b 0x0 0x01 REDA5R2 [IronAxe,0x0,0x0,0x0] PursueWithoutHeedAI // Level 3
UNIT
...
#undef GenericEnemyLevel // So we can use the same definition next chapter. 
```

REDA definitions can be found from Snek's asm thread. 




---


Conditional Installing
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>

`#ifdef` and `#ifndef` lets you install something or not based on whether a definition has been made or not. 

```
#ifdef MyDefinition 
	#include "EngineHacks/AoE/Installer.event"
#endif 


#ifndef AnotherDefinition
	#define AnotherDefinition MyDefinition // Now they're both 13
#endif 
```

---


Additional terms:
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>

Word - 4 bytes 

Short - 2 bytes 

Byte - two digits (eg. 27 = **0x1B** = b00011011) 

Bit - 1/8th of a byte, or one digit of the byte when expressed in binary.

Bitfield: 

Instead of counting to 255 with a byte, we use each bit in binary as a yes / no flag. 


Common abbreviations:  
  
MMS - Moving Map Sprite   
SMS - Standing Map Sprite   
  
Hacks:  
  
MSS - Modular Stat Screen   
MMB - Modular Minimug Box   
EMS - Expanded Modular Save   



More Commands
-

These commands are mostly used as a sanity check. (Eg. to ensure you remembered to balance your PUSH / POP commands and have been writing to Free Space, or to check that hacks do not conflict.)

```
MESSAGE CURRENTOFFSET  
PROTECT $789AB // The byte at this address cannot be written to (EA will error). 
PROTECT $789AB $78A00 // The bytes in this range cannot be written to. 
```



---


Item Icons
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>

![Alt-text](/png/ItemIconsPreview.png?raw=true "Optional Title")

- Put item icons into the `png` folder. 

![Alt-text](/png/ItemIconsSteps.png?raw=true "Optional Title")

- Run "GenerateMapSpritesInstaller.bat"
- - This will also rename your images, removing `,() {}-`. 
- Run "Png2DmpImages.bat"
- - You may need to ensure that Png2Dmp.exe has the correct relative path. 
- - Eg. Root/Somefolder/THISFOLDER is where we are.
- - but Root/EventAssembler/Tools/Png2Dmp.exe is where it expects it. 



Copy the ItemIconID definitions into your relevant CustomDefinitions file. 
Eg. 
```
#define BlankIcon 222 
#define Scythe_By_BeansyIcon 223 
```

You can now refer to these definitions in your ItemTable.csv to make items use said icon. 

---


Map Sprites
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>

Put standing map sprites into the `sms` folder and moving map sprites into the `mms` folder. 

- Run "GenerateMapSpritesInstaller.bat"
- - This will also rename your images, removing `,() {}-`. 
- Run "Png2DmpImages.bat"
- - You may need to ensure that Png2Dmp.exe has the correct relative path. 
- - Eg. Root/Somefolder/THISFOLDER is where we are.
- - but Root/EventAssembler/Tools/Png2Dmp.exe is where it expects it. 

Copy part of "GeneratedInstaller.event" into "Installer.event" to edit the SMS Size or MMS AP. 

Copy the definitions into your relevant CustomDefinitions file. 
Eg.
```
#define Cavalier_M_Sword_SALVAGEDstand 107 
#define Cavalier_M_Sword_SALVAGEDwalk 127 
```

MMS is the same as class ID and is not set in your class table, so keep that in mind. Eg. if you want to replace EirikaLord's MMS, you must have the definition be 1. 

SMS is set in your ClassTable.CSV, and you can now use the definitions you added in there. 


---

Resources
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>

To make your event or asm files have pretty colours for key words, please install these language files to your text editor:

https://feuniverse.us/t/syntax-highlighting-for-event-assembler/2131

https://feuniverse.us/t/asm-notepad-thumb-assembly-syntax-highlighting/529

---


Advanced
-
<p align="right">
<a href="https://github.com/Veslyquix/Guide#table-of-contents">Top</a>
</p>

Do not bother reading this section until you actually want to do asm yourself. 

Injecting Custom Code
-

If you want to change the mechanics of the game, you want to **hook** an address (which must be ALIGN 4'd aka ending in 0, 4, 8, or C), or replace a vanilla `POIN`. 
ASM wise, you want to ensure there's a free register to use and that you `PUSH` & `POP` registers you use. You can then return to the vanilla function if desired by loading the address into a register, and `bx`ing to it (if you used jumpToHack), or by `POP`ing `lr` into a **scratch** register and `bx`ing to it (if you used callHack). 

Finding a POIN: 

0x8034314 CanUnitUseVisit // From FE8_Clean.sym

08 03 43 14 -> 14 43 03 08 // Little Endian 

Search FE8_Clean.gba with HxD.exe for `14 43 03 08` or `15 43 03 08`. (It must be ALIGN 4'd aka at an address that ends in 0, 4, 8, or C.)

In this case, nothing was found, so there are no POINs to it, and our only option is to hook it. 

```
PUSH 
ORG $2AAD0 
callHack_r3(StabBonusFunc|1) // When using asm / thumb, addresses must end in 1, 5, 9, or D. `|1` achieves this. 

ORG $9B788 
jumpToHack_r2(DisplayDurabilitySupply) 
SHORT 0x46C0 // Replace the code `ldrh r6, [r0]` with NOP / no operation.  
POP

ALIGN 4 
StabBonusFunc: // Address will end in 0, 4, 8, or C, so references to it must include `|1` at the end. 
#incbin "StabBonus.dmp"

#include "DisplayDurabilitySupply.lyn.event"
```
