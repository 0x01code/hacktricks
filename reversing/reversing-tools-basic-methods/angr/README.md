<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **जुड़ें** या **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


# स्थापना
```bash
sudo apt-get install python3-dev libffi-dev build-essential
python3 -m pip install --user virtualenv
python3 -m venv ang
source ang/bin/activate
pip install angr
```
# मूलभूत कार्रवाई

## Introduction
## परिचय

This section provides an overview of the basic actions that can be performed using angr.

इस खंड में एनग्र का उपयोग करके किए जा सकने वाले मूलभूत कार्रवाइयों का अवलोकन प्रदान किया जाता है।

## Loading a Binary
## एक बाइनरी लोड करना

To analyze a binary using angr, you first need to load the binary into an angr project.

angr का उपयोग करके एक बाइनरी का विश्लेषण करने के लिए, आपको पहले एक एनग्र प्रोजेक्ट में बाइनरी लोड करना होगा।

```python
import angr

project = angr.Project("/path/to/binary")
```

## Finding Entry Point
## प्रवेश बिंदु खोजना

The entry point of a binary is the address where the execution of the program starts. You can find the entry point using the `entry` attribute of the angr project.

एक बाइनरी का प्रवेश बिंदु कार्यक्रम के निष्पादन की शुरुआत होती है जहां पता चलता है। आप एनग्र प्रोजेक्ट के `entry` गुण का उपयोग करके प्रवेश बिंदु खोज सकते हैं।

```python
entry_point = project.entry
```

## Exploring the Control Flow Graph (CFG)
## नियंत्रण फ़्लो ग्राफ़ (CFG) का अन्वेषण करना

The Control Flow Graph (CFG) represents the flow of execution within a binary. You can explore the CFG using the `cfg` attribute of the angr project.

नियंत्रण फ़्लो ग्राफ़ (CFG) एक बाइनरी में निष्पादन के फ़्लो को प्रतिष्ठित करता है। आप एनग्र प्रोजेक्ट के `cfg` गुण का उपयोग करके CFG का अन्वेषण कर सकते हैं।

```python
cfg = project.analyses.CFG()
```

## Finding Functions
## फ़ंक्शन खोजना

To find the functions present in a binary, you can use the `functions` attribute of the angr project.

एक बाइनरी में मौजूद फ़ंक्शन्स को खोजने के लिए, आप एनग्र प्रोजेक्ट के `functions` गुण का उपयोग कर सकते हैं।

```python
functions = project.kb.functions
```

## Symbolic Execution
## प्रतीकात्मक निष्पादन

Symbolic execution is a technique used to explore all possible paths of execution within a binary. You can perform symbolic execution using the `explorer` attribute of the angr project.

प्रतीकात्मक निष्पादन एक तकनीक है जिसका उपयोग बाइनरी के भीतर सभी संभावित पथों का अन्वेषण करने के लिए किया जाता है। आप एनग्र प्रोजेक्ट के `explorer` गुण का उपयोग करके प्रतीकात्मक निष्पादन कर सकते हैं।

```python
explorer = project.surveyors.Explorer(start=entry_point, find=[target_address])
explorer.run()
```

## Conclusion
## निष्कर्ष

These are some of the basic actions that can be performed using angr. By understanding and utilizing these actions, you can effectively analyze and explore binaries using angr.

ये कुछ मूलभूत कार्रवाइयाँ हैं जिनका उपयोग एनग्र का उपयोग करके किया जा सकता है। इन कार्रवाइयों को समझकर और उपयोग करके, आप एनग्र का उपयोग करके बाइनरी का विश्लेषण और अन्वेषण कर सकते हैं।
```python
import angr
import monkeyhex # this will format numerical results in hexadecimal
#Load binary
proj = angr.Project('/bin/true')

#BASIC BINARY DATA
proj.arch #Get arch "<Arch AMD64 (LE)>"
proj.arch.name #'AMD64'
proj.arch.memory_endness #'Iend_LE'
proj.entry #Get entrypoint "0x4023c0"
proj.filename #Get filename "/bin/true"

#There are specific options to load binaries
#Usually you won't need to use them but you could
angr.Project('examples/fauxware/fauxware', main_opts={'backend': 'blob', 'arch': 'i386'}, lib_opts={'libc.so.6': {'backend': 'elf'}})
```
# लोड की गई डेटा

## Main object information

## मुख्य ऑब्जेक्ट की जानकारी
```python
#LOADED DATA
proj.loader #<Loaded true, maps [0x400000:0x5004000]>
proj.loader.min_addr #0x400000
proj.loader.max_addr #0x5004000
proj.loader.all_objects #All loaded
proj.loader.shared_objects #Loaded binaries
"""
OrderedDict([('true', <ELF Object true, maps [0x400000:0x40a377]>),
('libc.so.6',
<ELF Object libc-2.31.so, maps [0x500000:0x6c4507]>),
('ld-linux-x86-64.so.2',
<ELF Object ld-2.31.so, maps [0x700000:0x72c177]>),
('extern-address space',
<ExternObject Object cle##externs, maps [0x800000:0x87ffff]>),
('cle##tls',
<ELFTLSObjectV2 Object cle##tls, maps [0x900000:0x91500f]>)])
"""
proj.loader.all_elf_objects #Get all ELF objects loaded (Linux)
proj.loader.all_pe_objects #Get all binaries loaded (Windows)
proj.loader.find_object_containing(0x400000)#Get object loaded in an address "<ELF Object fauxware, maps [0x400000:0x60105f]>"
```
## मुख्य उद्देश्य

The main objective of this document is to provide an introduction to the angr framework and its basic usage. This document will cover the installation process, basic concepts, and common use cases of angr. By the end of this document, you should have a good understanding of how to use angr for binary analysis and reverse engineering tasks.
```python
#Main Object (main binary loaded)
obj = proj.loader.main_object #<ELF Object true, maps [0x400000:0x60721f]>
obj.execstack #"False" Check for executable stack
obj.pic #"True" Check PIC
obj.imports #Get imports
obj.segments #<Regions: [<ELFSegment flags=0x5, relro=0x0, vaddr=0x400000, memsize=0xa74, filesize=0xa74, offset=0x0>, <ELFSegment flags=0x4, relro=0x1, vaddr=0x600e28, memsize=0x1d8, filesize=0x1d8, offset=0xe28>, <ELFSegment flags=0x6, relro=0x0, vaddr=0x601000, memsize=0x60, filesize=0x50, offset=0x1000>]>
obj.find_segment_containing(obj.entry) #Get segment by address
obj.sections #<Regions: [<Unnamed | offset 0x0, vaddr 0x0, size 0x0>, <.interp | offset 0x238, vaddr 0x400238, size 0x1c>, <.note.ABI-tag | offset 0x254, vaddr 0x400254, size 0x20>, <.note.gnu.build-id ...
obj.find_section_containing(obj.entry) #Get section by address
obj.plt['strcmp'] #Get plt address of a funcion (0x400550)
obj.reverse_plt[0x400550] #Get function from plt address ('strcmp')
```
## प्रतीक और पुनर्स्थापना

Symbols and relocations are important concepts in reverse engineering and binary analysis. They play a crucial role in understanding and manipulating the behavior of a binary executable.

प्रतीक और पुनर्स्थापना, रिवर्स इंजीनियरिंग और बाइनरी विश्लेषण में महत्वपूर्ण अवधारणाएं हैं। ये एक बाइनरी एक्जीक्यूटेबल के व्यवहार को समझने और प्रबंधित करने में महत्वपूर्ण भूमिका निभाते हैं।

### Symbols

प्रतीक (symbols) बाइनरी फ़ाइल में विभिन्न वस्तुओं को प्रतिष्ठित करते हैं, जैसे कि फ़ंक्शन, वेरिएबल्स, या अन्य संगठनात्मक यूनिट्स। प्रतीकों का उपयोग बाइनरी कोड के विभिन्न हिस्सों को पहचानने और संदर्भित करने के लिए किया जाता है। प्रतीकों को विभिन्न तरीकों से प्रदर्शित किया जा सकता है, जैसे कि लेबल, संकेत, या वेरिएबल्स के नाम।

### Relocations

पुनर्स्थापना (relocations) बाइनरी कोड में संशोधन करती हैं ताकि उसे अन्य विभिन्न पतों पर संदर्भित किया जा सके। जब एक बाइनरी कोड को लिंक किया जाता है, तो पुनर्स्थापना एंट्रीज़ उत्पन्न होती हैं जो बाइनरी कोड के विभिन्न सेक्शन्स को अन्य संदर्भित पतों पर ले जाती हैं। पुनर्स्थापना एंट्रीज़ के द्वारा, बाइनरी कोड को अन्य वस्तुओं के साथ संबंधित किया जा सकता है, जैसे कि लाइब्रेरी फ़ंक्शन्स या वेरिएबल्स।

पुनर्स्थापना एंट्रीज़ को विभिन्न तरीकों से प्रदर्शित किया जा सकता है, जैसे कि पुनर्स्थापना टेबल, पुनर्स्थापना ब्लॉक, या पुनर्स्थापना एंट्रीज़ के ऑफ़सेट।

यह जानना कि प्रतीक और पुनर्स्थापना कैसे काम करते हैं बाइनरी एक्जीक्यूटेबल को विश्लेषण करने और उसे मानव-पठनीय रूप में समझने में मदद करेगा।
```python
strcmp = proj.loader.find_symbol('strcmp') #<Symbol "strcmp" in libc.so.6 at 0x1089cd0>

strcmp.name #'strcmp'
strcmp.owne #<ELF Object libc-2.23.so, maps [0x1000000:0x13c999f]>
strcmp.rebased_addr #0x1089cd0
strcmp.linked_addr #0x89cd0
strcmp.relative_addr #0x89cd0
strcmp.is_export #True, as 'strcmp' is a function exported by libc

#Get strcmp from the main object
main_strcmp = proj.loader.main_object.get_symbol('strcmp')
main_strcmp.is_export #False
main_strcmp.is_import #True
main_strcmp.resolvedby #<Symbol "strcmp" in libc.so.6 at 0x1089cd0>
```
## खंड

Blocks are the basic units of code in the angr framework. A block represents a sequence of instructions that are executed sequentially without any branching or jumping. Each block starts at a specific address and ends at the address of the next block or at a branch instruction.

खंड अंग्र फ्रेमवर्क में कोड की मूल इकाइयां हैं। एक खंड एक ऐसी निर्देशिका को प्रतिष्ठित करता है जिसमें कोई शाखा या जंपिंग के बिना क्रमशः निष्पादित होने वाले निर्देश होते हैं। प्रत्येक खंड एक विशेष पते पर शुरू होता है और अगले खंड के पते या एक शाखा निर्देश पर समाप्त होता है।

## Basic Block

A basic block is a block that has no branches or jumps within it. It starts at a specific address and ends at the address of the next block or at a branch instruction.

एक मूल खंड एक ऐसा खंड है जिसमें इसके भीतर कोई शाखा या जंप नहीं होता है। यह एक विशेष पते पर शुरू होता है और अगले खंड के पते या एक शाखा निर्देश पर समाप्त होता है।

## Control Flow Graph (CFG)

A Control Flow Graph (CFG) is a graphical representation of a program's control flow. It consists of nodes that represent basic blocks and edges that represent the flow of control between these blocks. The CFG provides a high-level view of how a program executes and can be useful for understanding the program's behavior and identifying potential vulnerabilities.

नियंत्रण प्रवाह ग्राफ (CFG) एक कार्यक्रम के नियंत्रण प्रवाह का ग्राफिकल प्रतिनिधित्व है। इसमें मूल खंडों को प्रतिष्ठित करने वाले नोड और इन खंडों के बीच नियंत्रण के प्रवाह को प्रतिष्ठित करने वाले एज होते हैं। CFG कार्यक्रम के निष्पादन का एक उच्च स्तरीय दृष्टिकोण प्रदान करता है और कार्यक्रम के व्यवहार को समझने और संभावित सुरक्षा खतरों की पहचान करने के लिए उपयोगी हो सकता है।

## Symbolic Execution

Symbolic execution is a technique used in program analysis to explore all possible paths of execution by representing program inputs symbolically rather than concretely. It allows for the automatic generation of test cases and can be used to identify vulnerabilities such as buffer overflows and SQL injection.

प्रतीकात्मक निष्पादन कार्यक्रम विश्लेषण में एक तकनीक है जिसका उपयोग कार्यक्रम निष्पादन के सभी संभावित मार्गों का अन्वेषण करने के लिए किया जाता है, जहां कार्यक्रम इनपुट को वास्तविक रूप से नहीं बल्कि प्रतीकात्मक रूप से प्रतिष्ठित करके प्रतिष्ठित करता है। इससे परीक्षण के स्वचालित उत्पादन की अनुमति होती है और इसका उपयोग बफर ओवरफ्लो और SQL इंजेक्शन जैसी सुरक्षा की कमियों की पहचान करने के लिए किया जा सकता है।

## Constraint Solver

A constraint solver is a tool used in symbolic execution to solve constraints that arise from symbolic representations of program inputs. It takes these constraints as input and computes possible solutions that satisfy the constraints. Constraint solvers are used in various applications, including program analysis, software verification, and automated testing.

एक निबंधक समाधानकर्ता एक उपकरण है जिसका प्रतीकात्मक निष्पादन में उत्पन्न निबंधक प्रतिष्ठानों से उत्पन्न निबंधकों को हल करने के लिए उपयोग किया जाता है। यह इन निबंधकों को इनपुट के रूप में लेता है और इन निबंधकों को पूरा करने वाले संभावित समाधानों की गणना करता है। निबंधक समाधानकर्ता कई अनुप्रयोगों में उपयोग होते हैं, जिनमें कार्यक्रम विश्लेषण, सॉफ़्टवेयर सत्यापन और स्वचालित परीक्षण शामिल हैं।
```python
#Blocks
block = proj.factory.block(proj.entry) #Get the block of the entrypoint fo the binary
block.pp() #Print disassembly of the block
block.instructions #"0xb" Get number of instructions
block.instruction_addrs #Get instructions addresses "[0x401670, 0x401672, 0x401675, 0x401676, 0x401679, 0x40167d, 0x40167e, 0x40167f, 0x401686, 0x40168d, 0x401694]"
```
# सिमुलेशन प्रबंधक, स्थितियाँ

एनजीआर के एक महत्वपूर्ण फीचर है सिमुलेशन प्रबंधक। यह एक उच्च स्तरीय टूल है जो एनजीआर के विभिन्न विशेषताओं को उपयोगकर्ता के लिए सुलभ बनाता है। सिमुलेशन प्रबंधक के माध्यम से, आप एनजीआर के विभिन्न स्थितियों को निर्माण, प्रबंधित और विश्लेषण कर सकते हैं।

एनजीआर में, हर स्थिति एक विशेष रूप में प्रतिष्ठित होती है और एक स्थिति में आपके पास विभिन्न डेटा, रचनाएँ और विशेषताएँ होती हैं। आप सिमुलेशन प्रबंधक का उपयोग करके एक स्थिति को निर्माण कर सकते हैं, उसे प्रबंधित कर सकते हैं और उसका विश्लेषण कर सकते हैं।

सिमुलेशन प्रबंधक के माध्यम से, आप एनजीआर के विभिन्न स्थितियों के बीच प्रवेश कर सकते हैं और उन्हें विश्लेषण कर सकते हैं। आप एक स्थिति को बदल सकते हैं, उसे रीसेट कर सकते हैं और उसे वापस ला सकते हैं। इसके अलावा, आप सिमुलेशन प्रबंधक के माध्यम से विभिन्न स्थितियों के बीच डेटा को साझा कर सकते हैं।

सिमुलेशन प्रबंधक आपको एनजीआर के विभिन्न विशेषताओं को अधिक सुलभ बनाने में मदद करता है और आपको अधिक उच्च स्तरीय विश्लेषण करने की अनुमति देता है। यह आपको अपने एनजीआर प्रोजेक्ट को अधिक प्रभावी ढंग से प्रबंधित करने में मदद कर सकता है।
```python
#Live States
#This is useful to modify content in a live analysis
state = proj.factory.entry_state()
state.regs.rip #Get the RIP
state.mem[proj.entry].int.resolved #Resolve as a C int (BV)
state.mem[proj.entry].int.concreteved #Resolve as python int
state.regs.rsi = state.solver.BVV(3, 64) #Modify RIP
state.mem[0x1000].long = 4 #Modify mem

#Other States
project.factory.entry_state()
project.factory.blank_state() #Most of its data left uninitialized
project.factory.full_init_statetate() #Execute through any initializers that need to be run before the main binary's entry point
project.factory.call_state() #Ready to execute a given function.

#Simulation manager
#The simulation manager stores all the states across the execution of the binary
simgr = proj.factory.simulation_manager(state) #Start
simgr.step() #Execute one step
simgr.active[0].regs.rip #Get RIP from the last state
```
## फ़ंक्शन को कॉल करना

* आप `args` के माध्यम से एक तालिका और `env` के माध्यम से एक पर्यावरण चरों की डिक्शनरी को `entry_state` और `full_init_state` में पास कर सकते हैं। इन संरचनाओं में मूल्य स्ट्रिंग या बिटवेक्टर हो सकते हैं, और इन्हें स्टेट में आर्ग्यूमेंट्स और पर्यावरण के रूप में सीमित नकली नहीं किया जाएगा। डिफ़ॉल्ट `args` एक खाली सूची है, इसलिए यदि आप विश्लेषण कर रहे कार्यक्रम को कम से कम एक `argv[0]` ढूंढ़ने की उम्मीद करते हैं, तो आपको हमेशा उसे प्रदान करना चाहिए!
* यदि आप `argc` को संकेतात्मक बिटवेक्टर के रूप में चाहते हैं, तो आप `entry_state` और `full_init_state` कंस्ट्रक्टर के लिए `argc` के रूप में एक संकेतात्मक बिटवेक्टर पास कर सकते हैं। हालांकि, सतर्क रहें: यदि आप ऐसा करते हैं, तो आपको उस स्थिति के लिए एक सीमा जोड़नी चाहिए कि आपके लिए argc का मान आपके `args` में पास किए गए आर्ग्यूमेंट्स से अधिक नहीं हो सकता है।
* कॉल स्टेट का उपयोग करने के लिए, आपको इसे `.call_state(addr, arg1, arg2, ...)` के साथ कॉल करना चाहिए, जहां `addr` वह फ़ंक्शन का पता है जिसे आप कॉल करना चाहते हैं और `argN` उस फ़ंक्शन के Nth आर्ग्यूमेंट को पायथन इंटीजर, स्ट्रिंग या एरे के रूप में, या बिटवेक्टर के रूप में पास कर सकते हैं। यदि आपको मेमोरी आवंटित करनी है और वास्तव में एक ऑब्जेक्ट के पॉइंटर को पास करना है, तो आपको इसे PointerWrapper में लपेटना चाहिए, अर्थात् `angr.PointerWrapper("point to me!")`। इस API के परिणाम थोड़ा अप्रत्याशित हो सकते हैं, लेकिन हम इस पर काम कर रहे हैं।

## बिटवेक्टर्स
```python
#BitVectors
state = proj.factory.entry_state()
bv = state.solver.BVV(0x1234, 32) #Create BV of 32bits with the value "0x1234"
state.solver.eval(bv) #Convert BV to python int
bv.zero_extend(30) #Will add 30 zeros on the left of the bitvector
bv.sign_extend(30) #Will add 30 zeros or ones on the left of the BV extending the sign
```
## संकेतात्मक बिटवेक्टर और बाधाएं

एनजीआर (angr) एक पायथन लाइब्रेरी है जो बाइनरी एनालाइसिस और रिवर्स इंजीनियरिंग के लिए उपयोग होती है। यह एक शक्तिशाली टूल है जो संकेतात्मक बिटवेक्टर और बाधाओं का उपयोग करके बाइनरी कोड को विश्लेषण करने की क्षमता प्रदान करती है।

एनजीआर में, संकेतात्मक बिटवेक्टर एक विशेष प्रकार का डेटा प्रतिष्ठान है जो बाइनरी कोड के विभिन्न भागों को प्रतिनिधित करता है। यह बिटवेक्टर बाइनरी कोड के निर्दिष्ट बिटों की स्थिति को दर्शाता है और इसे विभिन्न रूपों में प्रदर्शित कर सकता है, जैसे कि बाइनरी फ़ाइल, रजिस्टर, मेमोरी और इनपुट पैरामीटर।

एनजीआर में बाधाएं एक महत्वपूर्ण भूमिका निभाती हैं। ये शर्तें होती हैं जो बिटवेक्टर को निर्दिष्ट मानों और संबंधों के लिए प्रतिबद्ध करती हैं। इन बाधाओं का उपयोग करके, एनजीआर बिटवेक्टर के लिए संभावित मानों की सीमा तय कर सकता है और इसे विश्लेषण करने के लिए उपयोगी सूचना प्रदान कर सकता है।

एनजीआर के संकेतात्मक बिटवेक्टर और बाधाओं का उपयोग करके, आप बाइनरी कोड के विभिन्न भागों को विश्लेषण कर सकते हैं, उन्हें मॉडिफ़ाई कर सकते हैं और नई बाधाएं तैयार कर सकते हैं। यह आपको बाइनरी कोड के विभिन्न पहलुओं को समझने और मानों की सीमा तय करने में मदद करता है, जिससे आप उसे अधिक सुरक्षित बना सकते हैं या नई विशेषताओं को जोड़ सकते हैं।
```python
x = state.solver.BVS("x", 64) #Symbolic variable BV of length 64
y = state.solver.BVS("y", 64)

#Symbolic oprations
tree = (x + 1) / (y + 2)
tree #<BV64 (x_9_64 + 0x1) / (y_10_64 + 0x2)>
tree.op #'__floordiv__' Access last operation
tree.args #(<BV64 x_9_64 + 0x1>, <BV64 y_10_64 + 0x2>)
tree.args[0].op #'__add__' Access of dirst arg
tree.args[0].args #(<BV64 x_9_64>, <BV64 0x1>)
tree.args[0].args[1].op #'BVV'
tree.args[0].args[1].args #(1, 64)

#Symbolic constraints solver
state = proj.factory.entry_state() #Get a fresh state without constraints
input = state.solver.BVS('input', 64)
operation = (((input + 4) * 3) >> 1) + input
output = 200
state.solver.add(operation == output)
state.solver.eval(input) #0x3333333333333381
state.solver.add(input < 2**32)
state.satisfiable() #False

#Solver solutions
solver.eval(expression) #one possible solution
solver.eval_one(expression) #solution to the given expression, or throw an error if more than one solution is possible.
solver.eval_upto(expression, n) #n solutions to the given expression, returning fewer than n if fewer than n are possible.
solver.eval_atleast(expression, n) #n solutions to the given expression, throwing an error if fewer than n are possible.
solver.eval_exact(expression, n) #n solutions to the given expression, throwing an error if fewer or more than are possible.
solver.min(expression) #minimum possible solution to the given expression.
solver.max(expression) #maximum possible solution to the given expression.
```
## हुकिंग

Hooking एक तकनीक है जिसमें हम किसी एक्शन को अपने नियंत्रण में लेने के लिए कोड को बदलते हैं। हुकिंग का उपयोग करके हम किसी भी फ़ंक्शन को बदल सकते हैं और उसे अपनी इच्छानुसार व्यवहार करने के लिए प्रेरित कर सकते हैं। हुकिंग का उपयोग करके हम एक्शन को निगरानी कर सकते हैं, डेटा को बदल सकते हैं, और अन्य विभिन्न कार्रवाईयों को कर सकते हैं।

यहां हम दो प्रमुख हुकिंग तकनीकों को देखेंगे:

### 1. Inline Hooking

इनलाइन हुकिंग में, हम किसी फ़ंक्शन को बदलने के लिए उसके कोड को सीधे बदलते हैं। हम फ़ंक्शन के प्रथम कुछ बाइट को ओवरराइड करके इसे हुक करते हैं। इस तरीके में, हम फ़ंक्शन को बदलने के लिए उसके नियंत्रण झंडे को बदलते हैं और फ़ंक्शन को अपने नियंत्रण में लेते हैं।

### 2. API Hooking

API हुकिंग में, हम किसी एपीआई को हुक करके उसके व्यवहार को बदलते हैं। हम एपीआई को हुक करके उसे अपने नियंत्रण में लेते हैं और उसके वापसी मान को बदलते हैं। इस तरीके में, हम एपीआई के व्यवहार को बदलकर उसे अपनी इच्छानुसार व्यवहार करने के लिए प्रेरित करते हैं।

ये दोनों तकनीकें बहुत उपयोगी हो सकती हैं जब हम किसी सॉफ़्टवेयर के व्यवहार को बदलना चाहते हैं या किसी विशेष फ़ंक्शन को निगरानी करना चाहते हैं।
```python
>>> stub_func = angr.SIM_PROCEDURES['stubs']['ReturnUnconstrained'] # this is a CLASS
>>> proj.hook(0x10000, stub_func())  # hook with an instance of the class

>>> proj.is_hooked(0x10000)            # these functions should be pretty self-explanitory
True
>>> proj.hooked_by(0x10000)
<ReturnUnconstrained>
>>> proj.unhook(0x10000)

>>> @proj.hook(0x20000, length=5)
... def my_hook(state):
...     state.regs.rax = 1

>>> proj.is_hooked(0x20000)
True
```
इसके अलावा, आप `proj.hook_symbol(name, hook)` का उपयोग कर सकते हैं, पहले तर्क के रूप में एक प्रतीक का नाम प्रदान करके, प्रतीक के जीवने वाले पते को हुक करने के लिए।

# उदाहरण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
