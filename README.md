
Introduction

In the next several class sittings, you will be working on implementing a file system that we will call cifs. We will simulate this file system on an image file that will behave as a block device.

You will test the creation of voluumes, establishing a file system, navigate a file directory hierarchy, create and delete files and folders, inspect files and folders, and read and write to files.

The whole project is worth 100 points distributed evenly among four steps.

There is substantial seed code provided in the archive file cifs.zip. You will follow the lab instructions literally and use the predefined data structures, the interface (functions) and the functionality already implemented in the seed code. A blockVolume (the block device simulator) is already completed for you in the sed code.
	To fully understand the descriptions of the project tasks it is necessary to read them along with the comments embedded in the seed code.

All test code in the testing functions testStep1(), testStep2(), testStep3() and testStep4() should be developed incrementally without deleting the earlier tests. Since all the data structures are predefined, you should encounter problems with compiliation of all successive tests.
	You must submit incremental solutions for each of the steps. Submissions that include the functionality from the further steps will be rejected. Meaning, you might break something implementing testStep3(), so freeze each code base when completed.
	You must use Ubuntu Linux for this lab.
Step 1: Create Files
Description

A cifs volume consists of a large number of blocks of equal size that can hold a variety of information. It is divided into three parts:

Bitvector

    The first group of blocks is reserved for a bitvector for managing the free storage on the volume. The bitvector that spans a number of blocks holds bits that indicate whether the corresponding blocks in the volume are free or not. The number of the blocks reserved for the bitvector is determined by the size of the volume, since there must be as many bits in the bitvector as there are blocks in the storage. You are required to use fast bit-wise operations to search and modify bits in the bitvector.
Superblock

    The block following the bitvector is reserved for the volume’s superblock. The superblock holds information about the file system such as the number of blocks, the size of a block, and the reference to the block holding the root folder of the file system.
Storage

    The storage consists of blocks that are used by folders and files. Each block can hold either

        a folder or a file descriptor,

        an index to other blocks, or

        raw data (file content as bytes).

The first two storage blocks (a folder descriptor and an index block) are allocated to hold the content of the the root folder of the file system. All blocks reserved for the bitvector, superblock are marked as unavailable in the bitvector. All other blocks - including the blocks that hold information for the root folder - are marked as taken when they are allocated.

A file-descriptor node holds meta-data information about a file or a directory such as its size, access rights, creation time, access time, and modification time, along with a reference to the block with actual content of the file or the directory. The superblock contains a reference to the block holding a file descriptor for the root folder of the volume.

In a block with the type of a folder, the size field indicates the current number of files in the folder, and the block reference field points to the index block that holds references to all files or folders in this folder. Each reference is an index to the block holding the file descriptor of the corresponding file or a folder. Of course, each of the subfolders may in turn hold other files or folders, and so on. There is an upper limit on the number of files or folders in a single folder.

In case of a block with a type of a file, the size field in the file descriptor indicates the actual size of the file. The block reference either points to a single data block if the size of the file is less than the size of a block (less the space needed for the type), or to an index block holding references to data blocks for larger files.
Draw It

As we have seen throughout the semester, drawing is very important. Diagramming is probably the correct technical term in a software engineering context, but it sounds less fun. For this step, I want you to draw the relationships between all the data structures, and what it would look like as you add files/data/folders. You need to be able to draw this in order to understand how all the data structures/bitvector/superblock/and storage interact with each other. The hash table for finding files quickly will also be very important to draw.

This will not be trivial, it would be completely reasonable to spend the first week of the project on this step.
Implementation

The seed code defines sizes of all the components of the volume; you may change them to fit your Ubuntu Linux system capabilities.

In this step, implement functions to:

    create the file system,

    create a file or a folder, and

    obtain file information.

The files are predefined for you in the code with extensive descriptions. Note that some elements of this functionality are already implemented.
Notes

In this step:

    You should focus on creation of folders and files in/from the file system volume.

    If you need a some specific data for testing (e.g., process id, user id, or access rights), then use the simulated filesystem context.

Submission

The submission must be in a subdirectory called step_1.

Submit your drawing.

Please submit all source code for your implementation of Step 1 only along with the build configuration files. You must provide transcripts of building the system, and of thorough testing sessions that prove that your code works as specified in the assignment.

The functionality should be tested in a function testStep1().
	Submissions that include the functionality from the further steps will be rejected.
Step 2: Mount the Filesystem
Description

In this step, you will implement mounting of the simulated file system. The supporting data structures and the mounting function are predefined and described with details in the seed code.

Mounting is a multi-stage process:

    The superblock is read from the volume and copied to the file system context for fast access.

    The second stage of the mounting process involves copying of the bit vector blocks from the simulated volume to memory-resident version. This is also done also for efficiency, since accessing hardware-based bitvector would be much slower than accessing its memory-based copy. However, care must be taken to make sure that any changes to the in-memory version are properly propagated to the hardware for permanent storage.

    The third stage of the mounting process involves a traversal of the simulated volume starting in the root and constructing an in-memory directory of all folders and files on the volume. The directory is an efficient data structure that provides fast access to the critical information about the content of the file system without resorting to slow access to an external device. The directory should hold the information about the folders and files, and preserve the file system hierarchy. For even more efficiency, it should be implemented as a hash table. A hash of the name of the folder or a file discovered during the traversal of the volume will be used as an index to the hash table (the index will be used as the file handle for any future fast reference). Each entry in the hash table is a head of a conflict-resolution linked list of nodes that keep the information about the folders and files whose names hash to the same entry; the file unique global identifier should be used for resolution of hashing conflicts.

    The node of the conflict resolution list includes a copy of the corresponding file descriptor block in the file system. To ensure efficient traversal of the file system, the structure of the file system hierarchy is preserved in the in-memory directory by including a file handle of the folder containing the currently analyzed folder or file (i.e., the index to the hash table of the parent folder). Examine the code to get the details.

The in-memory versions of the superblock, the bitvector, and the directory are part of the file system context structure.

If a new file is created or deleted, appropriate changes to the superblock, to the directory and to the bit vector must be made in both the memory and on the volume. Proper care must be taken to ensure consistency of the information in the in-memory data structures and on the volume.

To minimize the chance for collisions, the size of the hash table-based directory has been predefined for you to a prime number surpassing (but not by much) the number of blocks in the file system. The seed code implements the djb2 hashing (http://www.cse.yorku.ca/~oz/hash.html).
Notes

In this step:

    You must modify as needed the functions implemented in step 1. For example, the functions to create and to delete files must also update the in-memory directory.

    If you need a some specific data for testing, like process id, user id, or access rights, then use the simulated FUSE context.

Submission

The submission must be in a subdirectory called step_2.

Please submit all source code for your implementation of Step 1 and Step 2 only along with the build configuration files. You must provide transcripts of building the system, and of thorough testing sessions that prove that your code works as specified in the assignment.

The functionality should be tested in a function testStep1() and testStep2().
	Submissions that include the functionality from the further steps will be rejected.
Step 3: Open/Read/Write to Files
Description

In this step, you will be working on writing to a file and reading from a file.

In particular, you should implement functions to:

    open a file or a folder,

    read the content of an opened file or a folder,

    write content to an opened file or a folder.

Files must be opened to perform reading and writing operations. Similarily, a folder must be opened to create or delete files or folders that it contains.

The file system context includes the head of a list of all processes with currently opened files that must be properly maintained to ensure that:

    file are not re-opened before closing,

    access rights are observed, and

    files or folders are not deleted while being still in use (with the use of reference counts maintained in the directory).

The list also maintains a working directory for a process accessing the file system. The list has its head in the file system context which is initialized to NULL on the mounting of the volume.

The reading and writing functions in this project write and read files in their entirety. Specifically, the write function first deletes the old content, and then acquires blocks to hold the new data.

The details of the functions are in the comment sections of the code.
Notes

In this step:

    You must modify as needed the functions implemented in steps 1 and 2 to accommodate the new functionality.

    If you need a some specific data for testing, like process id, user id, or access rights, then use the simulated FUSE context.

Submission

The submission must be in a subdirectory called step_3.

Please submit all source code for your implementation of Step 1, Step 2, and Step 3 only along with the build configuration files. You must provide transcripts of building the system, and of thorough testing sessions that prove that your code works as specified in the assignment.

The functionality should be tested in a function testStep1(), testStep2() and testStep3().
	Submissions that include the functionality from the last step will be rejected.
Step 4: Delete/Close Files
