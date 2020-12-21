# Welcome to Basics of Storage ( 1 ) 

# Agenda - 

 - [ ]  Cover storage from a PC point of view 
 - [ ]  Prepare for server side storage sessions  
 - [ ] Assume No Prior knowledge about any disk or server

# Storage <=> secondary storage 

 - We aren't interested in Cache, RAM, or registers - We are interested
   in non primary storage.
   
		   Time taken to access value at address X - 	t(X) 
		   Time taken to access value at address Y  - 	t(Y)
	 `if ( t(X) == t(Y) )` 
	    
					 Primary Storage 
	

    else 	

					 Secondary Storage 
   
Example - CD, Floppy, Disks <=> non primary storage 
*Quiz* : When  can primary storage be slower than secondary storage? 


## Tracks, Sectors, cylinder, heads


![enter image description here](https://upload.wikimedia.org/wikipedia/commons/4/41/Hard_drive_geometry_-_English_-_2019-05-30.svg)
[Diagram Link](https://upload.wikimedia.org/wikipedia/commons/4/41/Hard_drive_geometry_-_English_-_2019-05-30.svg)

 - Concentric circles are called tracks 
 - craving out some area in the those concentric circles  gives sectors.
 - Craving out cross sections of tracks areas will represent cylinders.
 - Same track of every surface is aligned - which gives a cylinder 
 *The location of data needs - integer sector and integer  track id PAIR* - Single Surface
 *The location of data needs   - integer sector, integer track, head boolean  triplet* 
 *The location of data needs   - integer sector, integer track, head integer*
 _Quiz : How much far is the head from the surface?_
 Tracks are aligned  crave out a cylinder,  an address [ block , not bytes ]
 head #, cylinder #, sector # will give a location 
  

## Secondary Storage is Slow

"All problems in computer science can be solved by another level of indirection"
https://en.wikipedia.org/wiki/Fundamental_theorem_of_software_engineering

 - Collect a pool of _blocks_ [ data + triplet location ] in the primary memory 
 - Reserve a primary memory 's part for these collection of blocks [ Buffer Cache ] 
 - Buffer = data + header [ Decide on trade off  <-> more buffer number  - faster ] 
_Quiz_ - What is the trade off? Can normal user programs maintain the buffer pool?
 - Buffer = [ Blocks' triplet + id + status + pointer to Data + pointer to prev buffer + pointer to next buffer + pointer to next free buffer list + pointer to previous free buffer list + pointer to a queue  ] 
 
 _Quiz_ What could be possible status sets  of the buffer? See previous _Quiz_
 
   Valid, locked, delayed write, [ think of the crash ], Read/Write, #waiting/sleeping.
   Valid <=> delayed writes 
 
![enter image description here](https://drawings.jvns.ca/drawings/filesystem-cache.png)
[Diagram](https://drawings.jvns.ca/drawings/filesystem-cache.png)

 

## Managing the Buffer Cache

_Quiz_ 
What is the size of data in the PAGE CACHE? Is is it equal to or a multiple of size of the block on the secondary storage? 

 - Limited number of buffer in the buffer pool in the primary storage 
 - At start up of OS, the buffers in the buffer cache ARE empty, 
 - While running the OS, some buffers can be marked _free_ [ free buffer can contain the block of data ] 
	 - Meta Data node <=> node <=>  node <=> node <=> node <=> Meta Data node
	 - Each Node contains data from [ block X, block Y, block Z ] 
	 - Use Least Recently Used Methods - Insert at the end, detach from the front
	 - Most recently used[ written to ] at  the end of the _list_
 - How to tell if the data block is there in the buffer cache?
 - How to decide if one needs to see secondary storage or see the buffer cache?
 - Block Number which is a triplet [ modulo some CONST ] - construct id[hash index]
 - https://tldp.org/LDP/lki/lki-4.html
 - LRU Cache of Blocks  = Free list buffers + not free List buffers [ take +  put back ]
 _Quiz_: What one sees in the buffer cache at start of machine? Of a  process? 
 Process makes a request for block # 3876 [ Check the Bucket, check the buffer, check if buffer data exists, remove/read/write the buffer data, update  free list ] 
 
 

## Managing the buffer

What if we have delayed write marked upon a buffer data? 
Should the OS wait? No

 - **getblk** rules [ I/P - Block # from program, O/P - a locked buffer ] 
	 - The locked buffer [ An area containing the data requested block, or an free buffer where the data will be put ] 
	 - What if the data is there is buffer cache but it marked locked?
	One need not find a new buffer, to avoid duplication
	 - What if there is no buffer marked as _free_?  must wait 
	 - What if the buffer is marked _free_ + _delayed write_? need not not wait 
	 - Continue to find a new buffer location, return with the locked marked buffer
- **brelese**
	- wake up all process waiting for the buffer + wake up all process waiting for any buffer
	- put the buffer data into the disk secotrs back 
	- Put back buffer in the free list - insert at the end of the free list 
	- Unlock the buffer 

 - **bread** 
	 - getblk and read sectors into the buffer and return the buffer 
- **readahead**
	- https://man7.org/linux/man-pages/man2/readahead.2.html
	- Bring data without sleeping the process, for the other blocks 
- **bwrite**
	- Check for sleep or non sleep operations, brelease the buffer.
	- Insert at head of the free list nodes, if delayed write marked in status of buffer. 

## Representing a file 

![enter image description here](https://drawings.jvns.ca/drawings/inodes.png)[Diagram](https://drawings.jvns.ca/drawings/inodes.png)

Need to have address of the hard disk address set - Blocks
Disks <=> *[ Boot Blocks, Super Blocks, Inode List, Data Block]*
Think of _indirection_ 
super blocks = *[size of installation, total counter size of filem, free data blocks, free inodes ]*

*Quiz* : What if free inodes count is exhausted, but disk space is not exhausted? 

*Quiz* : Why should the inode copy be there in the primary storage? Think buffer cache

In Secondary Storage inode versus IN Primary storage Inode 
 **Secondary Storage Inode** <=> 
 *[ security, reg ||dir || other, modification time of file, modification of inode, size of file, addresses of the blocks on disk, ownership  ]* 
 **In Primary storage Inode** <=>
 [ isLocked, areProcessWaiting, isFileChanged, dev, inode integer + pointers + reference counter ] 
 *What it means for an indoe to be free?*
 
  if ( ref count >= 0 ), not in free list, but just unlocked - 
  Unlock does not imply free in case of inode, but in case of buffers in buffer cache.
  lck(inode) then getblck then unlck(inode). Ref count is decreased only on close.
 
 
https://man7.org/linux/man-pages/man2/stat64.2.html


# Indirection in Inode [ Actual TOC of inode] 

[ 0, 1 , 2 , 3 , 4 , 5, 6, 7, 8, 9, 10, 11, 12] 
Indirection Level 1 : 0 to 9  have the address to disk address. [ Direct Address ] 
Indirection Level 2 : 10 & 11 [ Single Indirect Pointer ] 
Indirection Level 3 : 12 [ Double Indirect Pointer ] 

			What it achive? 
			block triplet id = 4 bytes, size of block 1K
			Any block can hold 256 * 4 [ # triplet ids] 
			Direct Ptr - 10K 
			Single Indirrect -  256 * 1K
			Double Indirect - 1K * 256 * 256 
			Triple Indirect - 1K * 256 *256 * 256 

## Reading/Opening any file 

 - Get the inode from primary memory [ the inode number ] 
 - Find an inode with refCount = 0 
 - **iget** I/P inode # O/P locked inode 
	 - Find if in inode in inode cache, check until inode is not locked anymore. 
	 - [ Unlocked inode does not mean inode is free ] 
	 - If not locked,  travese the free list, remove from free list, increase refCount
	 - <> if not in inode cache, Not a method of knowing if no inode is in free list
	 - <> traverse inode free list, find free inode, read disk copy of inode, bread, init inode, reset inode. 
	 - return the inode struct 
 - **iput** I/P release the inode 
	 - lock inode, refCount--, [ Why lock inode is befroe decrease refCount]
	 - Check refCount = 0  -> check if inode modified [ change secondary inode ] 
	 - Put into the free list, unlock the inode 
	 ![enter image description here](https://i.ytimg.com/vi/CtJYQsEN6CY/maxresdefault.jpg)
https://www.kernel.org/doc/htmldocs/filesystems/API-iput.html


## Representing directories 

inode's type - a special type of file [ Indirection ] !! 
Directore <=> [ <inode, file name > , < inode, file name> , < inode, file name > ] 
*Quiz:* How do you open a file, what minimum arguments you need to specify? 

**namei** 

 - [ ] check if part starts from root and update current  inode to root's inode. 
 - [ ]   update current inode to current directory inode
 - [ ]  Keep traversing  the path name string, until the string for the pathname is valid.
 - [ ] Maintain the permission for search, read directory. get inode # for  the matched substring directory name,
 - [ ] *release* current inode,     update the current inode to the new matched substring directory name.  
 - [ ] return null if all the substrings are evaluated, or return null if a  substring does not have an entry while searching.
 - [ ] return the valid inode, if reached.

 https://man7.org/linux/man-pages/man1/namei.1.html
 
 *Quiz:*  Should one also maintain a fater access for free inode lists? 
 
       
Next Topics 

 - [ ] ialloc, super block list 
 - [ ] file systems 
 - [ ] examples 
 
 
 
# Super Block free Inode List !  
  

![enter image description here](https://www.cems.uwe.ac.uk/~irjohnso/coursenotes/lrc/internals/images/fs4.gif)  
    
Until there are no more _free inode on disk_ *or* _super block free inode list is empty_

_What happens when a file is closed_?
    - _have newly free inode number_ (f)
    - _have older remembered inode number in the superblock free inode list_ (r)
    -  if ( f < r ) { r becomes f } ( only if the superblock free inode list is full ) 
    - increment free inodes counter, check for superblock is _not_ locked
    - store f in super block free inode list ( only if superblock free inode list is not full )
    - 


# On change of content in files 
  
![enter image description here](https://www.cems.uwe.ac.uk/~irjohnso/coursenotes/lrc/internals/images/fs2.gif)
_Quiz_ : Why cannot one can use simply a bit to donate if the data block is free or not, but one can donate if an inode is free?

- This is _mkfs_ does or making a file system on a disk. ^^  [ Why 172, 171 block # adjacent ] 
-  See left most entries in each of the linked list node **link block**  ^^
- _Number of block numbers keep getting reduced as the machine runs and process keep writing data_
- Copy the content, fully of 230 into the upper list _super block free data list_, makes 230 as free
- **alloc** I/P - file system number , O/P is a buffer (getblk - first time in buffer cache )
- get a block from the super block free list, if needed push list up ( don't forget to lock the list ) and unlock, hence need to wake up the tasks ( bread ) 
- mark the super block as modified and decrease the free number of free blocks 

## On Deletion of files   

- block # 424 is marked for delete, this is a data block 
- if the super block free data block list can contain block # 424 [ has space for it ], just insert
- if the super block free data block list does not have any more room, the 424 block # now points to the older super block free data block list [ see the left most entry ] 

  _Quiz_ What does defragmentation does?  
  Rearranges the super block free data block block list.

## On Scheduling of Requests 
  
![enter image description here](https://i.ibb.co/Wx0VNJV/req1.png)

Data Block tuples < track #, block # > 
![enter image description here](https://i.ibb.co/0CJZNH1/reeq2.png)

![enter image description here](https://i.ibb.co/RPrZ2XR/req3.png)

https://man7.org/linux/man-pages/man2/posix_fadvise.2.html

https://en.wikipedia.org/wiki/Elevator_algorithm

http://faculty.salina.k-state.edu/tim/ossg/File_sys/disk_scheduling.html

http://www.cs.iit.edu/~cs561/cs450/disksched/disksched.html

_Quiz_ Does one need such rules in case of non revolving drives?




