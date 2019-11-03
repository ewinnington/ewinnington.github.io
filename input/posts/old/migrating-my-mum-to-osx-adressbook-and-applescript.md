Title: Migrating my mum to OS X - Adressbook and applescript
Published: 24/01/2012
Tags: [Migrated, OSX] 
---

One potentially show-stopping problem for migrating my mum was that she had over 1500 contacts in Windows Live Mail (WLM). These were sorted into many groups, which had to be recreated in the mac's address book application. The main problem was that Microsoft's tool does not give any option to export the groups a contact is associated with, either through hotmail, through the app or any other method I tried until I found this solution:

First, you should export the complete contact list from Windows Live Mail, this can be done from the edge of the address book tool bar. I used .vcf format, which creates 1 card per contact. You should also be able to find this option in File -> Export.

![Exporting contacts from Windows Live Mail](http://www.freeemailtutorials.com/i/img145.jpg)

On the Windows PC, in the address book for WLM, you have the split screen view of the groups on the right and the people assigned to them on the left. If you double click on a group name, that group opens in a detail window. In the bottom part of that window is a _**selectable**_ list of all the contacts belonging to that group. Copy the whole text area with CTRL-C and paste that in Excel with CTRL-V. Then you'll need to split that input into separate lines since that are comma separated. Save each group as an individual text file (usually named the same as the group) with the people belonging to group.

The next step is to move these files to the mac. First of all import all the contacts into the mac's address book. This can be done by dragging all the contacts .VCF cards to the address book window.

Now comes the reassigning the contacts to the groups. What I did was create the group in the address book, then edit my following applescript to set the group I wanted to import to "MyDesiredGroupName", then run the following script:

**tell** **me** **to** **set** **the** text item delimiters **to** (**ASCII character** 13) **tell** _application_ "Finder" **  set** Names **to** _paragraphs_ **of** (**read** (**choose file** with prompt "Pick text file containing names to go in group")) **  repeat** **with** nextLine **in** Names **    if** length **of** nextLine **is** **greater than** 0 **then** **      tell** _application_ "Address Book" **        set** thePeople **to** (**every** _person_ **whose** name = nextLine) **        set** theGroup **to** "MyDesiredGroupName" **        repeat** **with** thisPerson **in** thePeople **          add** thisPerson to _group_ theGroup **        end** **repeat** **        save** _application_ "Address Book" **      end** **tell** **    end** **if** **  end** **repeat** **end** **tell**

By setting the MyDesiredGroupName, you should be able to get most of your contacts imported into the correct group. I had a slight issue with about 1-2% of the contacts not being selected or found properly, especially those that had accents in their names (é, è, ö, ü, ...). If you look in the output log of the script, it is fairly easy to find those that were not selected correctly by the line "**set** thePeople **to** (**every** _person_ **whose** name = nextLine)" since they appear with a { } empty selection in the output log. 

In fact, I took the output log and passed it into a text editor searching for { } to find the few people that were not correctly imported. That allowed me to make sure the groups were correctly populated.

Remember to check the number of people belonging to the group at the end of the import.

That's it. I hope this helps someone else make the switch from WLM to apple's address book.
