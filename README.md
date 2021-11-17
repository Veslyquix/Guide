# Guide
 


File types - 
.event is a .txt, best viewed in a program like notepad++ or sublime
How to make notepad++ show certain colours for key words in asm / ea (install these language files into notepad++)

EA Terms 

Understanding EA (Event Assembler)
Goal: Write BYTEs to the rom in fancy ways. 

<style>
r { color: Red }
o { color: Orange }
g { color: Green }
</style>

# TODOs:

- <r>TODO:</r> Important thing to do
- <o>TODO:</o> Less important thing to do
- <g>DONE:</g> Breath deeply and improve karma


#define Seth 2 // Seth is character ID of 2. 

![Alt-text](/png/Seth_Str.png?raw=true "Optional Title")
#define BaseStrOffset 0x0D // We'll need this soon 

EA automatically writes to our CURRENTOFFSET, which is typically at the end of our rom. 


PUSH and POP are essentially brackets ( ) to put around ORG 
Eg. 
PUSH 
ORG $803DA5 // Character Table - Seth's Str
BYTE 20 15 // Give him base 20 Str and 15 Skl
POP 


Failing to put these around an ORG will brick your resulting rom. 


What if we want to write stats to a bunch of characters?
Your CharacterTable probably starts at $803D30. see root/Tables/Table Definitions.txt
> #define CharacterTable 0x803D30

Let's define its size! 

![Alt-text](/png/CharTableSize.png?raw=true "Optional Title")

#define CharTableSize 52 

Now let's make it easy to edit anyone's base stats. 

#define CurrentChar Franz
PUSH 
ORG CharacterTable
ORG CURRENTOFFSET + (CharTableSize * CurrentChar)
ORG CURRENTOFFSET + BaseStrOffset
BYTE 12 // Give Franz base 12 Str 
POP 
#undef CurrentChar 

This absolutely works, but it's not ideal to write out so much every time. Let's consolidate it! 

//#define BaseStatsOffset 0x0C 
#define CharEntry(CharID) "ORG CharacterTable + (CharTableSize * CharID) + BaseStrOffset"

PUSH 
CharEntry(Gilliam)
BYTE 13 11 // Base 13 Str and 11 Skl

CharEntry(Matthew)
BYTE 8 14 // Base 8 Str and 14 Skl
POP 

Or...

```ea
#define CharBaseStr(CharID, Value) "PUSH; ORG CharacterTable + (CharTableSize * CharID) + BaseStrOffset; BYTE Value; POP"
```

Now we simply type this to edit a unit's base str: 
CharBaseStr(Eirika, 14) 
CharBaseStr(Tana, 11) 

However, if we want to also edit skill that comes immediately afterwards, we'd need to create another version of the above macro. 




Adding 


These commands are mostly used as a sanity check. (Eg. to ensure you remembered to balance your PUSH / POP commands and have been writing to Free Space.
MESSAGE CURRENTOFFSET  
PROTECT $789AB // The byte at this address cannot be written to (EA will error). 
PROTECT $789AB $78A00 // The bytes in this range cannot be written to. 



#define 
#ifdef 
#ifndef 
#endif 
|IsPointer 
ALIGN 4 
labels 
Macros
Hooks 
(jumpToHack etc.) 
General layout of skillsys buildfile 

Tables - two main ways to edit a vanilla or custom table: 
- Using .CSV files which can be processed as part of MAKEHACK_FULL 
- or using a macro 

While you could make a table like this: 
// Required Level, GaidenSpell 
BYTE 20 Fire 
BYTE 0 0

It is generally more readable with bigger tables to use macros, such as: 
GaidenSpellEntry(Level, Spell)

Your .gba rom is made up of purely hex data. It is mostly unintelligable like this, except for the occasional POIN. 
#incbin "filename.dmp" will write the contained hex data to your rom. Similarly, #include "filename.lyn.event" and #incext "filename.png" png2dmp will install hex data. 

In EA, numbers are written in decimal unless prefixed by 0x or $. 
Eg. 
30 = #0x1E
255 = $FF 
Calc.exe can convert between hex and dec in Programmer mode 

Faq - I don't understand the hex data. Is this a problem? 
Good. Raw hex data is not supposed to be read by humans. 

Word - 4 bytes 
Short - 2 bytes 
27 = 0x1B = b00011011
Byte - two digits (eg. 0x7F = 127 = b11,111,111) 
Bit - 1/8th of a byte (b01,111,111) 

Bitfield: 
Instead of counting to 255 with a byte, we instead use each bit as a yes / no flag. 


Your rom starts at 16 megabytes (16 million bytes) and can be expanded up to 32 mb. 
This free space would be used up by about 90 seconds of uncompressed audio or 4 seconds of gba screen sized video. With compression


