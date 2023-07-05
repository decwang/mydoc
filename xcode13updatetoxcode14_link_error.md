
After long research, I discovered the solution. Clang emits objc_msgSend in Xcode 14; previous versions of Xcode don't understand it. This needs to be disabled. Here is the link I used for the solution. Commited changes are available at this link. Big thanks to these guys.

By the way, selecting the compatible version for Xcode 14 of Command Line Tools from XCode > Preferences > Locations section, and also updating the Clang version (you can type clang -v in the terminal) to be compatible with this version. You can add -fno-objc-msgsend-selector-stubs settings to C Flags under Apple Clang - Custom Compiler Flags from Targets > Build Setting directly from Xcode


https://github.com/xamarin/xamarin-macios/issues/16223
https://github.com/xamarin/xamarin-macios/commit/dec29fe4339e1a2ca5df11fdd969130a4c1dc443



When I manually add the SDK that I exported in Xcode 13 to xcode 13 and xcode 14, it works for all devices without any problems. But when I export in xcode 14, it only works for simulators in xcode 13, I can't build on real devices, (in xcode 14 it works on all devices without any problems). For the SDK I exported in Xcode 14, the errors I get when I run it on real device in xcode 13 are as follows:

directory not found for option '-F/(framework path)'
Undefined symbols for architecture arm64:_objc_msgSend$subfiles
Undefined symbols for architecture arm64:_objc_msgSend$subfiles
...
clang: error: linker command failed with exit code 1 (use -v to see invocation)
I deleted the framework search path for the directory not found error and I found a solution to it, but I couldn't find a solution for the errors I got for all the sub-files of the SDK below:

Undefined symbols for architecture arm64:_objc_msgSend$subfiles
And

clang: error: linker command failed with exit code 1 (use -v to see invocation)
Thank you very much in advance for your comments.

swiftobjective-cxcodexcode13xcode14
Share
Improve this question
Follow
edited Oct 7, 2022 at 8:02
asked Oct 7, 2022 at 6:53
Batuhan Doğan's user avatar
Batuhan Doğan
5166 bronze badges
Can you be more precise ? What SDK, which Mac, which macOS ? – 
Ptit Xav
 Oct 7, 2022 at 9:14
@PtitXav Hello. Actually, it's a framework I created myself. I am using m1 macbook. Version macOS 12.6. – 
Batuhan Doğan
 Oct 7, 2022 at 13:06
Add a comment
1 Answer
Sorted by:

Highest score (default)
3

After long research, I discovered the solution. Clang emits objc_msgSend in Xcode 14; previous versions of Xcode don't understand it. This needs to be disabled. Here is the link I used for the solution. Commited changes are available at this link. Big thanks to these guys.

By the way, selecting the compatible version for Xcode 14 of Command Line Tools from XCode > Preferences > Locations section, and also updating the Clang version (you can type clang -v in the terminal) to be compatible with this version. You can add -fno-objc-msgsend-selector-stubs settings to C Flags under Apple Clang - Custom Compiler Flags from Targets > Build Setting directly from Xcode

Share
Improve this answer
Follow
edited Oct 27, 2022 at 5:51
answered Oct 11, 2022 at 8:41
Batuhan Doğan's user avatar
Batuhan Doğan
5166 bronze badges
Thanks for this solution @batuhan - I'm not completely adept at using XCode. I've tried pasting that -fno-objc-msgsend-selector-stubs line in under C Flags and I keep getting errors. Are you able to give any further details on how to add this flag in XCode? Thanks! – 
Desmond
 May 6 at 7:05
Add a comment
Your Answer
Sign up or log in
Post as a guest
Name
Email
Required, but never shown

By clicking “Post Your Answer”, you agree to our terms of service and acknowledge that you have read and understand our privacy policy and code of conduct.

Not the answer you're looking for? Browse other questions tagged swiftobjective-cxcodexcode13xcode14 or ask your own question.
The Overflow Blog
Part man. Part machine. All farmer. 
Throwing away the script on testing (Ep. 583)
Featured on Meta
Statement from SO: June 5, 2023 Moderator Action
Starting the Prompt Design Site: A New Home in our Stack Exchange Neighborhood
Does the policy change for AI-generated content affect users who (want to)...
Temporary policy: Generative AI (e.g., ChatGPT) is banned
Related
2
Objective-C errors with Xcode
6
Invalid Binary Or Invalid Swift Support
2
Objective-C project use Swift encounters a incompatible
117
Invalid Swift Support - Files don’t match
0
Compilation issue : Incompatibility of Swift versions between two packages
0
Swift framework problems with Xcode compatibility
40
Xcode 12 beta and iOS 14: Weird console logs "objc[5551]: Class ... is implemented in both"
11
Xcode 12 doesn't support new iOS version
2
Xcode error when Package.swift platform changed from v13 to v14 "v14 unavailable"
7
Xcode 14 project compile error：com.apple.xcode.tools.swift.compiler is not absolute
Hot Network Questions
How to express "I want something" in Korean
What does >&- do in a unix / linux terminal?
What is/are the supply (current?) of the wires in a #6 AWG Aluminum wire?
Quit a postdoc after 2 months?
"Hard" SF reaction to "Soft" SF Aliens
Can ETFs be broken down into the stocks composing them without paying capital gains tax?
Does an ionic bond have a dipole?
Can I build a RAID 5+1 system?
Will magical light sources that aren't explicitly created by a spell illuminate magical darkness?
5 chess pieces dominating a 5x5 grid
Does rebooting a phone daily increase your phone's security?
How would you say "The older a rabbit gets, the more it behaves like a dog."?
The LED is dead... Why?
Blog site generator written in shell script
more hot questions
 Question feed

STACK OVERFLOW
Questions
Help
PRODUCTS
Teams
Advertising
Collectives
Talent
COMPANY
About
Press
Work Here
Legal
Privacy Policy
Terms of Service
Contact Us
Cookie Settings
Cookie Policy
STACK EXCHANGE NETWORK
Technology
Culture & recreation
Life & arts
Science
Professional
Business
API
Data
Blog
Facebook
Twitter
LinkedIn
Instagram
Site design / logo © 2023 Stack Exchange Inc; user contributions licensed under CC BY-SA. rev 2023.6.26.43510